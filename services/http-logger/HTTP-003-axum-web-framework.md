# HTTP-003: Axum Web Framework

**Date:** 2025-12-11\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

Building an HTTP reverse proxy requires handling HTTP requests and responses. While Hyper provides low-level HTTP primitives, it requires significant boilerplate for common web server patterns. A higher-level framework simplifies development while maintaining performance.

## Decision

Use Axum as the web framework for http-logger. Axum provides ergonomic routing, middleware, and request handling built on top of Hyper and Tokio.

## Consequences

### Positive

- Familiar framework with clear patterns
- Less boilerplate than raw Hyper
- Built on proven Hyper/Tokio foundation
- Ergonomic request/response handling

### Negative

- *None identified*

## Alternatives Considered

- **Hyper directly:** Rejected as too low-level and complex for this use case
- **Other frameworks (Actix, Rocket, etc.):** Not considered due to existing familiarity with Axum
