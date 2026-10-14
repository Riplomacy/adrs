# 0035: Web Admin Console as a Second Entry Point

**Status:** Accepted\
**Date:** 2026-10-14\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`

## Decision

We will add a web admin console for managing the bot alongside Discord. It is a static browser application with its own backend, which is the only thing the browser talks to and which calls internal services like any other caller. The console lives in its own stack, owning everything it needs, separate from the shared web bucket for published content (ADR-0032).

## Context

Staff manage the bot through Discord commands, one change at a time. Some data has outgrown that: promotion groups hold thousands of memberships, and Discord's interaction model (short messages, small select menus, single-form modals) makes browsing, searching and reviewing them impractical.

Internal services are only reachable with IAM credentials (ADR-0024), so a browser can't call them. The shared web bucket (ADR-0032) serves content to anyone, without sign-in.

The console has a handful of users and low volume, so cost and load should stay near zero.

## Options Considered

- **More Discord commands and modals:** Discord's interface limits make large lists impractical, however the commands are shaped.
- **The browser calls internal services directly:** would make internal services public and push sign-in and authorization into every service.
- **Host the console in the shared web bucket and distribution:** mixes a signed-in application with public content, and ties the console to another stack.
- **An always-on server:** ongoing cost and upkeep for a few users.

## Consequences

- Staff get an interface suited to large lists, while Discord stays the place for quick actions.
- A new public entry point and attack surface. Every request goes through the console's backend, which enforces sign-in and authorization.
- Internal services stay private; the console's backend is just another caller.
- The console's stack shares nothing with other stacks.
- Two interfaces offer some of the same operations. Business rules stay in the services, so both behave the same.
- Serverless and near zero cost at this usage.

## Confidence

High. Revisit if the console needs real-time updates, grows well beyond a handful of staff, or ever needs to serve non-staff users.
