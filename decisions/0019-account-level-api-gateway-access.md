# 0019: Account-Level Access to the Internal API Gateway

**Status:** Superseded by ADR-0024 (its account-level principle carries forward there)\
**Date:** 2025-12-12\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** INFRA-007

## Decision

Any principal in the AWS account may call the internal API Gateway, replacing the single shared role from ADR-0006.

## Context

ADR-0006 restricted the internal gateway to one shared IAM role. Smithy-generated clients expect standard AWS credentials, not role assumption, so the shared role would have needed custom credential providers or wrappers around every generated client.

## Options Considered

- **The single shared role from ADR-0006:** role assumption complicates the generated clients.
- **Per-service IAM permissions:** deployment complexity and ongoing maintenance.

## Consequences

- Generated clients work as-is with each Lambda's own credentials, with no per-service permission setup.
- The security boundary stays at the AWS account.
- Less granular control: anything in the account can call any internal endpoint.
