# 0018: Logging Reverse Proxy in the Test Environment

**Status:** Accepted\
**Date:** 2025-12-11\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot` test environment\
**Converted from:** TEST-002

## Decision

The Docker test environment puts a custom reverse proxy in front of the services. It routes requests by path, mirroring the production gateway, logs every request and response, and gives tests a single entry point.

## Context

In production, every service sat behind one API Gateway, routed by path (ADR-0007). In Docker, each service ran its own HTTP server on its own port, so the test architecture didn't match production. Debugging service-to-service calls also needed visibility into requests and responses that direct calls didn't provide.

## Options Considered

- **Services call each other directly:** doesn't match production, and gives no request visibility.
- **An existing logging proxy:** couldn't do the path-based routing needed.
- **nginx:** far more than these requirements need.

## Consequences

- Test routing mirrors production, and all traffic between services is visible for debugging.
- A single entry point simplifies test configuration.
- A small custom tool to own and maintain.
