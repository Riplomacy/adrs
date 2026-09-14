# DH-002: Matched Interaction Defer Budgets Across Command and Component Dispatch

**Date:** 2026-09-14\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

`api/service/command.rs` and `api/service/component.rs` each race their handler against an
internal timer (`defer_delay`) before Discord's real 3-second interaction-ack deadline: if the
handler hasn't produced a response by the timer, the code immediately sends
`CreateInteractionResponse::Defer` and finishes the real response later via a followup edit
(`handle_slow_response`), using the same `child-task-awaiter` Lambda Extension (added in
`64cbd8e5`, 2025-04-20) to keep the invocation alive for that followup.

That same commit (`64cbd8e5`) quietly changed the command path's internal budget from 2500ms to
2000ms while leaving the (then separate) component path at 2500ms. No commit message, code
comment, or later refactor ever explained or revisited the discrepancy -- every subsequent split
(the component-service extraction, the 2026-06-19 configurable-delay change in `85fa81fc`)
preserved the drift rather than reconciling it. Jean recalls hitting flaky, unexplained
interaction timeouts in the past that were never root-caused; this is presumed to be the same
mechanism, just never isolated before.

Surfaced 2026-09-14 when Lobo clicked the `squad_unban` ticket category button and Discord showed
"The application did not respond," even though the ticket channel and all its messages were
created successfully in the background. Investigation initially chased the wrong things --
downstream latency (BattleMetrics/TicketService calls), a `tokio::sync::Mutex` held across an
await in `ComponentRegister` dispatch, wrong Discord deferral-type semantics, `ModalV2GatewayLayer`
treating components specially -- all ruled out with evidence (BattleMetrics/TicketService latency
architecturally can't matter once the internal timer fires and defers, regardless of how slow the
real work is; Discord's docs confirm type 5 is valid for every interaction kind; the mutex fix
shipped and the bug persisted; `ModalV2GatewayLayer` passes components and commands through
identically).

The decisive test: a purpose-built `/bot_delay_button` (component-path twin of the existing
`/bot_delay` command, both doing a bare `tokio::time::sleep` with no I/O) reliably reproduced the
exact same "Unknown Webhook" failure on a genuine cold start, while `/bot_delay` on the same
Lambda, same cold-start conditions, never failed. Since neither test does any downstream I/O, the
only remaining variable was the internal timer itself: 2000ms for commands, 2500ms for components.
Lowering the component budget to 2000ms to match immediately fixed it, confirmed via the same
`/bot_delay_button` cold-start test.

Working theory (not independently measured): there is a fixed, cold-start-only cost -- somewhere
in the 500-1000ms range -- between an interaction handler starting and the response actually
reaching Discord, on top of the already-reported Lambda "Init Duration." A 2000ms internal budget
absorbs that cost and stays under Discord's 3000ms deadline; a 2500ms budget doesn't. This is
consistent with every observation (always succeeds warm regardless of budget; the command path,
even deliberately delayed past its budget, never failed cold; the component path failed
deterministically cold at 2500ms and stopped failing at 2000ms) without needing to invoke
downstream latency, which the extension design is explicitly meant to make irrelevant once the
internal timer fires.

## Decision

`api/service/component.rs`'s `defer_delay` budget is `2000`, matching `api/service/command.rs`.
Both call sites must use the same value going forward -- there is no known reason for interaction
type to change how much margin is needed under Discord's shared 3-second deadline.

## Consequences

### Positive

- Component (button/select) interactions get the same cold-start safety margin commands already
  had, eliminating a failure mode that likely caused unexplained flakiness for over a year.
- `/bot_delay_button` stays in the codebase as a permanent, cheap regression probe for this exact
  class of bug -- posts a persistent (non-ephemeral) button so it survives long enough to catch a
  real cold start, unlike an ephemeral test panel whose own interaction token expires first.

### Negative

- Handlers that previously completed via the fast path (a direct `Message` response, no `Defer`)
  between 2000-2500ms of real work now always take the deferred path instead. This is strictly
  safer, but means `squad_unban`'s ticket flow (whose real work is routinely ~2.5s+) will almost
  always show Discord's "Bot is thinking..." state rather than an instant reply.

### Neutral

- The root mechanism for the fixed cold-start tax itself is not identified or measured here --
  only that matching the two budgets empirically removes its effect at the margin these commands
  operate at. If a future handler needs meaningfully more than 2000ms of margin under Discord's
  deadline even when warm, that's a different, unrelated problem (the fix here only concerns the
  cold-start-specific gap).

## Alternatives Considered

- **Eager-defer specific slow categories (e.g. `squad_unban`) instead of tuning the shared
  budget:** Rejected as the fix, though briefly implemented and reverted during investigation --
  it would have masked the actual bug (the 2000/2500 drift) behind a narrower, handler-specific
  workaround, leaving every other component interaction still exposed to the same cold-start
  failure the moment its own handler happened to run long.
- **Reduce lock scope in `ComponentRegister` dispatch (`.lock().await.call(...).await` holding the
  guard across the whole inner await):** Applied as a real cleanup (a known Rust async footgun,
  and the one structural difference from the lock-free command-register path), but confirmed via a
  deployed, still-failing cold-start repro that it was not the cause.
- **Investigate Lambda cold-start network/ENI/connection-pool warm-up as the cause:** Ruled out --
  `DiscordHandlerGw` isn't VPC-attached, and `/bot_delay`/`/bot_delay_button` do zero network I/O
  besides the interaction response itself, yet still reproduce the command/component split.

## Implementation Notes

- `lambdas/discord/discord_handler/src/api/service/component.rs`: `defer_delay(2500)` →
  `defer_delay(2000)`.
- `lambdas/discord/discord_handler/src/components/bot_delay_button.rs` /
  `lambdas/discord/discord_handler/src/commands/bot/delay_button.rs`: new `/bot_delay_button`
  command + component pair, the component-path A/B counterpart to the pre-existing `/bot_delay`
  command. Posts a persistent (non-ephemeral) channel message so the button remains clickable long
  enough to catch a real cold start; restricted to Signal, same as `/bot_delay`.
