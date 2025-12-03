# ADR-2512012: Regional API Gateway Endpoints

**Date:** 2025-12-03\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

API Gateway defaults to Edge-optimized endpoints which use CloudFront for global distribution. Our APIs are consumed by external service integrations and internal services within the same AWS region, not directly by end users.

## Decision

All API Gateway endpoints will use Regional configuration instead of Edge-optimized endpoints.

## Consequences

### Positive

- Reduced latency (10-50ms savings) for all API calls

### Negative

- *None identified for our use case*

### Neutral

- Custom domain configuration remains the same
- IAM authentication and authorization unchanged

## Alternatives Considered

- **Edge-optimized:** Rejected as users don't interact directly with APIs

## Implementation Notes

- Configure all RestApi constructs to use Regional endpoint type instead of the default Edge-optimized configuration
