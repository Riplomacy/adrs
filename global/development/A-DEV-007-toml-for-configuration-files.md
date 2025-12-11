# DEV-007: TOML for Configuration Files

**Date:** 2025-12-11\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

Configuration files need to be human-readable and maintainable. Various formats exist including JSON, YAML, INI, and TOML, each with different characteristics and design philosophies.

## Decision

Use TOML for all configuration files across Riplomacy projects. This includes service mappings, application settings, and any other configuration needs.

## Consequences

### Positive

- Human-friendly syntax designed specifically for configuration
- Modern format with clear semantics
- Strong typing and explicit structure
- Comments support for documentation
- Consistent configuration format across all projects

### Negative

- *None identified*

## Alternatives Considered

- **JSON:** Rejected as it feels more machine-oriented than human-friendly
- **YAML:** Rejected in favor of TOML's more explicit syntax
- **INI:** Rejected as less modern and feature-complete

## Implementation Notes

- Use TOML for configuration files only, not for data serialization
- Apply consistently across all services and tools
