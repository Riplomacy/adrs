# 0027: Matched Interaction Defer Budgets

**Status:** Deprecated 2026-10-14 -- a parameter value and bug fix, not architecturally significant (ADR-0034)\
**Date:** 2026-09-14\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** DH-002

## Decision

Command and component interactions defer their Discord response after the same internal budget of 2000 ms.

## Context

Each interaction path defers its response if its handler hasn't answered by an internal deadline, staying under Discord's 3-second limit. An unexplained change had left commands at 2000 ms and components at 2500 ms. Component interactions failed on Lambda cold starts while commands never did. A no-I/O reproduction isolated the budget as the only difference, and matching the two removed the failures.

## Options Considered

- **Eagerly defer specific slow categories:** would have masked the drift and left every other component exposed.
- **Narrow a lock held across an await in component dispatch:** applied as a cleanup, but confirmed not to be the cause.
- **Cold-start networking:** ruled out; the handler isn't in a VPC, and the reproduction did no network I/O.

## Consequences

- Components get the same cold-start margin as commands.
- Handlers that used to answer directly between 2000 and 2500 ms now always defer.
- The underlying cold-start cost is still unexplained.
