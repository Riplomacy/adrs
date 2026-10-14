# 0002: Smithy for Service Modeling

**Status:** Accepted\
**Date:** 2025-12-03\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** Workspace\
**Converted from:** DEV-002

## Decision

Service APIs are modeled in Smithy, and their clients are generated from those models. Services talk to each other synchronously over HTTPS.

## Context

Service-to-service communication went through several approaches. SNS-SQS-Lambda was too expensive because of SQS costs. SNS-Lambda was cheaper, but added latency and only supported asynchronous calls. Direct Lambda-to-Lambda invocation was fast, cheap and synchronous, but every service API needed a hand-built client that had to be maintained.

## Options Considered

- **SNS-SQS-Lambda:** SQS costs too high.
- **SNS-Lambda:** latency, and no synchronous responses.
- **Hand-written Lambda-to-Lambda clients:** maintenance overhead.

## Consequences

- Clients are generated, so there's no hand-written client code to maintain.
- Type-safe contracts reduce integration errors, and API patterns stay consistent across services.
- Clients for other languages can be generated if ever needed.
- Smithy's IDL and tooling have a learning curve, and client generation adds a build step.
