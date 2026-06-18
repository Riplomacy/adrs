# Whitelist Service — Design Document

**Date:** 2026-04-04\
**Status:** Proposed\
**Author:** Jean-Sébastien Dominique

## Problem Statement

The whitelist system is fragmented across 4+ components:

- `whitelist_data_service/` — A service with staff/supporter/voucher CRUD and Steam ID listing, but doesn't generate configs or push to servers
- `whitelist_api/` (lib) — Assembles whitelist config files from raw data (has both v1 and v2 `assemble` paths)
- `whitelist_update_handler/` (lambda) — Triggered by SNS, fetches data directly from DDB tables owned by other services, assembles via `whitelist_api`, diffs against S3, pushes to servers via AWN/PSG. The v2 path that would use the data service is commented out
- `api_seeder_whitelist_handler/` — Manages seeder group membership directly in DDB with timed expiry (per ADR WL-001/WL-002)

Additionally, multiple components write directly to the `GroupWhitelist` DDB table: the seeder API, Discord promotion commands, and the (now-retired) `update_active` lambda.

The `whitelist_update_handler` bypasses the whitelist data service entirely, reading directly from DDB tables owned by other services (Patron, DiscordPatreonLink, GroupWhitelist) and calling the staff data service and user data service at generation time. This breaks service ownership boundaries and creates expensive cross-service API calls on every 3-minute refresh cycle.

## Requirements

1. The whitelist service owns all whitelist-related data in its own DDB tables
2. The event orchestrator (see `services/event-orchestrator/design-doc.md`) pushes enriched events to the whitelist service when upstream data changes
3. The whitelist service is the sole writer to the GroupWhitelist table
4. Config assembly happens inside the service, reading from its own DDB tables plus one `list_users()` call to the user data service for Discord→Steam resolution
5. Group members are stored as Discord IDs (primary) with support for Steam IDs (e.g., seeders from BattleMetrics webhooks)
6. S3-based change detection is kept (current approach is sufficient)
7. Server config stays in TOML
8. Drift is managed through periodic reconciliation

## Background

### Identity Model

All bot interactions (Discord commands, promotion groups, friend lists) use Discord IDs. Game server whitelist configs use Steam IDs (`Admin=<steam_id>:<group>`). The user data service owns the Discord↔Steam link.

The whitelist service stores members primarily by Discord ID, since that's the natural identifier from bot interactions. Some groups (seeders from BattleMetrics webhooks) use Steam IDs directly. At config generation time, the service calls `user_data_service.list_users()` once to build a `HashMap<DiscordID, SteamID>` and resolves all Discord IDs to Steam IDs for the output config. Members without a linked Steam account simply don't appear in the generated config — correct behavior, since you can't whitelist someone on a game server without a Steam ID.

### Current Event Flow

- EventBridge schedule (every 3 min) → SNS `WhitelistUpdateNotification` → `whitelist_update_handler`
- Discord handler can also publish to `WhitelistUpdateNotification` (manual trigger)
- `whitelist_update_handler` publishes to `WhitelistReadyNotification` after success

### Event Orchestrator Integration

The event orchestrator (proposed) is a central hub-and-spoke service that receives thin signals from producers, enriches them by querying relevant services, and dispatches enriched events to registered consumers. This replaces the need for a custom ingest handler in the whitelist service.

The whitelist service is a **consumer** of these enriched events:

| Signal | Enriched Payload Received | Whitelist Service Action |
|--------|--------------------------|-------------------------|
| `steam_linked` | discord_id, steam_id, supporter_tier?, staff_rank? | No table change needed (Discord IDs stored, Steam resolution at gen time); can trigger refresh |
| `steam_unlinked` | discord_id, old_steam_id, supporter_tier?, staff_rank? | Flag for review / reconciliation |
| `tier_changed` | discord_id, steam_id?, old_tier, new_tier | Upsert/remove supporter whitelist entry |
| `supporter_discord_linked` | discord_id, steam_id?, tier | Upsert supporter whitelist entry |
| `supporter_discord_unlinked` | old_discord_id, tier | Remove supporter whitelist entry |
| `rank_changed` | discord_id, steam_id?, old_rank, new_rank | Upsert/remove staff whitelist entry |
| `friend_updated` | owner_discord_id, friend_discord_id, action, source | Update friend list on staff/supporter entry |
| `left_server` | discord_id, was_supporter?, was_staff? | Remove staff/supporter entries |

