# 0013: TOML for Configuration Files

**Status:** Accepted\
**Date:** 2025-12-11\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** Workspace\
**Converted from:** DEV-007

## Decision

All configuration files across Riplomacy projects use TOML. TOML is for configuration only, not for data serialization.

## Context

Configuration files need to be human-readable and maintainable. JSON, YAML, INI and TOML each take a different approach.

## Options Considered

- **JSON:** more machine-oriented than human-friendly.
- **YAML:** TOML's syntax is more explicit.
- **INI:** less modern and less complete.

## Consequences

- A syntax designed for configuration, with explicit structure and comments.
- One configuration format across every project.
