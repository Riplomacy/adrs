# 0009: Cross-Account DNS Delegation

**Status:** Deprecated 2026-10-14 -- the delegated subdomain and its delegation were removed on 2026-08-27, when the shared gateway was retired (ADR-0024)\
**Date:** 2025-12-03\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** INFRA-004

## Decision

The services subdomain is delegated from the account that owns the root domain to the Riplomacy account, which manages its own records. The delegation is automated as infrastructure as code across both accounts.

## Context

Services needed a clean domain name for their endpoints. The root domain already existed in a separate AWS account from the one the services run in. Registering a new domain would cost more and fragment DNS management.

## Options Considered

- **Register a new domain:** extra annual cost and fragmented DNS.
- **Move the whole domain to the services account:** it was already in use in the other account.

## Consequences

- Reuses an existing domain, and the services account fully controls its subdomain.
- The services account controls exactly what the parent account can read.
- A cross-account dependency for setup, with manual deployment ordering, because stack dependencies can't span accounts.
