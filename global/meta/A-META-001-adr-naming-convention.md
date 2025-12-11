# META-001: ADR Naming Convention

**Date:** 2024-12-11\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

ADR filenames currently don't indicate their status (draft, accepted, superseded, rejected), making it difficult to quickly identify which decisions are active versus historical or in-progress when browsing directories.

## Decision

Use status prefixes in ADR filenames with the format: `STATUS-PREFIX-###-descriptive-title.md`

### Status Indicators
- `A-` = Accepted
- `D-` = Draft  
- `S-` = Superseded
- `R-` = Rejected

### Examples
- `A-META-001-adr-naming-convention.md`
- `D-INFRA-002-public-api-gateway-design.md`
- `S-DEV-001-old-rust-standards.md`
- `R-UDS-003-rejected-nosql-approach.md`

## Consequences

### Positive

- Status is immediately visible when browsing directories
- ADRs group by status when sorted alphabetically
- Consistent naming across all categories
- Easy to identify active vs. historical decisions

### Negative

- *None identified*

## Alternatives Considered

- **Status suffix:** Adding status after number (e.g., `INFRA-001D`) was less visible
- **Directory structure:** Organizing by status in subdirectories scattered related ADRs
- **No status for accepted:** Only marking non-accepted ADRs was inconsistent

## Implementation Notes

- Update README.md to reflect new naming convention
- Rename existing ADRs to follow new format
- Use this convention for all future ADRs
