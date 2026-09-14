# TICKET-001: Experimental In-House Ticketing System

**Date:** 2026-09-13\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

The current (third-party) ticket bot can't route ticket categories to the
specific staff teams that actually own them (Tech Team vs. Admin vs.
SquadStaff), and has no integration with this bot's own staff-roster,
permission model, or the BattleMetrics/identity data `riplomacy_bot` already
has direct access to. This is a feature gap, not a cost or reliability
problem with the existing bot.

Full design is `riplomacy_bot/design_docs/ticketing_system.md`.

## Decision

Build an in-house ticketing system as a new Smithy-backed **Ticket
Service**, run alongside the existing third-party ticket bot as an opt-in,
explicitly experimental alternative (not a cutover):

- A public category panel lets any member open a private ticket channel;
  categories and their staff role-mappings are configured (`tickets.toml`)
  without a code deploy.
- Channel lifecycle (create/authorize/close) and category-specific
  automated workflows live entirely in the new Ticket Service —
  `discord_handler` stays a thin front door (render panel, dispatch,
  relay response), never owning this domain logic itself, the same
  "no domain logic in the interaction handler" convention this repo
  already applies to data access.
- Ticket state (category, opener, snapshotted role mapping) is encoded in
  the channel's own `topic` field instead of a database — every relevant
  interaction already carries `channel_id`.
- Close is staff-initiated and **immediate**: Mark Resolved archives the
  transcript + attachments to a new S3 bucket and deletes the channel in
  one synchronous call. No delayed/cancellable close, no scheduler.
- The first category workflow is unban requests (`squad_unban`): checks
  identity, looks up BattleMetrics bans, and auto-unbans on an exact
  "Automatic Teamkill Kick" match under a safety cap — the first time this
  bot's runtime gets ban-*mutating* BattleMetrics access, not just reads.
  A successful auto-unban posts the result but does not auto-close the
  ticket, since close is now unrecoverable.

## Consequences

### Positive

- Closes a real routing/integration gap without betting the community's
  support channel on day-one feature parity against the existing bot.
- Reuses this repo's established patterns end to end: Smithy service
  shape, `bot_config`/`tickets.toml` config convention, `serenity::Http`
  Lambda-owned Discord client (`guides_apply` precedent), channel-topic
  as state store instead of a new table.
- Category-specific automated workflows (starting with auto-unban) are
  the actual value being tested, not just panel/routing plumbing.

### Negative

- Net-new mutate capability on `battlemetrics_api` (`unban(ban_id)`) is a
  new trust boundary — this bot's runtime can now action bans, not just
  read them.
- Best-effort (non-atomic) one-open-ticket-per-category check: a real,
  accepted race window that can produce a duplicate channel.
- Immediate, uncancellable close means an accidental or premature Mark
  Resolved can't be undone from within the system.

### Neutral

- Runs fully separate from the existing third-party ticket bot, with no
  cross-awareness between the two — retiring the existing bot is an
  explicitly separate, later decision.
- Two ticket-adjacent surfaces exist in parallel for the duration of the
  experiment (member choice, no forced migration).

## Alternatives Considered

- **Delayed/cancellable close** (Mark Resolved → countdown → auto-close,
  backed by an EventBridge Scheduler): considered and dropped. Staff
  clicking Mark Resolved is already the deliberate act; a grace-period
  countdown is nice-to-have, not necessary, and costs a second piece of
  infrastructure plus a stuck-state recovery story this experiment
  doesn't need. Revisit if immediate close proves too abrupt in practice.
- **Auto-close the ticket on a successful auto-unban match:** dropped once
  close became immediate/uncancellable — it would remove the last chance
  to catch a wrong automated match.
- **Fold ticket logic into `discord_handler` instead of a new service:**
  rejected, breaks the "no domain logic in the interaction handler"
  convention already applied to every other stateful feature in this repo.
- **Private threads instead of private channels:** would sidestep the
  500-channel/50-per-category caps, but Signal explicitly preferred the
  classic per-ticket-channel model.
- **DynamoDB (or another table) for ticket state:** rejected — every
  interaction that matters already carries `channel_id`, so the channel's
  own `topic` field is the natural key without a transactional data store.

## Implementation Notes

- New `TicketsConfig`/`TicketCategoryConfig` in `crates/bot_config/src/tickets.rs`,
  same shape as `staff.rs`'s rank `discord` field; validated by
  `config_apply`'s existing `check_role_refs` path.
- New Smithy client/server SDK following `services/staff_data_service`'s
  shape; CDK stack following `cdk/lib/services/staff-data/service-stack.ts`
  (weak `Fn.getStackOutput` cross-stack refs paired with explicit
  `addStackDependency` in `bot.ts`).
- New S3 bucket for transcripts/attachments, matching this repo's existing
  bucket practice (block-public-access only, no versioning/access logging).
- Full unit + Docker-integration test pyramid required, per this repo's
  test-first policy — see design doc's Testing section for the concrete
  plan (topic codec, authorization, autotk-match, `OpenTicket`/
  `EvaluateUnbanRequest`/`CloseTicket` end-to-end against `LocalWebhooks`
  and a new `TranscriptStoreTrait`).
- Exact Smithy operation/field naming, S3 bucket naming/lifecycle policy,
  and the Whitelist Issue workflow (§3c of the design doc) are
  implementation-time details, deliberately left open by this ADR.
