# WL-002: Preserve Permanent Member Status During Timed Membership Migration

**Date:** 2024-12-14\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

With the implementation of timed group memberships (WL-001), a question arose about what should happen when the seeder API attempts to add a 7-day timed membership to a player who already has permanent whitelist access. The system needed to decide between:

1. Converting permanent members to 7-day timed members
2. Preserving permanent status while migrating to new data format
3. Keeping permanent members in legacy format unchanged

## Decision

When adding a timed membership to an existing permanent member, the system will:

1. **Migrate to new format**: Move from legacy data structure to new timed membership structure
2. **Preserve permanent status**: Maintain permanent access regardless of requested expiry duration
3. **Update database**: Ensure the migration is saved to the database
4. **Maintain access**: User retains permanent whitelist access

## Consequences

### Positive

- **Preserves user privileges**: Permanent members don't lose their status due to seeder activity
- **Consistent data format**: All active memberships eventually migrate to new structure
- **Predictable behavior**: Permanent status is never downgraded, only preserved or upgraded
- **Database modernization**: Legacy data is gradually migrated without breaking changes

### Negative

- **Complex logic**: Migration logic is more sophisticated than simple replacement
- **Mixed expectations**: Seeder API requests 7-day expiry but gets permanent status

### Neutral

- **One-way migration**: Legacy permanent → new permanent (no reverse migration)
- **Seeder API behavior**: Still returns success even when expiry is not applied

## Alternatives Considered

- **Convert to timed**: Would have downgraded permanent members to 7-day access. Rejected as it removes earned privileges.
- **No migration**: Keep permanent members in legacy format. Rejected as it prevents data format unification.
- **Reject operation**: Return failure when trying to add timed membership to permanent member. Rejected as it would break seeder automation.

## Implementation Notes

- Migration occurs during any add operation, not just seeder API calls
- Database update ensures migration is persisted even when status unchanged
- Applies to both Steam ID and Discord ID memberships
