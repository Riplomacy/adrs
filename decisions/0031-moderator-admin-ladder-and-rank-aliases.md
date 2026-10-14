# 0031: Moderator/Admin Ladder and Rank Aliases

**Status:** Deprecated 2026-10-14 -- rank naming and data migration, not architecturally significant (ADR-0034)\
**Date:** 2026-09-25\
**Deciders:** Jean-Sébastien Dominique, Praetorian\
**Scope:** `riplomacy_bot`\
**Amends:** ADR-0030 (rank ladder only)\
**Converted from:** STAFF-003

## Decision

The active ladder is Moderator, Admin, Tech Team, Owner, where Moderator is the untrusted-yet tier ADR-0030 called Trial. Awaiting Training and Idle are inactive states, not steps on the ladder. Old rank names are kept as aliases only while current staff records still store them.

## Context

Before deployment, current staff records showed that ADR-0030's ladder didn't match real usage. Folding Moderator into Admin would have given admin cam to members who were trials in practice. Several aliases existed only for names that appeared in promotion history, which is displayed as recorded and needs no alias.

## Options Considered

- **Keep ADR-0030's ladder:** grants admin cam to de facto trials.
- **Rewrite stored rank names:** a data migration that aliases make unnecessary, and stored ranks would disagree with the history that set them.

## Consequences

- Nobody gains or loses admin cam as a side effect of the migration, and no records are rewritten.
- Fewer aliases, each with a reason to exist.
- Removing an alias later requires checking live data, not just configuration.
