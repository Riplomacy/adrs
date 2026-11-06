# 0039: API Keys Identify Outside Callers of the Public API

**Status:** Accepted\
**Date:** 2026-11-15\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`

## Decision

We will identify every system outside our AWS account that calls the public API with its own API Gateway API key, attached to a usage plan of its own. Each caller gets narrow, purpose-built operations rather than a pass-through to an internal service, and the handler limits each key to the operations that caller was granted.

## Context

Community-run Discord bots and the SquadJS game server plugins run outside our account and can't sign requests with AWS credentials. Until now the public API's data routes had no authentication at all and passed any action through to the internal services, writes included, so anyone who found the URL could change staff and whitelist data. Those routes have been reduced to the one read SquadJS uses, but the tech team's bots now need their own operations, such as a Steam64 lookup. We need to know which caller is making each request and to be able to cut one off without affecting the others. Proving identity beyond that isn't needed: these callers are a handful of vetted community tools. The monthly AWS bill is small enough that a recurring cost of a few cents matters.

## Options Considered

- **No authentication:** what the old routes did; it can't tell callers apart or revoke one, and it led to open writes.
- **A shared secret checked by the handler:** what the seeders and player-info endpoints do, player-info by signing each request with a timestamp. It works, but every handler reimplements the check, one secret ends up shared by everyone who calls that route, and there's no per-caller throttling.
- **A Lambda authorizer:** real authentication with expiring credentials, at the cost of an extra function to run and pay for, and of credential issuance that our callers would have to implement. More than identification needs.
- **IAM signing:** our callers have no AWS identities and can't sign requests.

## Consequences

- Each caller is identified on every request and can be revoked alone by disabling its key.
- API Gateway rejects a missing or unknown key before any function runs, and usage plans give per-caller throttling and quotas with no code.
- Keys and usage plans have no charge of their own; only the requests are billed, as before.
- An API key is a static shared secret that doesn't expire. A leaked key works until it's disabled, so each key is only as safe as the caller that holds it.
- A key works on every API in its usage plan, so plans can't be shared between callers.
- Usage plan throttling and quotas are best effort, not a cost ceiling. Cost protection needs a separate budget alert.
- Whether API Gateway bills requests it rejects for a bad key isn't documented, so a flood of bad-key traffic may carry a small cost.
- The seeders and player-info routes keep their own secrets for now, so the public API has two ways of identifying callers until they move over.

## Confidence

Medium. Revisit if a caller needs to act for individual members rather than as itself, if a caller handles anything sensitive enough that a leaked key would matter, or if rejected requests turn out to be billed and start costing money.
