# INFRA-001: Use Public API Gateway with IAM Restrictions

**Date:** 2025-12-03\
**Status:** Accepted (IAM implementation updated by INFRA-007)\
**Deciders:** Jean-Sébastien Dominique

## Context

Internal service communication requires API Gateway endpoints. Best practices recommend Private APIs for internal services, but this creates significant cost and complexity issues for the Riplomacy Bot project. Private APIs require Lambda functions in VPCs, VPC Endpoints (~$7/month), and NAT Gateways (~$45/month) for internet access. Total additional cost would be ~$52/month for proper private networking.

## Decision

Use Public API Gateway endpoints with IAM-based access restrictions instead of Private APIs. Configure resource policies to only allow a single shared IAM role that all internal services assume, effectively creating internal-only access without VPC complexity.

## Consequences

### Positive

- Eliminates VPC Endpoint costs (~$7/month savings)
- Eliminates NAT Gateway costs (~$45/month savings)
- Lambda functions remain in default service VPC with internet access
- Simpler networking architecture and deployment
- Maintains security through IAM authentication
- Zero maintenance overhead with single shared role approach
- Seamless integration with Smithy-generated service code

### Negative

- API endpoints are technically public (though IAM-protected)
- Slightly less defense-in-depth compared to network isolation

### Neutral

- Security model shifts from network-based to identity-based
- Single role provides uniform access control across all services

## Alternatives Considered

- **Private API Gateway:** Rejected due to $52/month additional infrastructure costs
- **Direct Lambda invocation:** Rejected due to deployment complexity with Smithy-generated services
- **Application Load Balancer:** Rejected due to higher costs and complexity

## Implementation Notes

- Create single shared IAM role for all internal service communication
- Configure API Gateway resource policy with single role ARN condition
- All Lambda functions and services assume the shared role for API calls
- Implement request signing using AWS SDK with assumed role credentials
