# Event Orchestrator Service — Design Document

**Date:** 2026-04-04\
**Status:** Proposed\
**Author:** Jean-Sébastien Dominique

## Problem Statement

Riplomacy's microservices (supporter data, user data, staff data, etc.) each manage their own domain data, but downstream services like the whitelist service need to react when the *combination* of data across services reaches a meaningful state. Currently there's no mechanism for services to signal state changes. We need a central orchestrator that receives thin signals from producers, enriches them by querying relevant services, and dispatches actionable events to known consumers.

### Motivating Example

A new Patreon member supports the community but hasn't linked Discord inside Patreon — we have Patreon ID and supporter tier, but no Discord ID, so no event is sent. Then they connect their Steam ID to their Discord ID through the bot — there's no link with their Patreon ID yet, so we only have a Discord-Steam association. Then they connect their Patreon account to Discord — we now have a Patreon-Discord link and their supporter tier is linked to their Discord ID, but the whitelist service also needs the Discord-Steam link. No single service has the full picture, and no single event carries all the information needed.

## Requirements

- Producers send thin "something changed" signals to the orchestrator (e.g., "Steam ID linked for Discord ID X")
- The orchestrator knows which services to query for each signal type to build a complete picture
- The orchestrator dispatches enriched events to a hardcoded list of consumer services per event type
- Consumer dispatch uses retry with a small number of attempts before logging failure
- All enriched events are logged as structured JSON for replay capability
- A replay mechanism (API endpoint) can re-dispatch failed events from logs
- No SQS, no SNS fan-out — direct invocation via API Gateway (Smithy clients) for both queries and dispatch

## Background

- Services communicate via Smithy-generated clients over HTTPS through a shared API Gateway (ADR DEV-002)
- Services follow a consistent pattern: Smithy models → generated SDK → Lambda behind API Gateway → DynamoDB
- CDK infrastructure with service-owned routes on shared API Gateway (ADR INFRA-002, INFRA-003)
- The whitelist service already has a 5-minute scheduled refresh that acts as a natural safety net for missed events
- The health-monitoring-service exists for operational monitoring — dispatch failures and enrichment errors should be surfaced there for visibility

## Alternatives Considered

### Dumb Events, Smart Consumers
Events are thin notifications. The consumer receives the event, then queries whatever other services it needs. This creates a web of inter-connected services where every consumer must know about every data source, making dependency management difficult.

### Enriched Events via Chaining
When an event fires, services publish follow-up enriched events after gathering more context. This creates a cascade risk where every service publishes its own information for every event, potentially leading to infinite loops.

### Event Stream (Ideal but Expensive)
A stream of events where all services can respond with the data they have, making all information available by reading the stream following the event. Services like Kinesis or MSK are too expensive for this scale.

### Chosen: Central Orchestrator (Hub-and-Spoke)
A central service receives thin signals, actively queries relevant services to build a complete picture, and dispatches well-defined enriched events to known consumers. The orchestrator's "full view" is bounded — it doesn't own the data, it just knows which services to query for each event type, and the set of event types is finite.

## Proposed Solution

The Event Orchestrator is a new Smithy-modeled service deployed as a Lambda behind the shared API Gateway. It exposes two categories of endpoints:

1. **Signal ingestion** — Producers call these to report state changes (thin payloads)
2. **Replay** — Operators call this to re-dispatch failed events

Internally, the orchestrator:
- Receives a signal
- Looks up the enrichment strategy for that signal type (which services to query, what data to fetch)
- Queries those services via their Smithy clients
- Assembles an enriched event
- Logs the enriched event as structured JSON
- Dispatches to each registered consumer, retrying individual failures (2-3 attempts)
- Logs dispatch results per consumer
- Reports persistent failures to the health monitoring service

### Architecture Diagram

```
                    ┌─────────────────────┐
                    │  Event Orchestrator  │
                    │                     │
 Producers          │  ┌───────────────┐  │          Consumers
 ─────────          │  │Signal Handler │  │          ─────────
                    │  └───────┬───────┘  │
 User Data ────────►│         │           │────────► Whitelist
 Service            │  ┌──────▼────────┐  │          Service
                    │  │  Enrichment   │  │
 Supporter ────────►│  │    Engine     │◄─┼── queries ──► User Data
 Data Service       │  └──────┬────────┘  │              Supporter Data
                    │         │           │              Staff Data
 Discord ──────────►│  ┌──────▼────────┐  │
 Bot                │  │   Structured  │  │────────► Staff Service
                    │  │  Event Log    │  │
 Staff Data ───────►│  └──────┬────────┘  │────────► Discord Role
 Service            │         │           │          Service
                    │  ┌──────▼────────┐  │
                    │  │   Consumer    │  │────────► BattleMetrics
                    │  │  Dispatcher   │  │          Service
                    │  └──────┬────────┘  │
                    │         │           │
                    └─────────┼───────────┘
                              │
                              ▼ (failures)
                    Health Monitoring Service
```

## Signal Types and Enrichment

