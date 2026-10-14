# 0021: Timed Group Membership

**Status:** Proposed\
**Date:** 2025-12-14\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** WL-001

## Decision

Group memberships can carry an expiry. Seeders are added to the seeder group with an expiring membership, refreshed each time they seed again, and a periodic process removes expired memberships. Whitelist assembly keeps checking group membership as before.

## Context

Seeders were whitelisted permanently through BattleMetrics webhooks. The process for removing seeders who no longer qualified was unreliable, so over 400 seeders accumulated when the real number of active seeders was much smaller. Inactive seeders stayed whitelisted indefinitely.

## Options Considered

- **Expiring whitelist entries:** would change the core whitelist logic and affect every whitelist type.
- **Manual seeder removal by staff:** high ongoing maintenance.

## Consequences

- The seeder list maintains itself, and whitelist bloat from inactive seeders goes away.
- Whitelist validation is unchanged.
- Gives other features a foundation for time-limited membership.
- The membership data model gets more complex, and expired memberships need a cleanup process.
- Staying a seeder now depends on seeding regularly.
