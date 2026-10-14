# 0020: Self-Service Staff Onboarding to BattleMetrics

**Status:** Deprecated 2026-10-14 -- feature behavior, not architecturally significant (ADR-0034)\
**Date:** 2025-12-14\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** STAFF-001

## Decision

New staff register their own email with the bot. The bot uses it to invite them to BattleMetrics, and confirms their identity automatically when exactly one invitation is pending. Trainers resolve the ambiguous cases.

## Context

Onboarding to BattleMetrics relied on manual coordination. New staff sent their email to a trainer by DM, the trainer invited them, and the trainer then had to confirm their identity when they joined, a step often forgotten. Emails were shared in DMs, and trainers had to be available at every step.

## Options Considered

- **Keep the manual process:** still depends on trainer availability and coordination.

## Consequences

- Emails are no longer shared in DMs, and forgotten confirmations no longer leave staff without permissions.
- Less trainer work for unambiguous cases.
- Emails are now stored in the database, if only until the invitation completes or expires.
- Concurrent invitations still need a trainer to confirm.