The whitelist service is also a **producer** of:

| Signal | When | Consumers |
|--------|------|-----------|
| `promotion_group_changed` | Group member added/removed | Discord Roles |

## Proposed Solution

### Architecture

```
                    ┌─────────────────────┐
                    │  Event Orchestrator  │
                    │  (enriched events)   │
                    └──────────┬──────────┘
                               │
          ┌────────────────────▼────────────────────┐
          │           Whitelist Service              │
          │                                         │
          │  ┌─────────────┐  ┌──────────────────┐  │
          │  │  Service     │  │  Updater Lambda  │  │
          │  │  Lambda      │  │  (thin: calls    │  │
          │  │  (API +      │  │   GenerateConfig │  │
          │  │   consumer)  │  │   + S3 diff      │  │
          │  └──────┬───────┘  │   + server push) │  │
          │         │          └────────┬─────────┘  │
          │         │                   │            │
          │  ┌──────▼───────────────────▼─────────┐  │
          │  │         Owned DDB Tables           │  │
          │  │  ┌──────────┐  ┌────────────────┐  │  │
          │  │  │ Staff WL │  │ Supporter WL   │  │  │
          │  │  │ discord  │  │ discord_id →   │  │  │
          │  │  │ _id →    │  │ tier, friends, │  │  │
          │  │  │ rank,    │  │ transfers,     │  │  │
          │  │  │ friends  │  │ sources        │  │  │
          │  │  └──────────┘  └────────────────┘  │  │
          │  │  ┌──────────┐  ┌────────────────┐  │  │
          │  │  │ Group WL │  │ Vouchers       │  │  │
          │  │  │ promo    │  │                │  │  │
          │  │  │ groups + │  │                │  │  │
          │  │  │ timed    │  │                │  │  │
          │  │  │ members  │  │                │  │  │
          │  │  └──────────┘  └────────────────┘  │  │
          │  └────────────────────────────────────┘  │
          │                                         │
          │  ┌────────────────────────────────────┐  │
          │  │  S3 Bucket (per-server diff cache) │  │
          │  └────────────────────────────────────┘  │
          └─────────────────────────────────────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
          AWN API          PSG SFTP     User Data Service
         (push cfg)       (push cfg)   (list_users at gen time)
```

### Data Flow

**Event-driven updates (via orchestrator):**
1. Upstream service changes data (e.g., staff rank change)
2. Upstream service signals the orchestrator (thin signal)
3. Orchestrator enriches the signal (queries user data service for Steam ID, etc.)
4. Orchestrator dispatches enriched event to whitelist service
5. Whitelist service updates its own tables

**Scheduled config generation (every 3 min):**
1. EventBridge → SNS → Updater Lambda
2. Updater calls `GenerateConfig(server_id)` on the whitelist service for each server
3. Whitelist service reads its own DDB tables + calls `user_data_service.list_users()` for Discord→Steam map
4. Whitelist service assembles the config string (groups + entries)
5. Updater diffs against S3, pushes to server if changed via AWN/PSG

**Manual operations (Discord commands, seeder API):**
1. Caller invokes whitelist service API (e.g., `UpsertGroupMember`)
2. Whitelist service updates GroupWhitelist table
3. Whitelist service signals `promotion_group_changed` to orchestrator (fire-and-forget)

### Key Design Decisions

**No identity map table.** The whitelist service does not duplicate the Discord→Steam mapping. It stores members by Discord ID and resolves to Steam IDs at config generation time via a single `list_users()` call. This avoids data duplication and keeps the user data service as the single source of truth for identity.

