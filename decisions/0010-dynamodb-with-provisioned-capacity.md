# 0010: DynamoDB with Provisioned Capacity

**Status:** Accepted\
**Date:** 2025-12-03\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** Workspace\
**Converted from:** INFRA-005

## Decision

Data is stored in DynamoDB using provisioned capacity, with all tables and indexes sharing the free tier's read and write capacity so that storage costs nothing.

## Context

Choosing a database meant balancing cost, performance and operational complexity. The workload doesn't need relational queries.

## Options Considered

- **RDS:** higher cost, and relational complexity the workload doesn't need.
- **DynamoDB on-demand:** costs more than provisioned capacity within the free tier.

## Consequences

- Zero database cost, with no servers to manage, on familiar technology.
- Capacity has to be planned across all tables to stay within the free tier.
- A workload that outgrows its provisioned capacity gets throttled.
- Less flexible than on-demand for spiky workloads.
