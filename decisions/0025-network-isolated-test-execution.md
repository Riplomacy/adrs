# 0025: Network-Isolated Test Execution for Concurrent Worktrees

**Status:** Accepted\
**Date:** 2026-09-13\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot` test environment\
**Converted from:** TEST-003

## Decision

The Docker test stack publishes nothing to the host, and the integration test suite runs inside the stack's own network as a container. Each checkout gets its own fully isolated stack, named after the checkout, with no manual naming scheme.

## Context

Work moved to multiple AI agents running in parallel across git worktrees. The test stack used a fixed network name and subnet, fixed container names, and about two dozen fixed host ports, and the test suite ran on the host and reached services through those ports. Two worktrees starting the stack at once collided on all of these.

## Options Considered

- **Serialize test runs with a lock:** agents can bypass a script-level lock with raw Docker commands, and nothing stops one agent rebuilding the environment under another's run.
- **Parameterize ports and names per worktree:** nothing needed host access any more, so removing that whole class of resource was simpler than an offset scheme.
- **A separate Docker VM per worktree:** heavier than a naming problem warrants.

## Consequences

- Any number of checkouts can run independent test stacks concurrently, with no shared state and no naming convention to maintain.
- Ad-hoc access from the host no longer works; manual inspection goes through the container tooling instead. Acceptable because testing is AI-driven.
- The test suite must be cross-compiled and run as a container, which adds a build step.
- Anything in the suite that depended on host paths had to move to container paths.
