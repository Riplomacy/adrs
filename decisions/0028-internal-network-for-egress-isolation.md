# 0028: Docker-Enforced Egress Isolation for the Test Network

**Status:** Accepted\
**Date:** 2026-09-14\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot` test environment\
**Converted from:** TEST-004

## Decision

The test network blocks all outbound internet access through Docker's own internal-network mode, which the daemon enforces itself, rather than through a bridge driver option.

## Context

The test network was meant to block outbound access through a bridge driver option. ADR-0025 removed it as a side effect of removing the fixed network settings, without ever discussing egress. A test that accidentally called the real BattleMetrics API instead of the fake exposed it, and investigation showed the driver option had never taken effect on the Docker backend in use anyway.

Nothing in the stack should ever reach a real external host: every external API has an in-network fake.

## Options Considered

- **Restore the bridge driver option:** tested, and confirmed not to take effect on this backend.
- **A different network driver:** puts containers on the LAN, which increases reachability rather than reducing it, and breaks name-based service discovery.

## Consequences

- Egress is blocked by Docker itself, independent of the backend.
- A misconfigured service now fails loudly instead of silently reaching production systems.
- Concurrent worktree isolation (ADR-0025) is unaffected, and image builds still have network access.
- Why the driver option was ignored remains unexplained.
