# DH-003: Staff Bug Reports and Feature Requests Filed Through the Bot

**Date:** 2026-10-07\
**Status:** Proposed\
**Deciders:** Jean-Sébastien Dominique

## Context

Staff are the main source of bug reports and feature requests for the bot. Today those arrive through Discord conversations and threads, and only reach the work tracker (DEV-009) when someone transcribes them, which loses requests and detail along the way. The tracker is private, and staff have no access to it.

## Decision

Staff submit bug reports and feature requests through the bot, which files them as issues in the work tracker via the GitHub integration (INFRA-011).

Submission is one-way: reporters don't get visibility into the tracker.

## Consequences

### Positive

- Requests land in the tracker in the reporter's own words, without a manual relay step
- Staff use the tool they already work in
- The tracker stays private

### Negative

- Reporters can't follow the status of what they submitted
- Lowers the barrier to filing, so low-quality or duplicate submissions become more likely

### Neutral

- Submissions are attributed to the reporting staff member within the issue, since GitHub sees only the bot's identity

## Alternatives Considered

- **Keep relaying requests by hand:** rejected; the transcription step is where requests get lost
- **A dedicated Discord channel or forum for requests:** rejected; still requires transcription into the tracker
- **Give staff access to the tracker:** rejected; the repositories stay private
