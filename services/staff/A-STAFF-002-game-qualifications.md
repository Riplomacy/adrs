# STAFF-002: Game Qualifications Decouple Staff Rank from Game

**Date:** 2026-09-25\
**Status:** Accepted (rank ladder amended by STAFF-003)\
**Deciders:** Jean-Sébastien Dominique, Praetorian

## Context

A staff member's rank currently determines everything they receive: Discord roles,
BattleMetrics roles and flags, and their whitelist group. Every rank is implicitly a Squad
rank -- holding one means "Squad staff at this level".

Riplomacy now staffs more than one game: Squad, Wardogs, and Hell Let Loose. A Wardogs-only
admin can't be represented in the staff roster, so nothing the bot manages -- Discord roles,
and ticket visibility built on them -- can be routed to them. Staff who admin several games
have no way to say so either.

Discord already separates the two: community-wide roles (Riplomacy Admin) sit alongside
game-specific ones (Squad Staff, Squad Admin, Trial Squad Moderator, Wardogs Staff, Wardogs
Admin, HLL Admin). The bot only manages the Squad side.

The Trial → Moderator → Admin ladder was meant as "getting to know you" → "trusted, but no
admin cam" → "fully trusted". In practice the Moderator step is barely used (two members), and
other games only have an Admin tier. The meaningful distinction left is admin cam access,
which depends on both seniority and game.

## Decision

Separate seniority from game assignment:

- **Ranks express Riplomacy-wide seniority.** The ladder becomes AwaitingTraining → Idle →
  Trial → Admin → TechTeam → Owner. The Moderator rank is retired. Ranks keep what is
  genuinely game-independent: level, friend allowance, management rights, and community-wide
  roles. Every active rank, Trial included, makes the holder Riplomacy staff.
- **Game qualifications express which games a staff member administers.** There is one per
  game (Squad, Wardogs, Hell Let Loose). A game qualification grants that game's staff role,
  plus the game-specific tier roles and in-game permissions matching the holder's rank --
  Trial + Squad is a Trial Squad Moderator without admin cam, Admin + Squad is a Squad Admin
  with it.
- **A staff member's effective access is their rank combined with each game they hold.**
  Inactive ranks (AwaitingTraining, Idle) suspend game access without removing the
  qualifications, so returning from Idle restores it.
- **Trainers grant game qualifications, including to themselves.** Game qualifications are
  self-manageable; other qualifications keep the existing rule that staff can't manage
  themselves.
- **Game qualifications are not promotions.** They carry no history beyond what adding and
  removing a qualification already records.
- **Wardogs and Hell Let Loose get Discord roles only for now.** In-game permission
  management stays Squad-only until the bot supports those servers.

## Consequences

### Positive

- Wardogs and Hell Let Loose staff become first-class roster members, and anything routed on
  staff roles (such as ticket categories) works per game.
- Multi-game staff are expressed directly instead of implied.
- One rank ladder for the whole community; adding a game doesn't add ranks.
- Trainers can bring staff into a game without an additional approver.

### Negative

- In-game permissions now depend on two inputs, so changing a game qualification must
  resynchronize permissions the same way a rank change does.
- Every existing staff record needs a one-time migration.
- Self-management widens what trainers can do on their own, by design.

### Neutral

- Wardogs and Hell Let Loose staff get Discord roles but no bot-managed in-game permissions
  until the bot supports those servers.
- Hell Let Loose access moves from "every staff member" to opt-in; staff who want it are
  given the qualification.
- Qualification changes aren't tracked with the same rigor as rank history.

## Alternatives Considered

- **A separate rank per game:** the most expressive model, but a data model change and a
  rework of every staff management flow, for per-game seniority nobody needs.
- **Game-specific ranks (e.g. Wardogs Admin) in the existing single rank:** configuration
  only, but can't represent someone who admins two games -- which describes most Riplomacy
  admins.

## Implementation Notes

- Existing staff migrate to the new ladder plus a Squad qualification: Trial Moderators to
  Trial, Moderators and Admins to Admin, Tech Team to Tech Team. Moderator survives as a
  former name of Admin so existing records still resolve.
- Wardogs staff are given the Wardogs qualification, and Wardogs-only admins are added to the
  roster. Hell Let Loose is opt-in: staff who want it are given the qualification.
- The bot's notion of supported games needs to include Wardogs.
- Ticket categories route to each game's staff role, which lets Wardogs ticket categories
  route to Wardogs staff.
