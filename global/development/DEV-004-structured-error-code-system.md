# DEV-004: Structured Error Code System

**Date:** 2025-12-03\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

Discord bot error messages were originally human-centric without structured formatting. With the introduction of automated integration tests, reliable error detection became critical for distinguishing expected errors from unexpected system failures. Additionally, users needed a concise way to report and discuss specific errors without copying entire error messages. Discord's API requires returning success responses even for application errors so error messages can be displayed to users.

## Decision

Implement structured error codes with prefixes for Discord integration responses. Display error codes inconspicuously alongside human-readable messages, prioritizing user experience while enabling automated testing and user support.

## Consequences

### Positive

- Automated tests can reliably detect and validate specific error conditions
- Users can report issues using concise error codes
- Clear distinction between expected command errors and system failures
- Maintains focus on human-readable error messages

### Negative

- Additional overhead to assign and maintain error codes for Discord responses

## Alternatives Considered

- **Unstructured error messages:** Rejected due to unreliable automated testing
- **Error codes only:** Rejected due to poor user experience

## Implementation Notes

- Use `CMD_` prefix for expected command errors
- Use `APP_` prefix for unexpected application/system errors
- Display format: "Human message\n-# Error: ERROR_CODE"
- Apply to Discord integration where errors must be embedded in success responses
- Maintain inconspicuous visual presentation to prioritize readability
