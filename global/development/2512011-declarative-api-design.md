# ADR-2512011: Declarative API Design

**Date:** 2025-12-03\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

Traditional imperative API design requires clients to check current state before performing operations, leading to complex error handling for conditions like "file already exists" or "user already disconnected." This approach forces clients to handle implementation details rather than expressing their desired end state.

## Decision

Design APIs declaratively where operations express intended end state rather than specific actions. Operations succeed when the desired state is achieved, regardless of whether changes were actually made. Apply this principle wherever uniqueness constraints and data consistency allow.

## Consequences

### Positive

- Eliminates complex conditional error handling in client code
- Operations are idempotent and safe to retry
- Clients express intentions rather than implementation steps
- Reduces coupling between client logic and current system state

### Negative

- Not applicable to all operations (e.g., user creation with conflicting data)

## Alternatives Considered

- **Imperative API design:** Rejected due to complex client-side state checking requirements
- **Mixed approach without clear guidelines:** Rejected due to inconsistent developer experience

## Implementation Notes

- Operations succeed when desired end state is achieved
- Examples: disconnect user (succeeds if already disconnected), delete file (succeeds if already deleted)
- Apply where uniqueness allows unambiguous intent determination
- Use traditional error handling for operations with conflicting data or ambiguous intent
- Document declarative behavior clearly in API specifications
- Logs provide visibility into actual operations performed