**Discord ID as primary, Steam ID as secondary.** Group members, staff entries, and supporter entries are keyed by Discord ID. Steam ID members are supported for groups that receive members from external systems (BattleMetrics webhooks for seeders). This matches how the bot naturally operates.

**Orchestrator replaces ingest handler.** Rather than building a custom SNS subscriber to translate upstream events, the event orchestrator handles signal reception, enrichment, and dispatch. The whitelist service just exposes a consumer endpoint.

**S3-based change detection kept.** The current approach (diff against last-pushed config in S3, 5-minute minimum interval via S3 timestamp) is sufficient. No request queue or cooldown mechanism needed.

**Server config stays in TOML.** Server-to-provider mappings, group permissions, and server-specific settings remain in the TOML config layer. Not worth moving to DDB since servers change infrequently.

**Reconciliation for drift.** A periodic full-sync (hourly) from upstream services catches missed orchestrator events. The 3-minute refresh cycle also acts as a natural safety net — even if an event is missed, the next config generation will produce the correct output as long as the owned tables are eventually consistent.

## API Surface

### Management Operations

```
UpsertStaffWhitelist { discord_id, rank, friends: Vec<DiscordID> }
RemoveStaffWhitelist { discord_id }

UpsertSupporterWhitelist { discord_id, tier, friends: Vec<DiscordID>, transfers, sources }
RemoveSupporterWhitelist { discord_id }

CreateGroup { name, description?, max_count?, manager? }
DeleteGroup { name }
UpsertGroupMember { group, id: DiscordOrSteam, expires_at? }
RemoveGroupMember { group, id: DiscordOrSteam }
GetGroup { name }
ListGroups

LookupByDiscordId { discord_id } → Vec<GroupMembership>
LookupBySteamId { steam_id } → Vec<GroupMembership>

GenerateConfig { server_id } → String

HandleEnrichedEvent { event } (orchestrator consumer endpoint)
Reconcile (triggers full upstream sync)
```

### Existing Operations (kept)

```
ListSteamIds → HashSet<String>
ListSteamIdsByGroup → HashMap<String, HashSet<String>>
```

## Task Breakdown

### Task 1: Define the whitelist service API contract

Extend `WhitelistDataServiceRequest`/`WhitelistDataServiceResponse` enums with all new operations listed above. Update the API crate (`services/whitelist_data_service/api/`) with new operations and client methods.

- **Test:** Serde round-trip tests for all new variants
- **Demo:** All API messages serialize/deserialize correctly, existing operations unchanged

### Task 2: Implement staff and supporter whitelist data storage

Store whitelist-relevant staff and supporter data in the service's own tables.

- Staff table: keyed by `discord_id`, stores `rank`, `friends: Vec<DiscordID>`
- Supporter table: keyed by `discord_id`, stores `tier`, `friends: Vec<DiscordID>`, `transfers`, `sources`

Implement DDB read/write operations and service handlers for upsert/remove.

- **Test:** CRUD integration tests against local DynamoDB
- **Demo:** Can upsert and query staff/supporter whitelist entries via the service API

### Task 3: Consolidate group management through the service API

Route all GroupWhitelist writes through the whitelist service.

- `UpsertGroupMember` accepts a `DiscordOrSteam` enum — Discord IDs for promotion commands, Steam IDs for seeders
- Include timed membership logic (WL-001/WL-002)
- After group changes, signal `promotion_group_changed` to the orchestrator (fire-and-forget)

- **Test:** Unit tests for group operations including timed membership, permanent member preservation
- **Demo:** Full group lifecycle through the service API

### Task 4: Implement orchestrator consumer endpoint

Handle enriched events dispatched by the event orchestrator.

- Each event type maps to a specific table update (see enriched events table above)
- `steam_linked`/`steam_unlinked` don't require table changes since Discord IDs are stored; can optionally trigger a whitelist refresh
- Each handler is idempotent

- **Test:** Unit tests for each event type with mock data
- **Demo:** Sending a `tier_changed` enriched event updates the supporter whitelist table

### Task 5: Implement Discord ID and Steam ID lookup

