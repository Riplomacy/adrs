# 0007: Single Shared API Gateway with Path-Based Service Routes

**Status:** Superseded by ADR-0024\
**Date:** 2025-12-03\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** INFRA-002

## Decision

All internal services share one API Gateway with one custom domain, routed by path prefix per service rather than by subdomain.

## Context

The original plan was one private API per service, each on its own subdomain. After moving to public APIs (ADR-0006), private hosted zones were no longer available. Public hosted zones and API Gateway custom domains each cost $0.50/month, so a subdomain per service would grow the cost with every service.

## Options Considered

- **One subdomain per service:** cost scales with the number of services.
- **A gateway per service without custom domains:** unwieldy generated URLs.

## Consequences

- A fixed cost of about $1/month regardless of service count, and one domain to manage.
- Service discovery becomes path-based.
- All services share one gateway's limits and quotas, and it's a single point of failure.
