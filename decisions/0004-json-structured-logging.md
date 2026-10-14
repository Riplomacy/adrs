# 0004: JSON Structured Logging

**Status:** Accepted\
**Date:** 2025-12-03\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** Workspace\
**Converted from:** DEV-005

## Decision

Logs are structured JSON, carrying the same level of detail the unstructured text logs did.

## Context

Unstructured text logs carried enough detail for tracing events, and that detail had proven valuable for rebuilding consistent data after corruption incidents. But unstructured text can't be parsed reliably by monitoring and observability tooling.

## Options Considered

- **Keep unstructured logs:** limits automated parsing.
- **A custom structured format:** JSON is widely supported.

## Consequences

- Logs can be parsed automatically for monitoring and alerting, and integrate with aggregation tools.
- Event tracing and data-recovery capabilities are kept.
- Existing log statements need converting.
- Log payloads are somewhat larger than plain text.
