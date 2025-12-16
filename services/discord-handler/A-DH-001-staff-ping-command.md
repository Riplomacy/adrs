# DH-001: Staff Ping Command

**Date:** 2025-12-16\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

Staff members frequently need to contact head admins and tech team members for assistance but currently lack a direct way to ping these roles through Discord. The current system forces staff to either:

1. Ping individual users repeatedly until someone responds
2. Use workarounds like pinging unrelated staff members who then relay the message
3. Wait indefinitely for help without any escalation mechanism

This creates inefficiencies in staff operations and can lead to delayed responses for critical issues. A tech request thread (ID: 1404955946727510169) specifically highlighted this need, with staff requesting "a command that lets you ping someone / a group based on staff rank."

## Decision

Add a staff escalation mechanism to the Discord handler service that allows authorized users to ping specific staff groups based on organizational hierarchy. This will be implemented as a `/ping-staff` Discord slash command that accepts a staff group parameter and sends notifications to the appropriate personnel.

The command will not accept user-provided text content to avoid content moderation responsibilities.

## Consequences

### Positive

- Staff can efficiently escalate issues to appropriate personnel
- Reduces spam from repeated individual pings
- Improves response times for critical staff issues
- Scales better than manual ping management
- Avoids content moderation concerns

### Negative

- May increase notification volume for targeted staff groups
- No contextual information provided with pings

## Alternatives Considered

- **Manual role pings:** Continue current system - rejected due to being disabled to prevent non-staff abuse
- **Dedicated support channels:** Create separate channels for each staff level - rejected as it fragments communication

## Implementation Notes

- Consider role-based access control for command usage
- Rate limiting and audit logging should be considered