- `LookupByDiscordId`: Search staff table, supporter table, and all groups for entries containing that Discord ID
- `LookupBySteamId`: Search all groups for entries containing that Steam ID
- Return: list of `{ group_name, membership_type, expires_at? }` entries

- **Test:** Unit tests with various membership scenarios
- **Demo:** Can look up a Discord user and see all their whitelist group memberships

### Task 6: Move config assembly into the service

Implement `GenerateConfig { server_id }`:

1. Call `user_data_service.list_users()` once to build `HashMap<DiscordID, SteamID>`
2. Read staff entries → resolve friends via map → produce staff groups + entries
3. Read supporter entries → resolve via map → produce supporter groups + entries
4. Read promotion groups → resolve Discord ID members via map, keep Steam ID members as-is → produce promotion entries
5. Read vouchers → produce voucher entries
6. Combine, sort, return as whitelist config string

TOML config (group names, permissions per server) read from config layer. Reuse `Group` and `Entry` types from `whitelist_api::game`.

- **Test:** Unit tests verifying correct output from test data, including Discord→Steam resolution and handling of unlinked members
- **Demo:** `GenerateConfig("Invasion")` returns a complete whitelist config string

### Task 7: Rewire the whitelist update handler

Make the update handler a thin lambda: for each server, call `GenerateConfig(server_id)`, diff against S3, push if changed. Remove all direct DDB reads and upstream service calls. Keep S3 data store logic and AWN/PSG push logic as-is.

CDK: Remove table grants and upstream service invokes, keep whitelist service invoke + S3 + AWN/PSG.

- **Test:** Integration test with mock service returning known config
- **Demo:** Whitelist generation works end-to-end using only the whitelist service

### Task 8: Migrate existing consumers to use the service API

- `api_seeder_whitelist_handler`: Replace `GroupWhitelistData` calls with `UpsertGroupMember` (Steam ID, 7-day expiry)
- Discord promotion commands: Replace `GroupWhitelistData` calls with service API calls
- Discord staff commands (rank change, friend add/remove): Add direct calls to the whitelist service as a bridge until the orchestrator is deployed

CDK: Remove direct table grants, grant whitelist service invoke.

- **Test:** Existing tests pass with mocked service
- **Demo:** All group management flows through the whitelist service

### Task 9: Implement periodic reconciliation

`Reconcile` operation calls upstream services (staff data service, supporter data service, user data service) and compares with owned data. Log discrepancies, update owned tables, include expired membership cleanup.

Trigger via separate EventBridge schedule (hourly) or a "last reconciled" timestamp check in the 3-minute cycle.

CDK: Grant upstream service invokes to whitelist service (only used during reconciliation), add schedule.

- **Test:** Unit test verifying drift detection and correction
- **Demo:** After a missed event, reconciliation restores correct state

### Task 10: Clean up deprecated code paths

- Remove `Whitelist::assemble` (v1) and the commented-out `update_whitelist_v2`
- Remove `update_active` lambda references (no longer in use)
- Remove direct DDB table references from update handler environment
- Evaluate whether `whitelist_api` lib can be reduced to shared types or removed
- Remove unused Cargo dependencies
- Update CDK to reflect final table ownership

- **Test:** Full build succeeds, all tests pass
- **Demo:** Clean codebase, single code path, no dead code

## Dependencies

- **Event Orchestrator Service** — Tasks 4 and the orchestrator-dependent parts of Task 8 require the event orchestrator to be deployed. Other tasks can proceed independently.
- **User Data Service** — The `list_users()` API must remain available for config generation (Task 6) and reconciliation (Task 9).
- **Staff Data Service** — Read access needed for reconciliation only (Task 9).
- **Supporter Data Service** — Read access needed for reconciliation only (Task 9).

## Migration Strategy

Tasks 1–3 and 5–7 can be implemented without the event orchestrator. The whitelist service can be deployed and used for group management and config generation while the orchestrator is being built. Task 8 bridges the gap by having Discord commands call the whitelist service directly for staff/supporter updates until the orchestrator flow is live. Task 4 is wired up once the orchestrator is deployed.
