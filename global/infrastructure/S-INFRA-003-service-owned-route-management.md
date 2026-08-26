# INFRA-003: Service-Owned Route Management

**Date:** 2025-12-03\
**Status:** Superseded by INFRA-008 (no shared gateway left to route on; each service owns a Function URL instead)\
**Deciders:** Jean-Sébastien Dominique

## Context

With a single shared API Gateway serving all services through path-based routing, route management becomes a critical concern. Routes need to be dynamically added when services are deployed and removed when services are taken down. Centralized route management would create deployment dependencies and bottlenecks.

## Decision

Each service owns and manages its own routes in the shared API Gateway. Services add their routes during deployment and remove them during teardown, ensuring routes are only available when the backing service is actually running.

## Consequences

### Positive

- Services remain independently deployable without coordination
- Routes automatically reflect actual service availability
- No central route registry to maintain
- Services have full control over their API paths

### Negative

- Potential for route conflicts between services
- Shared API Gateway becomes a dependency for all service deployments

## Alternatives Considered

- **Centralized route management:** Rejected due to deployment coordination overhead and single point of management
- **Static route configuration:** Rejected because routes would exist even when services are down

## Implementation Notes

- Services use CloudFormation/CDK to manage their API Gateway routes
- Implement route naming conventions to prevent conflicts (e.g., `/{service-name}/*`)
- Use CloudFormation stack dependencies to ensure API Gateway exists before service deployment
- Consider route validation during deployment to catch conflicts early
