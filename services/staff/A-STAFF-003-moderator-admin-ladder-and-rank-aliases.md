# STAFF-003: Moderator/Admin Ladder and Rank Aliases for Current Records Only

**Date:** 2026-09-25\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique, Praetorian

Amends STAFF-002 (rank ladder only; game qualifications stand as decided there).

## Context

STAFF-002 named the ladder Trial → Admin and retired the Moderator rank, folding it into Admin.
Before deploying, a look at the current staff records showed that decision didn't match how the
ranks were really used:

- Five current members are stored under an old trial-tier name that the bot treated as
  Moderator. Most of them are trials in practice; folding Moderator into Admin would have given
  them admin cam.
- No current member is stored as Moderator at all; that name only appears in promotion history.
- Several other old rank names are kept around as aliases even though they only appear in
  promotion history.

Rank aliases exist so a stored rank name keeps resolving after a rename. Promotion history is
displayed exactly as recorded, so an old name appearing there needs no alias.

## Decision

- **The active ladder is Moderator → Admin → Tech Team → Owner.** Moderator is the
  untrusted-yet tier (no admin cam) that STAFF-002 called Trial; Admin is fully trusted. Game
  qualifications combine with Moderator or Admin exactly as STAFF-002 describes.
- **Awaiting Training and Idle are inactive states, not ladder steps.** Neither sits above the
  other; both suspend game access as STAFF-002 describes.
- **Everyone currently at the trial tier becomes Moderator**, including the members stored
  under the old trial-tier names. Those names become aliases of Moderator, so no records are
  rewritten.
- **Aliases are kept only for rank names stored on current staff records.** Names that survive
  only in promotion history are not aliased; history keeps showing them as recorded, while the
  current rank is shown by its resolved name.
- **Before removing a rank or alias, check the current records.** An automated check guards the
  stored names known at the time of this decision.

## Consequences

### Positive

- Nobody gains or loses admin cam as a side effect of the migration.
- No staff records need rewriting; promotion history stays an accurate record of what was
  recorded at the time.
- Fewer aliases, each with a clear reason to exist.

### Negative

- Removing an alias later requires checking live data, not just configuration.

### Neutral

- Moderators who are ready for admin cam are promoted to Admin individually by trainers, which
  records the promotion in their history.

## Alternatives Considered

- **Keep STAFF-002's Trial → Admin with Moderator folded into Admin:** grants admin cam to
  members who are trials in practice.
- **Rewrite stored rank names to the new ones:** loses nothing functionally but adds a data
  migration that aliases make unnecessary, and makes the stored rank disagree with the history
  entry that set it.

## Implementation Notes

- The Trial Squad Moderator Discord role is retired; Squad Moderator is the Moderator + Squad
  role.
- Members holding the Trial Squad Moderator role move to Squad Moderator on their next staff
  update.
