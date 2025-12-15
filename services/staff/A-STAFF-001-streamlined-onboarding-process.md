# STAFF-001: Streamlined Staff Onboarding Process

**Date:** 2025-12-14\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

Currently, staff onboarding to BattleMetrics requires manual coordination:
1. New staff member DMs trainer with email address
2. Trainer invites them via bot to BattleMetrics
3. When they join, notification posted to Discord channel
4. Trainer manually confirms identity (often forgotten)
5. New staff member receives their permissions

This process has privacy concerns (email shared via DMs), requires constant trainer availability for coordination, and the confirmation step is frequently overlooked, leaving new staff without proper permissions.

## Decision

Implement self-service onboarding with automatic correlation:

### Process Flow
1. Trainer adds person as staff member (existing process)
2. New staff member runs Discord command to register their email
3. System stores email address associated with staff member
4. Trainer triggers BattleMetrics invitation (existing process, now uses stored email)
5. When BattleMetrics member joins:
   - **Single pending invitation:** Auto-confirm identity and grant permissions
   - **Multiple pending:** Require trainer confirmation to resolve ambiguity
6. Permissions granted automatically upon identity confirmation

### Technical Components
- Staff email storage in database
- Pending invitations tracking with timestamps
- Discord slash command for email self-registration
- Automatic invitation correlation logic
- Fallback manual confirmation interface
- Automatic permission granting upon confirmation

## Consequences

### Positive
- Eliminates email sharing via DMs (privacy improvement)
- Reduces trainer workload for unambiguous cases
- Self-service approach reduces coordination overhead
- Maintains audit trail of all invitations
- Faster permission granting for single pending invitations
- Eliminates forgotten confirmation steps

### Negative
- Still needs manual intervention for multiple concurrent invitations
- Email addresses stored in database (privacy concern)

## Alternatives Considered

- **Keep current manual process:** Still requires trainer availability and coordination

## Implementation Notes

- Store emails with pending invitation timestamps for correlation
- Support email updates if initial invitation fails
- Auto-delete emails after invitation expires (3 days) or is fulfilled
- Permissions granted immediately upon identity confirmation
