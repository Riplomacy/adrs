# 0014: Service Prefix in Smithy Routes

**Status:** Deprecated 2026-10-14 -- it was shaped around the shared gateway's path routing, which was retired (ADR-0024)\
**Date:** 2025-12-11\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** DEV-008\
**Supersedes:** ADR-0016

## Decision

Smithy route definitions include the service's path prefix, so services handle the full paths the gateway passes them.

## Context

The shared API Gateway (ADR-0007) passes the full request path, prefix included, to the service. Stripping it at the gateway would need custom request mapping templates.

## Options Considered

- **Strip the prefix at the gateway:** needs custom mapping templates.
- **Strip the prefix in service code:** duplicated logic that wouldn't match the models.
- **Handle paths both with and without the prefix:** needlessly complex.

## Consequences

- Models match what the gateway actually sends, with no custom gateway configuration.
- The prefix appears in every route definition.
