# HTTP-002: Optional Prefix Stripping

**Date:** 2025-12-11\
**Status:** Superseded\
**Deciders:** Jean-Sébastien Dominique

**Superseded by:** DEV-008 (services now include prefix in Smithy models)

## Context

Services in the Docker environment are configured with endpoints that include the service prefix (e.g., `http://http-logger/users`) to match how clients interact with the API. However, the backend services themselves don't expect the prefix in their routes - they expect `/contacts` not `/users/contacts`. This is because service prefixes are configured at the infrastructure level rather than in the service code itself.

## Decision

Add an optional `strip_prefix` configuration flag to service mappings. When enabled, the proxy removes the matched prefix from the request path before forwarding to the backend service.

Example:
- Request: `POST /users/contacts`
- Prefix: `users`
- With `strip_prefix = true`: Forwards as `POST /contacts`
- With `strip_prefix = false`: Forwards as `POST /users/contacts`

## Consequences

### Positive

- Backend services receive paths they expect
- Flexible configuration for services that need different behaviors
- Clients can use consistent endpoint format

### Negative

- May need adjustment when deploying to production depending on API Gateway behavior

## Alternatives Considered

- **Always strip prefix:** Rejected as some services might need the full path
- **Always keep prefix:** Rejected as it doesn't match current service expectations
- **Require services to handle both:** Rejected as it duplicates routing logic in services

## Implementation Notes

- Stripping happens before forwarding the request to the backend
- Only the matched prefix is removed, rest of path remains intact
