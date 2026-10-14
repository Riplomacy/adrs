# 0036: Discord as the Identity Provider for Web Sign-In

**Status:** Accepted\
**Date:** 2026-10-14\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`

## Decision

We will sign web users in through Discord, so a web identity is the user's Discord account: the same identity the bot and its records already use. The bot stores no passwords, and keeps no Discord user tokens beyond sign-in.

## Context

The web admin console (ADR-0035) needs sign-in, which the bot has never had. Every person the bot knows about, whether a staff member, a group member or a group manager, is identified by their Discord ID, and permissions derive from the bot's own records about that ID. The community and its staff all live on Discord. Signing in with Discord's OAuth had already been tried in an earlier experiment.

## Options Considered

- **Our own accounts and passwords:** a credential store to secure, and a second identity to link to each Discord ID.
- **A managed identity service federated with Discord:** another service and its cost, for a single identity source.
- **AWS identities for staff:** unworkable for community staff, with no link to their Discord identity.
- **Sign-in links sent by the bot in Discord:** a custom, non-standard flow that reinvents what OAuth already provides.

## Consequences

- No identity mapping: the signed-in identity is the Discord ID that permissions are already based on.
- Staff sign in with an account they already have, and there are no passwords to store or reset.
- Discord becomes a dependency of web sign-in: a Discord outage or an OAuth change blocks it.
- An OAuth application credential to protect.
- Authorization remains ours, decided from the bot's own records, not from Discord.
- After sign-in, session lifetime and revocation are ours to manage.

## Confidence

High. Revisit if web users ever need to exist without a Discord account.
