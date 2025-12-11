# HTTP-001: Path-Based Service Routing

**Date:** 2025-12-11\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

The http-logger serves as a reverse proxy in the Docker testing environment to mirror production API Gateway behavior (see TEST-002). Services need to be accessible through a single endpoint with different URL paths routing to different backend services, matching the production architecture defined in INFRA-002.

## Decision

Implement path-based service routing using a TOML configuration file that maps URL prefixes to backend service targets. Each service mapping defines:
- `prefix`: The URL path prefix to match (e.g., "users", "content")
- `target`: The backend service URL to forward requests to
- `strip_prefix`: Optional flag to remove the prefix before forwarding

## Consequences

### Positive

- Mirrors production API Gateway routing behavior in testing
- Simple, declarative configuration
- Easy to add or modify service routes
- Clear visibility of all service mappings at startup

### Negative

- *None identified*

## Alternatives Considered

- **Hardcoded routes:** Rejected as inflexible and requiring code changes for new services
- **Dynamic service discovery:** Rejected as unnecessarily complex for testing environment

## Implementation Notes

- Configuration loaded from `mapping.toml` at startup
- Service mappings printed to console for visibility
- First matching prefix wins (order matters)
- Requests with no matching service return 404
