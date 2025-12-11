# INFRA-004: Cross-Account DNS Delegation

**Date:** 2025-12-03\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

Services need a clean domain name for API endpoints. The root domain `amateurradio.engineer` already exists in the AmateurRadio AWS account, but the Riplomacy services are deployed in a separate Riplomacy AWS account. Creating a new domain would cost additional money and complicate DNS management.

## Decision

Use cross-account DNS delegation to create `services.amateurradio.engineer` subdomain in the Riplomacy account while keeping the root domain in the AmateurRadio account. Automate the delegation process through CDK deployments with cross-account role assumptions.

## Consequences

### Positive

- Reuses existing domain without additional registration costs
- Clean subdomain structure for services
- Automated delegation setup through infrastructure as code
- Service account maintains full control over subdomain DNS
- Service account controls exactly what parent account can access

### Negative

- Cross-account dependency for initial setup
- Manual deployment ordering required (CDK doesn't support cross-account stack dependencies)

### Neutral

- Parent account deployment fails if service account stack not deployed first

## Alternatives Considered

- **Register new domain:** Rejected due to additional annual costs and DNS fragmentation
- **Move entire domain to service account:** Rejected due to existing usage in parent account

## Implementation Notes

- Service account creates `services.amateurradio.engineer` hosted zone and outputs NS records
- Service account creates role allowing parent account to read specific stack outputs via DescribeStack
- Parent account assumes role to read NS records and create delegation
- Deploy service account stack first, then parent account stack
- Service account can manage all DNS records under subdomain independently
