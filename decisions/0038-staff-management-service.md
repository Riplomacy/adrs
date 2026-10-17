# 0038: Staff Management Service

**Status:** Accepted\
**Date:** 2026-10-17\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`

## Decision

We will move staff management into a new staff management service. It decides who may change whom, creates staff, changes ranks and qualifications, and applies what a rank grants in Discord and BattleMetrics. The Discord commands and the admin console become thin callers that say which staff member is acting, and the service enforces the management rules. The staff data service stays a plain record store.

## Context

All of this lives in the Discord interaction handler today. A rank change checks that the caller may manage the target and the destination rank, saves the record with its promotion history, then brings Discord roles, BattleMetrics organization roles and BattleMetrics player flags in line with the new rank. Those last steps are best effort: failures come back to the caller as notes, and a promotion that grants player flags is refused when the member has no linked game account. The rules from ADR-0030 (game qualifications) and ADR-0031 (the moderator/admin ladder and rank aliases) are enforced there too.

The admin console (ADR-0035) needs to edit staff. It can't reuse handler code, and writing to the staff record store directly would change a rank without changing the access that comes with it. ADR-0035 also committed to keeping business rules in services so that both interfaces behave the same.

## Options Considered

- **Put the orchestration in the staff data service:** mixes record storage with calls to external systems. The data service would need the bot token and BattleMetrics credentials, and every reader of staff records would depend on a service with external side effects.
- **A shared library both callers link:** each caller would need Discord and BattleMetrics credentials, and the console's backend would gain the bot's powers. The rules would be enforced only by each caller using the library correctly.
- **Keep staff editing in Discord only:** the console stays read-only for staff, and the logic stays in the entry point.

## Consequences

- One implementation of the management rules. Discord and the console behave the same, and the console can edit staff.
- The new service holds the Discord bot token and BattleMetrics credentials for staff changes. The console's backend never gets them.
- The service trusts its caller to name the acting staff member. Callers are authenticated with IAM (ADR-0024), so a compromised caller could act as anyone, which is no worse than the handler today.
- Partial failures in Discord or BattleMetrics are reported to the caller rather than rolled back, as today.
- A staff change waits on Discord and BattleMetrics calls inside the service, so interactive Discord flows must keep deferring their replies (ADR-0027).
- A sizable move of logic and tests out of the handler, and one more service to deploy and run.

## Confidence

Medium to high. Revisit if staff changes ever need all-or-nothing behavior across Discord, BattleMetrics and the record, or if an entry point needs different rules than the others.
