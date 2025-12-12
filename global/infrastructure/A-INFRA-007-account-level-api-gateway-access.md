# INFRA-007: Use Account-Level API Gateway Access Control

**Date:** 2025-12-12\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

INFRA-001 established using Public API Gateway with IAM restrictions and proposed a single shared IAM role for internal service communication. However, this shared role approach creates complexity with Smithy-generated clients, which expect standard AWS credentials rather than role assumption logic. The shared role would require custom credential providers or wrapper code around the generated clients.

## Decision

Update the IAM access control implementation from INFRA-001 to use account-level permissions instead of a single shared role. Configure API Gateway resource policy to allow any principal within the AWS account to invoke the API.

## Consequences

### Positive

- Smithy-generated clients work out of the box with Lambda execution role credentials
- No custom credential providers or role assumption logic needed
- Simplified service deployment - no per-service permission configuration required
- Maintains security boundary at the AWS account level established in INFRA-001
- Zero additional infrastructure or maintenance overhead

### Negative

- Less granular access control compared to service-specific roles
- Any resource in the account can call any API endpoint

### Neutral

- Security model remains identity-based rather than network-based
- Account-level isolation still prevents external access

## Alternatives Considered

- **Single shared IAM role (INFRA-001 original):** Rejected due to complexity with Smithy-generated clients requiring role assumption
- **Per-service IAM permissions:** Rejected due to deployment complexity and maintenance overhead

## Implementation Notes

- Configure API Gateway resource policy with `AccountPrincipal` condition
- Lambda functions use their natural execution role credentials
- No additional IAM configuration needed per service
- Smithy clients configured with standard AWS SDK credential chain
