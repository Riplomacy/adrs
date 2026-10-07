# INFRA-012: Personal Access Token for GitHub Authentication

**Date:** 2026-10-07\
**Status:** Proposed\
**Deciders:** Jean-Sébastien Dominique

## Context

The GitHub integration (INFRA-011) needs an identity to act as. GitHub offers two options for automation: a personal access token issued by a user account, or a GitHub App with its own identity that mints short-lived installation tokens.

## Decision

The bot authenticates to GitHub with a fine-grained personal access token rather than a GitHub App.

## Consequences

### Positive

- Minimal setup: no App registration, private key handling, or token-minting flow
- The token's permissions can be restricted to exactly what issue filing needs

### Negative

- Actions on GitHub are attributed to a personal account rather than a distinct bot identity
- The credential is long-lived compared to an App's short-lived installation tokens

### Neutral

- Moving to a GitHub App later stays possible without changing what the integration does

## Alternatives Considered

- **GitHub App:** deferred; the right long-term identity, but its setup cost isn't justified for a single low-volume capability
- **Classic personal access token:** rejected; its permissions can't be restricted to a single repository

## Implementation Notes

- Fine-grained tokens expire. An expired token breaks issue filing silently until it is rotated.
- The token is tied to a personal account. If that account loses access to the organization, issue filing breaks with it.
