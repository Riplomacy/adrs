# 0008: Service-Owned Routes on the Shared Gateway

**Status:** Superseded by ADR-0024\
**Date:** 2025-12-03\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** INFRA-003

## Decision

Each service owns its routes on the shared API Gateway: it adds them when it's deployed and removes them when it's torn down.

## Context

With one shared gateway serving every service (ADR-0007), routes had to appear and disappear with the services behind them. Managing them centrally would have created deployment dependencies and a bottleneck.

## Options Considered

- **Central route management:** deployment coordination and a single point of management.
- **Static route configuration:** routes would exist even while their service is down.

## Consequences

- Services stay independently deployable, with full control over their own paths.
- Routes reflect which services are actually running, with no central registry.
- Routes from different services can conflict.
- The shared gateway becomes a dependency of every service deployment.
