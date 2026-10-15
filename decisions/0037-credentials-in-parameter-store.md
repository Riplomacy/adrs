# 0037: Service Credentials in SSM Parameter Store

**Status:** Accepted\
**Date:** 2026-10-14\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`

## Decision

We will keep the bot's third-party credentials in SSM Parameter Store as plain String parameters, one per external service or group of services, each holding that group's TOML. Each function reads only the parameters it needs, and its IAM policy allows only those. Credentials leave git and the configuration layer.

## Context

All credentials (the Discord bot token, game server RCON passwords, BattleMetrics, Patreon and hosting provider keys) live in one file in a private git repository. That file ships in the configuration layer that nearly every function receives, so every function holds every secret. Development is now mostly done by AI agents working in the checkout, so credentials on disk get read by accident. Rotating one credential means rebuilding and republishing the whole layer. The file has already been split into one file per service. The monthly AWS bill is small enough that a recurring cost of a few cents matters.

## Options Considered

- **Keep them in git and the configuration layer:** every function keeps every secret, and secrets stay on disk next to the code.
- **SecureString parameters:** encrypted with a KMS key, but each read is a KMS request. Past the free tier that adds a recurring cost for a threat we accept.
- **Secrets Manager:** a monthly fee per secret, and its rotation features would go unused.
- **One parameter per value:** finer-grained access, but many more parameters to manage, and related values such as a username and password would no longer be edited together.
- **Function environment variables:** values would pass through deployment templates and show in the console, and changing one means redeploying.

## Consequences

- Least privilege that IAM enforces: a function can read only the services it calls.
- No credentials in git or in local checkouts. Copies in the old repository's history remain until each credential is rotated.
- Rotation is independent of code and configuration deploys. Running containers keep the old value until they restart, which a configuration update can force.
- The configuration layer carries only non-secret configuration.
- String parameters aren't encrypted with a key we control: any principal in the account allowed to read them sees the value. We accept this to keep the cost at zero.
- Every function depends on Parameter Store when it starts.
- Editing a credential needs a tool that checks the TOML before writing it, since there's no longer a file to edit.
- Local tools and the test environment need Parameter Store access, or a fake of it.

## Confidence

High. Revisit if the AWS bill grows enough that encryption costs become negligible, or if the account gains principals that shouldn't read credentials.
