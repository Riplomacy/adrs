# 0030: Game Qualifications Decouple Staff Rank from Game

**Status:** Accepted (rank ladder amended by ADR-0031)\
**Date:** 2026-09-25\
**Deciders:** Jean-Sébastien Dominique, Praetorian\
**Scope:** `riplomacy_bot`\
**Converted from:** STAFF-002

## Decision

The staff model separates seniority from game assignment. A staff member's rank expresses Riplomacy-wide seniority. A game qualification, one per game, expresses which games they administer. Their effective access is their rank combined with each game they hold, and inactive ranks suspend game access without removing qualifications.

## Context

A staff member's rank determined everything they received: Discord roles, BattleMetrics roles and flags, and their whitelist group. Every rank was implicitly a Squad rank. Riplomacy now staffs Squad, Wardogs and Hell Let Loose, but a Wardogs-only admin couldn't be represented, and staff who admin several games had no way to say so. Discord already separates community-wide roles from game-specific ones; the bot only managed the Squad side.

## Options Considered

- **A separate rank per game:** the most expressive, but a data model change and a rework of every staff flow, for per-game seniority nobody needs.
- **Game-specific ranks within the single rank:** configuration only, but can't represent someone who admins two games, which describes most admins.

## Consequences

- Staff for every game become first-class roster members, and anything routed on staff roles works per game.
- One rank ladder for the whole community; adding a game adds no ranks.
- In-game permissions depend on two inputs, so a qualification change must resynchronize them like a rank change does.
- Every existing staff record needed a one-time migration.
- Trainers can grant qualifications without a second approver, which widens what they can do alone.
- Non-Squad staff get Discord roles only, until the bot supports those servers.
