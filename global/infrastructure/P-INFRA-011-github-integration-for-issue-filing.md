# INFRA-011: GitHub Integration for Issue Filing

**Date:** 2026-10-07\
**Status:** Proposed\
**Deciders:** Jean-Sébastien Dominique

## Context

With GitHub Issues as the work tracker (DEV-009), requests and bug reports originating in Discord still have to be relayed and filed by hand. The bot already sits between staff and the rest of the system, but has no integration with GitHub.

## Decision

The bot integrates with GitHub to file issues in the work tracker. GitHub becomes an external dependency of the bot, used for issue filing only.

## Consequences

### Positive

- Requests captured in Discord reach the tracker without manual transcription
- The integration surface stays narrow: one capability, one repository

### Negative

- A new external dependency whose outages or API changes affect issue filing
- A new credential to manage (see INFRA-012)

### Neutral

- Issue filing is low-volume and not on any critical path; a GitHub outage degrades only this capability

## Alternatives Considered

- **No integration, manual filing:** rejected; keeps the relay-and-transcribe bottleneck
- **A third-party form or bridge service:** rejected; adds another external party and leaves staff outside the bot they already use
