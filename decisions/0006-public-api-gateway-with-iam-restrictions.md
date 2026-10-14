# 0006: Public API Gateway with IAM Restrictions

**Status:** Superseded by ADR-0024 (its access model was first revised by ADR-0019)\
**Date:** 2025-12-03\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** INFRA-001

## Decision

Internal services are exposed through public API Gateway endpoints restricted by IAM, rather than through private APIs. Only a single shared role that all internal services assume may call them.

## Context

Internal services needed API endpoints. Best practice is a private API, but that requires Lambdas in a VPC, VPC endpoints (about $7/month) and a NAT gateway (about $45/month) for internet access: about $52/month in extra cost.

## Options Considered

- **Private API Gateway:** about $52/month of extra infrastructure.
- **Direct Lambda invocation:** deployment complexity with Smithy-generated services.
- **Application Load Balancer:** higher cost and complexity.

## Consequences

- Avoids the VPC endpoint and NAT gateway costs, and keeps the networking simple.
- Security becomes identity-based rather than network-based; endpoints are technically public, though IAM-protected.
- Less defense in depth than network isolation.
- One shared role gives uniform access control across services.
