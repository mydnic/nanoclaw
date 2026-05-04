# Migration: v1 → v2 (2026-05-04)

## Summary

Migrated NanoClaw from v1 architecture to v2. Migration ran at 19:06 UTC on 2026-05-04.

## What changed

- **Architecture**: v2 uses a central SQLite DB (`data/v2.db`) with entity model (users, agent_groups, messaging_groups, sessions) instead of v1's monolithic store.
- **Channel**: Telegram via Chat SDK (`mydclawdbot`).
- **Agent group**: `telegram_main` → agent "Eru" (`ag-1777921430256-vgcp79`).
- **Session**: One active session (`sess-1777921431519-n24i9j`), two-DB split (inbound.db / outbound.db).
- **Scheduled tasks**: 4 tasks migrated (ABCRECHE Mon, SENDBOO Tue, MyDynamic Wed, DRICLE Thu).

## Fixes applied during migration

1. **Container user permissions** (`src/container-runner.ts`): When host runs as root (uid=0), session DB files are root-owned (0644). The container's `node` user (uid 1000) couldn't write to `outbound.db` or create the heartbeat file. Fix: `chmodSync(sessDir, 0o777)`, `chmodSync(outboundDb, 0o666)`, `chmodSync(claudeDir, 0o777)` before each spawn.

2. **OneCLI Anthropic host pattern**: The Anthropic secret had no host patterns set, so the API key wasn't injected for any requests. Fix: `onecli secrets update --id 09dd0b03-fd7e-4f82-a649-a3711165c0a0 --host-pattern "api.anthropic.com"`.

3. **Stale Claude Code session**: Migration seeded `session_state` with an old v1 session ID that Claude Code couldn't resume. Fix: `DELETE FROM session_state` in the outbound DB.

## Access

- Owner: `telegram:542475468` (Clément)
- Unknown sender policy: `known` (only registered users)
- Anthropic credential: via OneCLI vault (id `09dd0b03-fd7e-4f82-a649-a3711165c0a0`), host pattern `api.anthropic.com`

## Service

Running as `nanoclaw.service` (system-level). Note: the v2 naming convention would be `nanoclaw-v2-7545d4f2` — the verify step reports `service: not_found` because it looks for the v2 name. The service works correctly.
