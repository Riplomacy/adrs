# 0033: GitHub as an External Dependency for Issue Filing

**Status:** Accepted 2026-10-14\
**Date:** 2026-10-07\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** INFRA-011

## Decision

The bot integrates with GitHub to file issues in the work tracker. GitHub becomes an external dependency, used for issue filing only.

## Context

Work is tracked in GitHub Issues. Requests and bug reports that start in Discord were relayed and filed by hand. The bot sits between staff and the rest of the system, but had no integration with GitHub.

## Options Considered

- **Keep filing by hand:** keeps the relay-and-transcribe bottleneck.
- **A third-party form or bridge service:** adds another external party, and takes staff outside the bot they already use.

## Consequences

- Requests captured in Discord reach the tracker without manual transcription.
- The integration surface stays narrow: one capability, one repository.
- A new external dependency, whose outages or API changes affect issue filing.
- A new credential to manage.
- Issue filing is low-volume and off every critical path, so a GitHub outage degrades only this capability.
