# ADR-2512006: Use Smithy for Service Modeling

**Date:** 2025-12-03\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

Service-to-service communication has evolved through multiple approaches. SNS-SQS-Lambda was too expensive due to SQS costs. SNS-Lambda reduced costs but added latency and only supported asynchronous communication. Lambda-Lambda direct invocation provides fast, cheap, reliable communication with synchronous response capability, but requires manually building and maintaining clients for each service API.

## Decision

Use Smithy to model all service APIs and generate clients automatically. This provides synchronous HTTPS communication through API Gateway while eliminating the overhead of manual client development and maintenance.

## Consequences

### Positive

- Automated client generation eliminates manual API client development
- Type-safe service contracts reduce integration errors
- Consistent API patterns across all services
- Multi-language client generation capability available if needed
- Service discovery through API Gateway endpoints instead of Lambda function names

### Negative

- Learning curve for Smithy IDL and tooling
- Additional build step for client generation

## Alternatives Considered

- **SNS-SQS-Lambda:** Rejected due to high SQS costs
- **SNS-Lambda:** Rejected due to latency and lack of synchronous responses
- **Manual Lambda-Lambda clients:** Rejected due to maintenance overhead

## Implementation Notes

- Define service APIs using Smithy IDL
- Generate Rust clients for service-to-service communication
- Use generated clients with HTTPS calls to API Gateway endpoints
- Integrate client generation into build process
- Maintain service contracts in version control alongside implementation
