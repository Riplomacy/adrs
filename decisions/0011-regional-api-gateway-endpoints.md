# 0011: Regional API Gateway Endpoints

**Status:** Accepted\
**Date:** 2025-12-03\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** INFRA-006

## Decision

API Gateway endpoints are Regional rather than edge-optimized.

## Context

API Gateway defaults to edge-optimized endpoints, distributed globally through CloudFront. Our APIs are called by external service integrations and by services in the same region, not directly by end users.

## Options Considered

- **Edge-optimized:** no benefit when users don't call the APIs directly.

## Consequences

- Lower latency for every API call.
- Custom domain setup and IAM authorization are unaffected.
