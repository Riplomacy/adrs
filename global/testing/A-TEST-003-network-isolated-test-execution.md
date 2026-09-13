# TEST-003: Network-Isolated Test Execution for Concurrent Worktrees

**Date:** 2026-09-13\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

The project is moving to an AI-first workflow where multiple agents work in parallel across git worktrees. The Docker test environment (TEST-001) did not support this: `tests/docker-compose.yml` hardcoded a fixed bridge network name and subnet, every service had a hardcoded `container_name`, and ~24 services published fixed host ports. The Rust integration-test suite (`rust-tests`) ran on the host and reached into the stack via those published `localhost:PORT` addresses. Two worktrees running `./start-services.sh` at the same time collided on all three axes — container names, the network's OS-level bridge device, and host ports.

A lock-based serialization approach (only one Docker stack runs at a time, regardless of worktree) was considered and rejected: agents can shell out to raw `docker`/`docker compose` commands that bypass a script-level lock, and nothing stops one agent rebuilding the environment out from under a test run another agent started.

## Decision

1. Remove every published host port and the fixed bridge name/subnet from the compose stack. Nothing in the stack is reachable from the host any more.
2. Run the integration test suite itself inside the compose network, via a new `test-driver` service — a musl-static build of the `riplomacy-tests` binary (`cargo zigbuild --target {aarch64,x86_64}-unknown-linux-musl`) in a minimal Alpine container. Every URL the suite hits was rewritten from `http://localhost:PORT` to the corresponding Compose service DNS name (e.g. `http://internal-api-proxy`, `http://battlemetrics-mock:80`).
3. Derive `COMPOSE_PROJECT_NAME` from the checkout's toplevel directory name (`tests/scripts/env.sh`, sourced by every wrapper script) so the main checkout and every worktree get their own fully isolated network and container set automatically, with no manual naming scheme.

## Consequences

### Positive

- The main checkout and any number of worktrees can run independent Docker test stacks concurrently with zero shared state.
- No port or container-name collisions, and no naming convention to maintain by hand — Compose auto-namespaces everything from `COMPOSE_PROJECT_NAME`.
- Removing ports as a class of resource eliminates 24 of the ~57 collision points outright, rather than just parameterizing them.

### Negative

- Ad-hoc host-side access (`curl localhost:PORT`, manual `aws dynamodb` calls) no longer works — replaced by `docker compose exec`, `./scripts/logs.sh`, and a `docker compose run --rm aws-cli` one-shot service for the two manual DynamoDB-seeding scripts. Acceptable here because the project's testing is AI-driven, not manual.
- The test suite must be cross-compiled to a musl target and run as a container instead of a plain `cargo test` on the host, adding a build step (`build-all.sh` now also produces `target/test-driver/riplomacy-tests`).
- A compile-time host path (`env!("CARGO_MANIFEST_DIR")`) previously used by `test_patreon_reconcile.rs` to reach a bind-mounted local-S3 fixture broke under this model (the path only existed on the host, not inside the container) and had to be replaced with the same fixed container path (`/tmp/s3`) the `patreon-reconcile` service itself uses.

## Alternatives Considered

- **Lock-based serialization (`flock` around `run-tests.sh`):** Rejected — doesn't stop agents running raw `docker` commands that bypass the lock, and doesn't prevent one agent rebuilding the environment mid-run of another's test pass.
- **Parameterize ports/names per worktree instead of removing them (e.g. a port-offset scheme):** Rejected once it became clear nothing actually needed host access any more — removing the whole class of resource is simpler than an offset scheme and one fewer thing to keep consistent as services are added.
- **Isolate at the Docker-daemon level (a separate Colima VM profile per worktree):** Rejected as heavier than the problem warranted — the collisions were a Compose-naming problem, not something that needed a whole extra VM per worktree to solve.

## Implementation Notes

- `tests/scripts/env.sh` — derives and exports `COMPOSE_PROJECT_NAME`; sourced by `start-services.sh`, `run-tests.sh`, `scripts/clean.sh`, `scripts/logs.sh`.
- `tests/services/test-driver.yml` + `tests/test-driver/{Dockerfile,entrypoint.sh}` — the in-network test runner; entrypoint resets the BattleMetrics mock then execs the compiled test binary.
- `tests/services/dev-tools.yml` — a `profiles: ["tools"]` AWS-CLI one-shot service for the manual seeding scripts (`setup-preset-profiles.sh`, `setup-signal-test-user.sh`), invoked via `docker compose run --rm aws-cli /scripts/...`.
- `tests/scripts/build-all.sh` — extended to cross-compile `riplomacy-tests` to the host-matching musl target and stage it at `target/test-driver/riplomacy-tests`.
