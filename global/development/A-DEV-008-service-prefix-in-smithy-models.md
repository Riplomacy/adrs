# DEV-008: Include Service Prefix in Smithy Models

**Date:** 2025-12-11\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

API Gateway passes the full request path to Lambda functions, including the service prefix (e.g., `/user-data-service/contacts`). While it's possible to configure API Gateway to strip prefixes using custom request mapping templates, this adds complexity. Services need to handle the paths they actually receive from API Gateway.

## Decision

Include the service prefix in Smithy model route definitions. Services will handle paths that include their prefix (e.g., `/user-data-service/contacts` instead of just `/contacts`).

## Consequences

### Positive

- Smithy models match actual API Gateway behavior
- No custom API Gateway configuration needed
- Clear and explicit about full paths services handle

### Negative

- Service prefix appears in route definitions

## Alternatives Considered

- **Strip prefix at infrastructure level:** Rejected as it requires custom API Gateway request mapping templates
- **Strip prefix in service code:** Rejected as it duplicates logic and doesn't match Smithy model definitions
- **Handle both with/without prefix:** Rejected as unnecessarily complex

## Implementation Notes

- Update all Smithy models to include service prefix in route paths
- Ensure consistency between Smithy definitions and actual API Gateway paths
