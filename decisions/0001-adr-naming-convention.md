# 0001: Status Prefixes in ADR Filenames

**Status:** Superseded by ADR-0034\
**Date:** 2024-12-11\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** Workspace\
**Converted from:** META-001

## Decision

ADR filenames carry a status prefix (Accepted, Proposed, Superseded, Rejected) ahead of a category prefix and a per-category number.

## Context

ADR filenames didn't indicate their status, which made it hard to tell active decisions from historical or in-progress ones when browsing directories.

## Options Considered

- **Status suffix after the number:** less visible.
- **One directory per status:** scattered related ADRs.
- **Marking only non-accepted ADRs:** inconsistent.

## Consequences

- Status is visible when browsing, and ADRs group by status when sorted.
- Naming is consistent across categories.
- Every status change requires renaming the file.
