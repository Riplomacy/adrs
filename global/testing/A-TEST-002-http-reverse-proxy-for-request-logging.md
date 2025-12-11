# TEST-002: HTTP Reverse Proxy for Request Logging

**Date:** 2025-12-11\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

In production, all services are accessed through a single API Gateway endpoint with different paths routing to different services (see INFRA-002). In the Docker testing environment, each service runs its own HTTP server on different ports, creating a mismatch between production and testing architectures. Additionally, debugging service-to-service communication requires visibility into HTTP requests and responses that is not easily available when services communicate directly.

## Decision

Use a custom HTTP reverse proxy (Riplomacy/http-logger) in the Docker environment to:
1. Route requests by path to appropriate service containers (matching production behavior)
2. Log all HTTP requests and responses for troubleshooting
3. Provide a single entry point that mirrors the production API Gateway structure

## Consequences

### Positive

- Testing environment mirrors production routing behavior
- Complete visibility into HTTP traffic between services for debugging
- Single entry point simplifies client configuration in tests
- Full control over functionality and behavior
- Lightweight solution tailored to exact requirements

### Negative

- *None identified*

## Alternatives Considered

- **Direct service communication:** Rejected because it doesn't match production architecture and provides no request visibility
- **Existing reverse proxy logging applications:** Rejected because they couldn't handle the required path-based routing
- **nginx:** Rejected as overly complex and feature-rich for the simple requirements

## Implementation Notes

- Deploy Riplomacy/http-logger as reverse proxy container in Docker Compose
- Configure path-based routing to match production API Gateway routes
- Enable request/response logging for all service communication
- Use single proxy endpoint for all test client requests
