# ADR-2512007: Use Docker for Staging Environment

**Date:** 2025-12-03\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

Initially attempted to use a separate AWS account for staging environment within AWS Organizations. However, AWS free tier limits are shared across all accounts in an organization, causing production and staging environments to compete for free tier resources. This resulted in unexpected costs when DynamoDB usage exceeded free tier limits across both environments.

## Decision

Use Docker-based local staging environment instead of separate AWS account. Run Lambda functions in Docker containers with mocked external services and local DynamoDB for testing and development.

## Consequences

### Positive

- Eliminates unexpected AWS costs from shared free tier limits
- Faster development cycle without AWS deployment delays
- Complete control over test environment and data
- Enables automated integration testing locally

### Negative

- Significant project overhead to containerize Lambda functions and mock services
- Local environment may not perfectly match AWS runtime behavior
- Additional maintenance for Docker setup and mocks
- Not all AWS services are available for local Docker use

## Alternatives Considered

- **Separate AWS staging account in organization:** Rejected due to shared free tier limits causing unexpected costs
- **Separate AWS account outside organization:** Rejected due to potential doubling of costs
- **Shared AWS account with resource prefixes:** Rejected due to potential production data contamination

## Implementation Notes

- Containerize Lambda functions for local execution
- Use local DynamoDB for data storage in staging
- Mock external service dependencies
- Implement automated integration test suite
- Maintain Docker environment alongside AWS deployments
