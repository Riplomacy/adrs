# TEST-004: `internal: true` for Docker Test-Network Egress Isolation

**Date:** 2026-09-14\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

`TESTING_ARCHITECTURE.md` §7.5 claimed the Docker test network blocked all outbound internet
access, verified empirically on 2026-07-10 via `com.docker.network.bridge.enable_ip_masquerade:
"false"` plus a fixed subnet on the `riplomacy-test` bridge network.

TEST-003 (2026-09-13, "Network-Isolated Test Execution for Concurrent Worktrees") removed the
fixed bridge name and subnet from that same network block so concurrent worktrees don't collide.
`enable_ip_masquerade: "false"` was deleted in the same edit, since it lived in the same YAML
block — but TEST-003's Context/Decision/Consequences never discuss egress at all; its entire
rationale is about container-name/port/subnet collisions between concurrent stacks. This reads as
an unnoticed side effect, not a considered tradeoff.

Discovered 2026-09-14 when a `ticket_service` integration test accidentally omitted
`BATTLEMETRICS_ENDPOINT`, and the resulting call reached the *real* `api.battlemetrics.com`
instead of failing to connect. Verified directly: `docker network inspect` on the running test
network showed `"Options": {}` (the masquerade driver-opt was never applied in the first place —
it appears Colima's bridge driver silently ignores it, which may also explain why the original
2026-07-10 verification didn't catch a later regression: it may never have generalized across
backends) and `curl https://example.com` from inside a stack container returned a real HTTP 200.

## Decision

Set `internal: true` on the `riplomacy-test` network (still `driver: bridge`, still no fixed
name/subnet, so TEST-003's concurrent-worktree isolation is unaffected). `internal` is enforced
by the Docker daemon/libnetwork itself (no default route is ever installed for the network) rather
than delegated to an iptables driver-opt whose effect depends on the backend's bridge
implementation — verified empirically to actually block egress (DNS resolution itself fails) under
Colima, where the driver-opt approach did not.

This costs nothing: per this repo's existing testing philosophy (`TESTING_ARCHITECTURE.md` §4.4,
"Fake external APIs, don't call them"), nothing in the stack should ever need to reach a real
external host at runtime — Discord, Patreon, BattleMetrics, and PSG are all in-network fakes.
Image pulls/builds are unaffected (`internal` only governs the runtime container network, not the
build-time network Docker uses).

## Consequences

### Positive

- Egress is blocked by a mechanism enforced by Docker itself, not a driver-opt that at least one
  backend (Colima) silently no-ops.
- A misconfigured service that starts making a real external call now fails loudly (connection/DNS
  error) instead of silently succeeding against production infrastructure with real consequences.
- Verified compatible with TEST-003: no fixed subnet/name reintroduced, full suite (94 tests)
  still passes unchanged.

### Negative

- *None identified* — nothing in the stack has a legitimate reason to reach the real internet.

### Neutral

- Doesn't fix or explain why the masquerade driver-opt was silently ignored under Colima; that's a
  pre-existing Docker/Colima behavior, not something this ADR changes or relies on further.

## Alternatives Considered

- **Restore `enable_ip_masquerade: "false"` as TEST-003 had it, without the fixed subnet:**
  Rejected — this is exactly what was tested first and confirmed to still not take effect under
  Colima (`docker network inspect` showed the driver-opt wasn't applied even after a clean
  recreate). The mechanism itself doesn't generalize across backends.
- **A different network driver (e.g. `macvlan`/`ipvlan`):** Rejected — those put containers
  directly on the physical/virtual LAN with their own addresses, which typically increases
  external reachability rather than reducing it, and doesn't fit the "containers reach each other
  by Compose DNS name" model the rest of the stack depends on.

## Implementation Notes

- `tests/docker-compose.yml` — `networks.riplomacy-test.internal: true`.
- No other file changes; TEST-003's `COMPOSE_PROJECT_NAME`-derived per-worktree isolation is
  untouched.
