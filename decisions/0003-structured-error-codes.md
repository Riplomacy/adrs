# 0003: Structured Error Codes in Discord Responses

**Status:** Accepted\
**Date:** 2025-12-03\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** DEV-004

## Decision

Errors shown in Discord carry a structured error code alongside the human-readable message. Codes distinguish expected command errors from unexpected system failures, and are displayed inconspicuously.

## Context

Error messages were written for humans only, with no structure. Automated integration tests needed a reliable way to tell expected errors from unexpected failures. Users needed a short way to report a specific error without copying a whole message. Discord requires a success response even for application errors, so the error has to be carried inside the message itself.

## Options Considered

- **Unstructured messages:** unreliable for automated testing.
- **Error codes only:** poor user experience.

## Consequences

- Tests can detect and assert specific error conditions reliably.
- Users can report issues by code.
- Expected errors and system failures are clearly separated.
- Every Discord-facing error needs a code assigned and maintained.
