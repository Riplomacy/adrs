# 0017: Axum for the Test Proxy

**Status:** Accepted\
**Date:** 2025-12-11\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `http-logger`\
**Converted from:** HTTP-003

## Decision

The test proxy is built on Axum.

## Context

A reverse proxy has to handle HTTP requests and responses. Hyper provides the low-level primitives, but needs a lot of boilerplate for common server patterns.

## Options Considered

- **Hyper directly:** too low-level for this use.
- **Other frameworks such as Actix or Rocket:** not considered, given existing familiarity with Axum.

## Consequences

- Less boilerplate than raw Hyper, on the proven Hyper and Tokio foundation.
- A familiar framework with clear patterns.
