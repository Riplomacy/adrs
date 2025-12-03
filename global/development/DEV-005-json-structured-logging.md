# DEV-005: JSON-Based Structured Logging

**Date:** 2025-12-03\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

Log messages originally used unstructured text format with sufficient information for observability and event tracing. This approach proved valuable for recreating consistent data states after corruption incidents. However, the lack of structured format limits automated log parsing capabilities for monitoring and observability tooling.

## Decision

Migrate from unstructured text logging to JSON-based structured logging format. Maintain the same level of information detail while enabling automated log parsing and analysis.

## Consequences

### Positive

- Enables automated log parsing for monitoring and alerting
- Improves observability through structured data analysis
- Maintains existing traceability and event recreation capabilities
- Better integration with log aggregation and analysis tools

### Negative

- Migration effort to convert existing log statements
- Slightly larger log payload sizes compared to plain text

## Alternatives Considered

- **Continue with unstructured logging:** Rejected due to limited automated parsing capabilities
- **Custom structured format:** Rejected in favor of widely-supported JSON standard

## Implementation Notes

- Use JSON format for all new log entries
- Maintain consistent field naming conventions across services
- Include sufficient context for event tracing and data state recreation
- Preserve existing information detail level during migration
