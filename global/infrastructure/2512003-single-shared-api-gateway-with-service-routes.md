# ADR-2512003: Use Single Shared API Gateway with Service Routes

**Date:** 2025-12-03\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

Original plan was one Private API per service with individual subdomains for clean addressing. Since switching to Public APIs, Private Hosted Zones are not available. Public Hosted Zones cost $0.50/month each, and API Gateway Custom Domain Names also cost $0.50/month each. With multiple services, using one subdomain per service would result in $0.50/month base cost plus $0.50/month per service for custom domain names.

## Decision

Use a single Public Hosted Zone with one API Gateway Custom Domain Name. Route all services through a single shared API Gateway using path-based routing (e.g., `/user-service/*`, `/whitelist-service/*`) instead of subdomain-based routing.

## Consequences

### Positive

- Fixed $1.00/month total cost regardless of service count
- Eliminates per-service $0.50/month custom domain costs
- Simplified DNS management with single domain
- Centralized API Gateway infrastructure with distributed route management

### Negative

- Less clean service addressing compared to subdomains
- All services share single API Gateway limits and quotas
- Single point of failure for all service APIs

### Neutral

- Service discovery shifts from DNS-based to path-based
- Services manage their own routes during deployment

## Alternatives Considered

- **One subdomain per service:** Rejected due to $0.50/month per service cost scaling
- **Service-specific API Gateways without custom domains:** Rejected due to complex AWS-generated URLs

## Implementation Notes

- Configure single shared API Gateway with custom domain
- Services add their own path-based routes during deployment
- Use `/{service}/*` pattern for service-specific routing
- Implement service-specific base paths in Smithy-generated clients