| Signal | Producer | Enrichment Queries | Enriched Payload | Consumers |
|--------|----------|-------------------|-----------------|-----------|
| `steam_linked` | User Data Service | Supporter (by discord_id), Staff (by discord_id) | discord_id, steam_id, supporter_tier?, staff_rank? | Whitelist, BattleMetrics, Discord Roles |
| `steam_unlinked` | User Data Service | Supporter (by discord_id), Staff (by discord_id) | discord_id, old_steam_id, supporter_tier?, staff_rank? | Whitelist, BattleMetrics |
| `tier_changed` | Supporter Data Service | User (by discord_id for steam_id) | patreon_id, discord_id, steam_id?, old_tier, new_tier | Whitelist, Discord Roles, BattleMetrics |
| `supporter_discord_linked` | Supporter Data Service | User (by discord_id for steam_id) | patreon_id, discord_id, steam_id?, tier | Whitelist, Discord Roles, BattleMetrics |
| `supporter_discord_unlinked` | Supporter Data Service | User (by discord_id for steam_id) | patreon_id, old_discord_id, tier | Whitelist, Discord Roles, BattleMetrics |
| `rank_changed` | Staff Data Service | User (by discord_id for steam_id) | discord_id, steam_id?, old_rank, new_rank | Whitelist, Discord Roles, BattleMetrics |
| `staff_qualification_changed` | Staff Data Service | User (by discord_id for steam_id) | discord_id, steam_id?, qualifications | Whitelist, Discord Roles |
| `friend_updated` | Staff/Supporter Data | User (by friend discord_id for steam_id) | owner_discord_id, friend_discord_id, friend_steam_id?, action (add/remove), source (staff/supporter) | Whitelist |
| `left_server` | Discord Bot | Supporter (by discord_id), Staff (by discord_id) | discord_id, steam_id?, was_supporter?, was_staff? | Whitelist, BattleMetrics, Staff Service |
| `promotion_group_changed` | Whitelist Service | User (by discord_id for steam_id) | group_name, discord_id, steam_id?, action (add/remove) | Discord Roles |

## Task Breakdown

### Task 1: Define the Smithy model

- Create the service repository following the existing pattern (supporter_data_service as template)
- Define signal input structures for each signal type
- Define the enriched event output structures
- Define the replay API operation
- Model the service with all operations behind `/event-orchestrator` path prefix
- **Test:** Smithy model builds and generates valid server/client SDKs
- **Demo:** Generated SDK compiles, signal and enriched event types are available as Rust structs

### Task 2: Implement the enrichment engine

- Create an enrichment module that maps signal types to enrichment strategies
- Each strategy defines which service clients to call and how to assemble the enriched event
- Use the existing Smithy client pattern (supporter-data-service-client-sdk, user-data-service-client-sdk, etc.) to query services
- Handle cases where enrichment queries return no data (e.g., no Steam ID linked yet) — the enriched event still gets dispatched with optional fields as `None`
- **Test:** Unit tests with mocked service clients verifying enrichment logic for each signal type
- **Demo:** Given a `steam_linked` signal with a discord_id, the enrichment engine queries supporter and staff services and returns a complete enriched event struct

### Task 3: Implement structured event logging

- Define a structured JSON log format for enriched events that includes: event_id (UUID), signal_type, timestamp, enriched_payload, and dispatch_targets
- Log every enriched event before dispatch using `tracing::info!` with structured fields
- Ensure the log format supports CloudWatch Logs Insights queries for filtering by signal_type, discord_id, event_id, and time range
- **Test:** Verify log output format matches expected structure
- **Demo:** Processing a signal produces a queryable structured log entry in CloudWatch-compatible JSON format

### Task 4: Implement the consumer dispatcher

- Create a dispatcher that takes an enriched event and a list of consumer targets
- Dispatch to each consumer via their Smithy client endpoint (HTTPS through API Gateway)
- Implement retry logic: 2 retries with brief backoff per consumer
- Log dispatch success/failure per consumer with the event_id for correlation
- On final failure, log a structured error entry with enough info to replay and report the failure to the health monitoring service
- **Test:** Unit tests with mocked consumers verifying retry behavior and per-consumer independence (one consumer failing doesn't block others)
- **Demo:** Dispatching an enriched event to 3 consumers where one fails shows retries in logs and successful delivery to the other two

### Task 5: Wire up signal handlers and deploy the service

- Implement the Lambda handler using the generated server SDK (same pattern as supporter_data_service)
- Each signal endpoint: validate input → enrich → log → dispatch
- Create CDK infrastructure: Lambda function, API Gateway route (`/event-orchestrator/*`), IAM permissions to invoke consumer Lambdas
- Grant the orchestrator's Lambda permission to call all service APIs through API Gateway
- Environment variables for consumer service endpoints
- **Test:** Integration test sending a signal through the API and verifying the enriched event reaches a mock consumer
- **Demo:** Deploy the service, call the `steam_linked` signal endpoint, observe structured logs showing enrichment and dispatch to consumers

### Task 6: Integrate producers

- Add the event-orchestrator-client-sdk as a dependency to producer services
- In User Data Service: after `ConnectSteam` / `DisconnectSteam` succeeds, call the orchestrator's `steam_linked` / `steam_unlinked` signal
- In Supporter Data Service: after tier changes or Discord link changes, call the appropriate signal
- In the Discord Bot: after staff rank/qualification changes, friend updates, and server leave events, call the appropriate signals
- Producer calls should be fire-and-forget (async invocation) — a failure to notify the orchestrator should not fail the primary operation
- **Test:** Verify producer operations still succeed even when orchestrator is unreachable
- **Demo:** Link a Steam ID through the User Data Service, observe the orchestrator receive the signal, enrich it, and dispatch to consumers

### Task 7: Implement the replay endpoint

- Add a `replay` operation to the Smithy model that accepts a list of event_ids or a filter (signal_type + time range)
- The replay handler accepts the original signal and re-runs the full enrich→dispatch pipeline
- **Test:** Replay a previously failed event and verify it reaches consumers
- **Demo:** Invoke the replay endpoint with a signal_type and event payload, observe re-enrichment and dispatch in logs
