# 0024: Internal Services Use Lambda Function URLs

**Status:** Accepted -- fully implemented 2026-08-27, when the shared gateway, its custom domain, certificate, hosted zone and DNS delegation were removed\
**Date:** 2026-08-26\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** INFRA-008\
**Supersedes:** ADR-0006, ADR-0007, ADR-0008, ADR-0019

## Decision

Every internal service gets its own Lambda Function URL with IAM authentication, instead of a route on the shared API Gateway, and the shared gateway is retired. Callers are granted access account-wide, carrying forward ADR-0019's principle of no per-service configuration. This covers internal service-to-service traffic only; the public-facing API gateway is unaffected.

## Context

The shared, path-routed gateway (ADR-0007, ADR-0008) put a layer of indirection in front of Lambdas that were all in the same account and directly reachable anyway. Every new service had to register a route on the shared gateway before it could be called, and every service depended on that gateway's health.

Lambda Function URLs give each Lambda its own HTTPS endpoint with IAM authentication and no gateway in between. The HTTP handling library every service already used accepts the Function URL event shape as-is, so moving a service doesn't change its handler.

## Options Considered

- **Keep the shared gateway and fix its route lifecycle:** keeps the indirection and the per-service registration step.
- **A private API Gateway per service:** the VPC and NAT cost ADR-0006 rejected, multiplied by every service.
- **Direct Lambda invocation:** doesn't fit the Smithy-generated HTTP client and server model.

## Consequences

- One fewer moving part per call, with no shared blast radius.
- A service is callable as soon as its own stack deploys.
- Removes the custom domain and hosted zone cost, and the DNS delegation chain (ADR-0009).
- Each service has a distinct generated endpoint, so callers read it from the callee's stack outputs instead of building a path.
- Those cross-stack references are invisible to automatic dependency ordering, so every one needs an explicit stack dependency, or a first deployment can run out of order and fail.
- The test environment's proxy keeps its path-based routing regardless.
