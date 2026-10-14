# 0016: Optional Prefix Stripping in the Test Proxy

**Status:** Superseded by ADR-0014\
**Date:** 2025-12-11\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `http-logger`\
**Converted from:** HTTP-002

## Decision

Each route in the test proxy can optionally strip its matched prefix before forwarding the request to the backend service.

## Context

In Docker, clients call services through a prefixed path, but the services themselves expected unprefixed routes, because the prefix was configured at the infrastructure level rather than in service code.

## Options Considered

- **Always strip:** some services might need the full path.
- **Never strip:** didn't match what services expected.
- **Make services handle both:** duplicates routing logic in every service.

## Consequences

- Backends receive the paths they expect, while clients keep a consistent endpoint format.
- May need adjusting depending on how the production gateway passes paths.
