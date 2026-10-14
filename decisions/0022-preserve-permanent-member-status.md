# 0022: Preserve Permanent Member Status

**Status:** Deprecated 2026-10-14 -- a behavior rule, not architecturally significant (ADR-0034)\
**Date:** 2025-12-14\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** WL-002

## Decision

Adding a timed membership for someone who already has a permanent membership keeps them permanent. The record is migrated to the new membership format either way.

## Context

With timed memberships (ADR-0021), the seeder webhook adds a 7-day membership every time it fires. That raised the question of what happens when it fires for someone who already has permanent access.

## Options Considered

- **Convert to timed:** removes earned permanent access.
- **Leave permanent members in the legacy format:** prevents unifying the data format.
- **Reject the add:** would break seeder automation.

## Consequences

- Permanent members never lose their status because of seeder activity.
- Legacy records migrate to the new format gradually.
- A seeder request for 7 days can silently result in permanent status.
