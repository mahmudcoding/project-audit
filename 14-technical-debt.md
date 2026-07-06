# Technical Debt

## Debt Summary

The most important debt in Aloqa is not messy code style. It is system coordination debt:

- deployment assumptions differ across repos
- contract layers can drift
- backend coverage is thin
- realtime behavior spans too many systems without visible contract tests
- frontend architecture is in transition from shared UI packages to platform-owned UI

## P0: Decide the Production Edge

### Problem

The frontend BFF architecture expects browser `/api/*` traffic to reach Next.js. The frontend production nginx supports that model.

Source paths:

- `aloqa-frontend/docs/adr/0037-web-bff-backend-session.md`
- `aloqa-frontend/deploy/nginx.prod.conf`

The backend production nginx routes `/api/v1/` directly to the backend API gateway.

Source path: `aloqa-backend/deploy/prod/nginx/nginx.conf`.

### Impact

If the backend nginx is the browser-facing edge, web auth/session assumptions can be wrong. This can affect CSRF, cookie handling, refresh behavior, and token exposure assumptions.

### Fix

Choose and document one edge configuration. Make the inactive config impossible to deploy accidentally, or generate both from one source.

## P0: Add Backend Integration Tests for Core Flows

### Problem

Backend repository guidance says tests are mostly absent except limited logger benchmark coverage.

Source path: `aloqa-backend/AGENTS.md`.

### Impact

The backend has high-risk flows in auth, ABAC, messaging, files, meetings, and realtime. Bugs can cross service boundaries and may not appear in isolated manual tests.

### Fix

Start with integration tests for:

- register/login/refresh/logout
- create company/workspace/channel
- role assignment and permission denial
- send message and outbox event
- WebSocket subscribe and message fanout
- file upload/share/revoke
- meeting join/waiting/admit
- meeting admin permission update

## P0: Automate Contract Drift Checks

### Problem

Contracts exist in multiple places:

- backend OpenAPI
- backend protobuf
- frontend route registry
- frontend BFF behavior

Source paths:

- `aloqa-backend/shared/api/api-gateway/v1/`
- `aloqa-backend/shared/proto/`
- `aloqa-frontend/packages/core/src/api/routes.ts`

### Impact

Frontend can call routes that backend does not expose, backend can change routes without frontend updates, and generated code can silently drift.

### Fix

Add CI checks:

- route registry path exists in OpenAPI
- OpenAPI generated client is up to date
- protobuf generated code is up to date
- breaking-change checks for public contracts

## P1: Version Realtime Event Schemas

### Problem

Realtime events are shared implicitly between backend producers, WebSocket gateway, and frontend event maps.

Source paths:

- `aloqa-backend/ws-gateway/`
- `aloqa-backend/messaging-service/`
- `aloqa-backend/realtime-service/`
- `aloqa-frontend/packages/core/src/realtime/events.ts`

### Impact

Event fields or names can drift and break clients at runtime.

### Fix

Define event schema files and versioning rules. Add tests that replay backend event fixtures through frontend parsers.

## P1: Add Outbox Observability

### Problem

Outbox tables are present, but this audit cannot determine whether production has visibility into lag, retry, or stuck events.

Source paths:

- `aloqa-backend/platform/migrations/20260610000001_messaging_outbox.*`
- `aloqa-backend/platform/migrations/20260612000001_channels_outbox.*`
- `aloqa-backend/platform/migrations/20260622085254_meeting_outbox.*`

### Impact

Chat, channel, meeting, search, and notification behavior can silently lag or stop.

### Fix

Track:

- oldest unprocessed event age
- per-topic publish error count
- retry count
- dead-letter count
- Kafka consumer lag

## P1: Clarify Table Ownership

### Problem

Multiple services share one Postgres schema. That is pragmatic, but table ownership must be explicit.

Source path: `aloqa-backend/platform/migrations/`.

### Impact

Services can accidentally depend on each other's internal tables or bypass domain rules.

### Fix

Add a table ownership document:

- table
- owning service
- allowed readers
- allowed writers
- public contract, if any

## P1: Resolve Frontend Feature Package Migration

### Problem

ADR-0023 says app UI should be app-owned and feature packages should be headless. Some feature packages still expose platform UI entrypoints.

Source paths:

- `aloqa-frontend/docs/adr/0023-platform-first-architecture.md`
- `aloqa-frontend/packages/features/`

### Impact

Shared UI can become the wrong abstraction for platform-specific UX and can slow platform-specific iteration.

### Fix

For each feature package, classify:

- already headless
- transitional
- should move UI to app
- allowed exception

Then migrate only when touching that feature.

## P1: Harden File Security Tests

### Problem

Files touch upload, scan, storage, sharing, quotas, and content access.

Source paths:

- `aloqa-backend/file-service/`
- `aloqa-backend/platform/migrations/20260608000003_files.*`
- `aloqa-backend/platform/migrations/20260609000002_file_shares.*`
- `aloqa-frontend/apps/web/app/api/upload/[...path]/route.ts`

### Impact

File bugs can become data exposure or malware-handling issues.

### Fix

Add tests for:

- unauthorized file access
- revoked share access
- workspace boundary access
- scan failure behavior
- quota enforcement
- deleted file access
- meeting attachment access

## P1: Meeting State-Machine Tests

### Problem

Meeting behavior has many tables, events, permissions, and UI states.

Source paths:

- `aloqa-backend/realtime-service/`
- `aloqa-backend/shared/proto/meeting/`
- `aloqa-backend/platform/migrations/`
- `aloqa-frontend/packages/features/calls/`

### Impact

Edge cases can break live calls: waiting room, bans, admin permissions, device requests, breakout rooms, and LiveKit state can disagree.

### Fix

Create a meeting test suite around state transitions.

## P2: Deployment File Linting

### Problem

The backend production compose has a duplicate `POSTGRES_HOST: postgres` line under `messaging-service`.

Source path: `aloqa-backend/deploy/prod/docker-compose.yml`.

### Impact

Probably low, but it indicates deployment files are not linted or normalized.

### Fix

Add compose validation and duplicate-key linting to CI.

## P2: Documentation Drift Cleanup

### Problem

Some backend guidance appears stale compared with the implemented services.

Source paths:

- `aloqa-backend/AGENTS.md`
- `aloqa-backend/file-service/`
- `aloqa-backend/messaging-service/`
- `aloqa-backend/realtime-service/`
- `aloqa-backend/search-service/`
- `aloqa-backend/ws-gateway/`

### Impact

New engineers can make wrong decisions based on old status notes.

### Fix

Update repo docs after each milestone. Add dates and ownership to status claims.

## P2: Production Observability Inventory

### Problem

The repos show deployment scripts and service configs, but not a complete observability picture.

### Impact

Incidents can take longer to detect and resolve.

### Fix

Create an observability matrix:

- service
- logs
- metrics
- traces
- dashboards
- alerts
- runbooks

## Technical Debt Assessment

Aloqa's debt is manageable if addressed as system hardening, not cosmetic cleanup. Fix edge routing, tests, contracts, and realtime observability before broad refactors.
