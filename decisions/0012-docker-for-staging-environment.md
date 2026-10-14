# 0012: Docker for the Staging Environment

**Status:** Accepted\
**Date:** 2025-12-03\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** TEST-001

## Decision

Staging is a local Docker environment running the Lambdas in containers, with local DynamoDB and fakes for external services, instead of a separate AWS account.

## Context

A separate staging account inside the AWS organization was tried first. The organization shares one free tier across all its accounts, so staging and production competed for it, and DynamoDB usage across both caused unexpected costs.

## Options Considered

- **A staging account inside the organization:** shared free tier, unexpected costs.
- **A staging account outside the organization:** could double costs.
- **The production account with resource prefixes:** risks contaminating production data.

## Consequences

- No more unexpected costs from a shared free tier.
- A faster development loop with full control over test data, and automated integration testing locally.
- Significant up-front work to containerize the Lambdas and build fakes, and ongoing upkeep for both.
- The local environment may not match AWS exactly, and not every AWS service can run locally.
