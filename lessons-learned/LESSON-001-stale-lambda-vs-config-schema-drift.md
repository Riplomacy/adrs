# LESSON-001: Stale Lambda Binary Panicked on New Config After Schema Drift

**Date:** 2026-09-18\
**Severity:** Outage\
**Affected:** `bread_leaderboard` (dynamic config system generally)

## What Happened

`BreadLeaderboardHandler` (hourly schedule) panicked on every invocation from 2026-09-14 14:10 UTC through 2026-09-17 — 3 failed attempts per run (Lambda's default async retry), ~72 errors/day, leaderboard silently stopped updating for 3 days. Found via a daily error report. It self-resolved once an unrelated `cargo xtask deploy Bread` (for a different feature) happened to rebuild and redeploy the function's code; the first run afterward (2026-09-18 00:10 UTC) succeeded cleanly.

## Root Cause

`bot_config`'s schema (the Rust structs a Lambda's compiled binary uses to parse `/opt/config`) changed between whenever `bread_leaderboard` was last code-deployed and 2026-09-14 (CloudTrail showed no `UpdateFunctionCode` for it in the prior 90 days). A routine `/config_apply` on 2026-09-14 13:15 UTC pushed new config content, built and validated against **current** `bot_config` — but `bread_leaderboard`'s binary was still running the **old**, pre-drift schema. Its `BotConfig::load()` call (`environment.rs:20`, `.expect("Failed to load configuration file from layer")`) couldn't parse the new content and panicked.

`/config_apply`'s own validation step gave false confidence: it always builds `bot_config` fresh from current source to validate staged content, which proves the change is safe for code at HEAD — not for whatever's actually deployed in each tagged Lambda. Nothing in the apply path checks the other side of that gap.

## Resolution

Incidental — a code deploy for an unrelated feature happened to rebuild `bread_leaderboard`'s binary against current `bot_config`, which resolved the drift. No deliberate fix was applied to `bread_leaderboard` itself for this incident.

## Takeaway

Config content and code are two separate facts that must both be current — one doesn't imply the other, and `/config_apply` only ever manages the content side. After any `bot_config` schema change, redeploy (`cargo xtask deploy`) every config-layer-tagged function that hasn't had a recent code deploy — don't treat `/config_apply` validation passing as proof the change is safe fleet-wide. Rarely-deployed, infrequently-invoked functions are the highest-risk targets since they're least likely to get an incidental redeploy that would otherwise mask this gap: `bread_api_handler`, `bread_pick_winner`, `active_staff_monthly` as of this writing.
