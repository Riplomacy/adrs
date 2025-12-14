# WL-001: Timed Group Membership for Seeder Whitelist Management

**Date:** 2024-12-14\
**Status:** Proposed\
**Deciders:** Jean-Sébastien Dominique

## Context

The current whitelist system creates permanent entries for seeders via BattleMetrics webhooks. The existing system to remove seeders who no longer meet requirements is imperfect and results in an accumulation of over 400 seeders when the active seeder count should be much smaller. Inactive seeders remain whitelisted indefinitely, creating maintenance overhead.

## Decision

Implement timed group membership where group memberships can have expiration dates. When seeder webhooks trigger, users will be added to the seeder group with a 7-day expiration. Subsequent webhook triggers for the same user will refresh the expiration to 7 days from the trigger time.

The whitelist logic remains unchanged - it continues to check group membership. A periodic cleanup process (hourly) will remove expired group memberships. The current seeder removal system will be replaced by this automatic expiration mechanism.

## Consequences

### Positive

- Automatically maintains an accurate list of active seeders
- Reduces whitelist bloat from inactive seeders
- No changes required to existing whitelist validation logic
- Provides foundation for other timed group features
- Simplifies seeder management by removing complex removal logic

### Negative

- Adds complexity to group membership data model
- Requires new cleanup process for expired memberships

### Neutral

- Group membership becomes more dynamic rather than static
- Seeder status now requires periodic webhook activity to maintain
- Replaces existing removal system with time-based approach

## Alternatives Considered

- **Timed Whitelist Entries:** Modify whitelist system directly to support expiration dates. Rejected because it would require changes to core whitelist logic and affect all whitelist types.
- **Manual Seeder Management:** Require staff to manually remove inactive seeders. Rejected due to high maintenance overhead.

## Implementation Notes

- Add `expires_at` field to group membership records (nullable for permanent memberships)
- Modify seeder webhook handler to set 7-day expiration on group membership
- Create hourly cleanup Lambda to remove expired memberships
- Ensure webhook handler updates existing membership expiration rather than creating duplicates
- Use ISO 8601 datetime format for expiration timestamps
- Remove existing seeder removal system logic
