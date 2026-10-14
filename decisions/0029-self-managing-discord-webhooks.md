# 0029: Self-Managing Discord Webhooks

**Status:** Accepted 2026-10-14\
**Date:** 2026-09-16\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** INFRA-009

## Decision

The bot manages its own Discord webhooks instead of depending on ones created by hand. Each webhook's target channel is configured once. The bot resolves the webhook's current URL at runtime, and when a send fails because the webhook is missing or gone, it creates a new one, stores its URL, and retries once.

Each webhook's state is stored separately, so a race between two simultaneous repairs can only affect that one webhook.

## Context

The bot posts through six webhooks that were created by hand in Discord, with their URLs fixed at deploy time. If one is deleted or regenerated on Discord's side, everything posting through it breaks silently until someone recreates it and redeploys. The bot already has the permission to create webhooks wherever it posts.

## Options Considered

- **One shared store entry for all webhooks:** the store has no conditional write, so a race on one webhook could revert a concurrent change to another.
- **A store with conditional writes:** over-engineering; the worst case of a race is one harmless orphaned webhook.
- **Keep deploy-time URLs, consult the store only on failure:** a repaired URL wouldn't reach other running instances until the next deploy.

## Consequences

- A deleted webhook repairs itself on the next send, with no manual step or redeploy.
- Free at current volume.
- Every send path gains a runtime lookup.
- The error-reporting library, used almost everywhere as a minimal utility, takes on new dependencies.
- Simultaneous repairs aren't fully race-proof; revisit if failures or volume grow.
- Webhook display names become a convention the bot enforces.
