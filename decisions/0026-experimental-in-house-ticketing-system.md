# 0026: Experimental In-House Ticketing Service

**Status:** Accepted\
**Date:** 2026-09-13\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** TICKET-001

## Decision

Ticketing is built in-house as a new Smithy-backed Ticket Service, run alongside the existing third-party ticket bot as an opt-in experiment, not a cutover.

- The Ticket Service owns the whole ticket lifecycle and its category-specific workflows. The Discord handler stays a thin front door, with no ticket logic of its own.
- Ticket state lives in the ticket channel itself, not in a database.
- Closing a ticket is immediate: the transcript is archived to storage and the channel deleted in one step.
- The first workflow handles unban requests, and can lift automatic teamkill bans itself under a safety cap. This gives the bot write access to BattleMetrics bans for the first time.

The full design is in `riplomacy_bot`'s ticketing system design document.

## Context

The third-party ticket bot can't route ticket categories to the staff teams that own them, and doesn't integrate with this bot's staff roster, permissions, or BattleMetrics and identity data. The gap is in features, not cost or reliability.

## Options Considered

- **Delayed, cancellable close:** a second piece of infrastructure and a stuck-state recovery story for a nice-to-have. Revisit if immediate close proves too abrupt.
- **Automatically close a ticket after a successful auto-unban:** removes the last chance to catch a wrong automated match.
- **Ticket logic in the Discord handler:** breaks the rule that the interaction handler holds no domain logic.
- **Private threads instead of channels:** avoids Discord's channel caps, but the per-ticket channel model was preferred.
- **A table for ticket state:** every relevant interaction already identifies the channel, so the channel itself is the natural store.

## Consequences

- Closes a real routing and integration gap without staking community support on day-one parity with the existing bot.
- Category workflows, starting with auto-unban, are what the experiment actually tests.
- Ban mutation is a new trust boundary for the bot.
- The one-open-ticket-per-category check is best effort, so duplicates are possible.
- An accidental or premature close can't be undone.
- Two ticket systems run side by side for the duration of the experiment; retiring the old bot is a separate, later decision.
