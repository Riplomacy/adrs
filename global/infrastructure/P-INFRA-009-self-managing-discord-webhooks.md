# INFRA-009: Self-Managing Discord Webhooks via Per-Name SSM Parameters

**Date:** 2026-09-16\
**Status:** Proposed\
**Deciders:** Jean-Sébastien Dominique

## Context

The bot depends on 6 live Discord webhooks (server restart/shutdown/update notices, staff activity reports, join-message changes, error reporting, active-staff summaries, bread-game events). All 6 are created by hand in Discord's UI, their URLs pasted into SSM Parameter Store (one exception posts through a TOML config field instead), then baked into Lambda env vars at CDK synth time via `valueForStringParameter`. No code path has ever called Discord's webhook-creation API. If a webhook is ever deleted or regenerated on Discord's side — accidentally, or as part of channel cleanup — every caller silently breaks until someone notices, manually recreates it in Discord, updates SSM, and redeploys.

Discord's API supports bots creating and managing their own webhooks (`MANAGE_WEBHOOKS` permission, `POST /channels/{channel.id}/webhooks`), and the bot already runs with that permission everywhere it currently posts through a pre-created one.

## Decision

The bot manages its own webhooks instead of depending on ones created by hand. Signal seeds each webhook's target channel (and thread, where the message needs to land in a thread) once, via a dedicated SSM parameter per webhook name. At runtime the bot resolves the webhook's current URL from that parameter — no longer baked into a deploy-time env var — and on a failed send (missing URL, or Discord's "Unknown Webhook" error) it creates a fresh webhook on the configured channel, persists the new URL back to SSM, and retries once.

Each webhook gets its **own** SSM parameter rather than one shared blob holding all 6. This is the concurrency answer: SSM's `PutParameter` has no compare-and-swap/conditional-write primitive, so a race between two invocations healing at the same time can't be prevented outright — isolating by key means a race on one webhook can never clobber a different webhook's entry. A race on the *same* webhook (two invocations healing it at once) is accepted as low-consequence: last-write-wins between two valid URLs, worst case one harmless orphaned webhook sitting unused in the channel, nowhere near Discord's 15-webhooks-per-channel cap at this account's traffic.

Runtime SSM lookups were confirmed against AWS's published pricing to cost nothing at this account's actual Lambda invocation volume (standard parameter tier, standard throughput — no per-call charge, no version bump or new paid tier needed) before choosing this over keeping deploy-time-only resolution.

## Consequences

### Positive

- No more manual "create it in Discord, paste the URL, redeploy" step when a webhook needs to be (re)created
- Survives a webhook being deleted out from under the bot without a redeploy — it repairs itself on the next send attempt
- Costs nothing at current scale: confirmed against AWS's Parameter Store pricing that standard-tier, standard-throughput API calls are free regardless of volume, and this account's busiest webhook-touching Lambda is nowhere near the shared throughput ceiling

### Negative

- Adds a runtime SSM `GetParameter` (and occasionally `PutParameter`) call to every webhook-touching Lambda's send path, where today it's a zero-latency read of an already-baked env var
- The error-reporting crate, used by roughly two dozen other Lambdas as a zero-setup global (`report_error("msg")`), needs its own lazily-constructed SSM client and bot-token HTTP client to support this — new surface area in what was previously a minimal, dependency-light utility

### Neutral

- Doesn't attempt to fully race-proof concurrent heals (that would need a service with conditional writes, e.g. DynamoDB) — judged unnecessary at current webhook-failure and invocation frequency; revisit if either grows materially
- Discord-visible webhook display names become a naming convention the bot itself enforces (used to detect an already-existing webhook before creating a duplicate)

## Alternatives Considered

- **One shared SSM parameter (JSON blob) for all 6 webhooks:** Rejected — since `PutParameter` has no conditional write, a heal racing on any one webhook's entry risks silently reverting a concurrent change to a different webhook's entry in the same blob. Per-name parameters confine the blast radius of an unavoidable race to the single webhook involved.
- **DynamoDB with a conditional write for true race-proofing:** Rejected as over-engineering for this — no realistic volume of concurrent heals justifies adding another service dependency to avoid a worst case of one harmless orphaned webhook.
- **Keep deploy-time-baked env vars, only consult SSM as a fallback on failure:** Rejected — a healed URL written to SSM wouldn't reach other warm or future Lambda execution environments until the next redeploy, since the env var itself never changes; every invocation needs to resolve from SSM for a heal to actually take effect everywhere, not just in the execution environment that triggered it.

## Implementation Notes

- A shared crate owns webhook resolution and healing by name; both the general webhook-sending code path and the error-reporting crate depend on it rather than duplicating the create-and-persist logic.
- Rollout is staged rather than a single cutover: new SSM parameters and IAM grants land first, Signal seeds the real channel/thread IDs (carrying over each webhook's currently-live URL to avoid an unnecessary first heal), then the code switch to runtime resolution follows, and only then is the old deploy-time env var wiring removed.
- Full design detail — exact crate/module boundaries, SSM parameter naming, the Discord API error shape matched to trigger a heal — lives in the working implementation plan, not duplicated here per ADR convention (this record is the why/what, not the how).
