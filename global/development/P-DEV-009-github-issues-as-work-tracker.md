# DEV-009: GitHub Issues as the Central Work Tracker

**Date:** 2026-10-07\
**Status:** Proposed\
**Deciders:** Jean-Sébastien Dominique

## Context

Open work for the Riplomacy workspace (bugs, feature requests, refactors, documentation) has been tracked in an in-repo Markdown file. That file grew past a hundred items across many sections, mixing open and completed work. It has no way to label, filter, link, or close items individually, and nobody outside the repository can submit to it. Staff requests reach it only after being relayed and transcribed by hand.

The workspace spans several repositories, but its work is owned and triaged by a single maintainer.

## Decision

GitHub Issues is the work tracker for the Riplomacy workspace. All issues are tracked centrally in the `riplomacy_bot` repository, regardless of which component they concern, and are moved to the owning repository only when that becomes useful.

In-repo text files no longer track open work.

## Consequences

### Positive

- Items can be labeled, filtered, cross-referenced with commits, and closed individually
- One place to look for all open work across the workspace
- Opens the door to accepting submissions from outside the repository

### Negative

- Open work no longer lives alongside the code, so it isn't available offline or in a plain checkout
- Ties work tracking to GitHub as a platform

### Neutral

- Issues concerning other repositories sit in `riplomacy_bot` until moved

## Alternatives Considered

- **Keep the Markdown file:** rejected; it doesn't scale and can't accept outside submissions
- **A dedicated tracker repository:** rejected; adds a repository without adding capability over centralizing in `riplomacy_bot`
- **Per-repository issues from the start:** rejected; most work touches `riplomacy_bot`, and splitting up front spreads triage across many places
