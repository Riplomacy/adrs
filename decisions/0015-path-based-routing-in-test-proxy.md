# 0015: Path-Based Routing in the Test Proxy

**Status:** Deprecated 2026-10-14 -- a component design detail of ADR-0018, not architecturally significant (ADR-0034)\
**Date:** 2025-12-11\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `http-logger`\
**Converted from:** HTTP-001

## Decision

The test proxy routes requests to backend services by URL path prefix, using a mapping declared in configuration.

## Context

The test proxy (ADR-0018) mirrors the production gateway's path routing (ADR-0007), so services have to be reachable through a single endpoint, with each path prefix routed to a different backend.

## Options Considered

- **Hardcoded routes:** a code change for every new service.
- **Dynamic service discovery:** unnecessary complexity for a test environment.

## Consequences

- Test routing mirrors production, with a simple declarative mapping that's easy to change.
