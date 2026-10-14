# 0023: Staff Escalation Pings

**Status:** Deprecated 2026-10-14 -- a feature, not architecturally significant (ADR-0034)\
**Date:** 2025-12-16\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** DH-001

## Decision

Authorized staff can ping a staff group, chosen by rank, through a bot command. The command carries no free text, which avoids content moderation concerns.

## Context

Staff needed a way to reach head admins and Tech Team for help. Role pings were disabled to stop abuse by non-staff, so staff pinged individuals repeatedly, relayed through others, or waited.

## Options Considered

- **Manual role pings:** disabled to prevent abuse.
- **Dedicated support channels per staff level:** fragments communication.

## Consequences

- Staff can escalate to the right people, with less spam from repeated individual pings.
- More notifications for the targeted groups.
- Pings carry no context.
