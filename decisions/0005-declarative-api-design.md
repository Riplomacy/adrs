# 0005: Declarative API Design

**Status:** Accepted\
**Date:** 2025-12-03\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** Workspace\
**Converted from:** DEV-006

## Decision

APIs are declarative: an operation expresses the intended end state and succeeds once that state holds, whether or not anything had to change. This applies wherever uniqueness and data consistency make the intent unambiguous.

## Context

Imperative APIs make clients check current state first and handle conditions like "already exists" or "already disconnected". That pushes implementation details into every client instead of letting it state what it wants.

## Options Considered

- **Imperative APIs:** complex client-side state checks.
- **A mix without clear guidelines:** inconsistent for developers.

## Consequences

- Clients no longer need conditional error handling for states that already match.
- Operations are idempotent and safe to retry.
- Coupling between client logic and current system state is reduced.
- Doesn't apply to every operation; conflicting or ambiguous requests still fail normally.
