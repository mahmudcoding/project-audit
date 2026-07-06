# Executive Summary

## What Aloqa Is

Aloqa is a Slack-like collaboration platform with chat, workspaces, companies, channels, direct messages, files, search, notifications, and a large video-meeting subsystem. It is implemented as two coordinated monorepos under `/Users/mahmud/Projects/aloqa`:

- Backend: `aloqa-backend`, a Go workspace of microservices behind an API gateway and WebSocket gateway.
- Frontend: `aloqa-frontend`, a platform-first TypeScript monorepo for web, desktop, and mobile clients.

The backend owns persistence, contracts, business rules, service-to-service gRPC, WebSocket fanout, Kafka outbox pipelines, LiveKit integration, and production infrastructure. The frontend owns platform clients, user workflows, design systems, web BFF behavior, client-side state, offline/mobile concerns, and release checks.

Important source paths:

- `aloqa-backend/go.work`
- `aloqa-backend/Taskfile.yml`
- `aloqa-backend/shared/api/api-gateway/v1/api-gateway.openapi.yaml`
- `aloqa-backend/shared/proto/`
- `aloqa-backend/platform/migrations/`
- `aloqa-frontend/AGENTS.md`
- `aloqa-frontend/apps/`
- `aloqa-frontend/packages/`
- `aloqa-frontend/docs/adr/`

## Current Engineering Shape

The project is well beyond a prototype. It has:

- A multi-service backend workspace with auth, organization, messaging, file, notification, realtime, search, API gateway, and WebSocket gateway services.
- A broad HTTP contract with 160 OpenAPI operations across 138 path items.
- gRPC contracts for auth, organization, messaging, files, meetings, notifications, presence, and search.
- A substantial PostgreSQL schema with identity, workspace, role, channel, message, file, meeting, breakout-room, notification, outbox, and moderation tables.
- A frontend architecture that deliberately separates headless feature logic from platform-owned UI.
- Web, desktop, and mobile app surfaces with separate runtime assumptions.
- Production compose and nginx files in both repos.

At the same time, it has the risk profile of a fast-moving product whose surface area has grown faster than verification and deployment consolidation:

- Backend test coverage appears thin compared with the number of services and migrations.
- Frontend architecture docs are stronger than parts of the legacy package shape, especially where feature packages still expose `ui-web`, `ui-desktop`, and `ui-mobile`.
- The backend and frontend disagree in important places about who owns production edge routing for `/api/*`.
- Auth is split across backend cookies and frontend BFF sealed sessions. This is reasonable, but subtle.
- Realtime depends on multiple systems staying aligned: Postgres outbox, Kafka, WebSocket gateway, Redis state, LiveKit, and frontend event mapping.

## Main Strengths

The backend has clear service separation. The Go workspace includes independent modules for gateway, auth, organization, messaging, files, notifications, realtime, search, platform, shared contracts, and WebSocket gateway. Source paths: `aloqa-backend/go.work`, `aloqa-backend/*/go.mod`.

The contract-first backend foundation is strong. HTTP routes come from OpenAPI files under `aloqa-backend/shared/api/api-gateway/v1/`, while internal service contracts are under `aloqa-backend/shared/proto/`. Generated code is intentionally excluded from manual editing by `aloqa-backend/AGENTS.md`.

The frontend has a serious architecture model. ADR-0023 defines "reuse logic, not UI", and ADR-0037 defines the web BFF. Source paths: `aloqa-frontend/docs/adr/0023-platform-first-architecture.md`, `aloqa-frontend/docs/adr/0037-web-bff-backend-session.md`.

The data model covers real collaboration-product complexity. The migrations under `aloqa-backend/platform/migrations/` include users, sessions, companies, workspaces, channels, messages, files, custom roles, ABAC permissions, direct messages, meeting chat, meeting admins, room settings, device requests, breakout rooms, waiting rooms, bans, and outbox tables.

The project has operational thinking. There are local Taskfile workflows, production compose files, release CI/CD docs, smoke scripts, and nginx configurations. Source paths: `aloqa-backend/Taskfile.yml`, `aloqa-backend/deploy/prod/docker-compose.yml`, `aloqa-frontend/docs/CICD.md`, `aloqa-frontend/deploy/smoke-live.sh`.

## Highest Risks

### 1. Edge Routing Drift

The frontend architecture says browser REST traffic should go through the Next.js BFF under `/api/*`, where the BFF manages a sealed `aloqa_bff_session` cookie and forwards backend `access_token` cookies server-side. Source paths: `aloqa-frontend/docs/adr/0037-web-bff-backend-session.md`, `aloqa-frontend/apps/web/app/api/[...path]/route.ts`.

The backend production nginx file routes `/api/v1/` directly to `api-gateway`. Source path: `aloqa-backend/deploy/prod/nginx/nginx.conf`.

The frontend production nginx file routes `/api/*` to the web app. Source path: `aloqa-frontend/deploy/nginx.prod.conf`.

This is the most important deployment ambiguity. I cannot determine from the codebase which nginx file is the actual production edge today. If the backend nginx file is active for browser traffic, it can bypass the frontend BFF model for `/api/v1/*`.

### 2. Backend Verification Gap

The backend has many services and a very large data model, but its own repository instructions note that tests are mostly absent except limited logger benchmarking. Source path: `aloqa-backend/AGENTS.md`.

Given the number of auth, permission, realtime, file, and meeting workflows, this is a high-severity maintenance risk.

### 3. Contract and Implementation Drift

There are three contract layers:

- HTTP OpenAPI in `aloqa-backend/shared/api/api-gateway/v1/`
- gRPC proto files in `aloqa-backend/shared/proto/`
- Frontend API route registry in `aloqa-frontend/packages/core/src/api/routes.ts`

This is a good design, but it requires active drift checks. The frontend route registry includes comments for superseded routes and feature areas that may not be fully represented in the backend OpenAPI. That means product work can accidentally proceed in UI before backend support exists, or backend routes can change without client updates.

### 4. Realtime Complexity

Chat and meetings use a multi-hop realtime path:

1. HTTP or gRPC command writes to Postgres.
2. Service writes an outbox row.
3. Relay publishes Kafka events.
4. WebSocket gateway consumes events.
5. WebSocket gateway fans out to channel or meeting subscribers.
6. Redis stores room, presence, typing, reaction, or notification state.
7. Frontend realtime client normalizes backend event frames.

Source paths: `aloqa-backend/messaging-service/`, `aloqa-backend/realtime-service/`, `aloqa-backend/ws-gateway/`, `aloqa-backend/platform/migrations/`, `aloqa-frontend/packages/core/src/realtime/`.

This architecture can scale, but it needs integration tests, event versioning, idempotency rules, and operational visibility.

### 5. Documentation Drift

Some repository guidance appears older than current implementation. For example, backend guidance still references earlier stub-service status, while the inspected repo contains implemented file, messaging, notification, realtime, search, and WebSocket services. Source paths: `aloqa-backend/AGENTS.md`, `aloqa-backend/file-service/`, `aloqa-backend/messaging-service/`, `aloqa-backend/realtime-service/`, `aloqa-backend/search-service/`, `aloqa-backend/ws-gateway/`.

Documentation drift is not cosmetic here. It can cause engineers to make wrong architecture or release decisions.

## Recommended Priorities

### Priority 1: Decide and Document the Production Edge

Choose one authoritative edge-routing model:

- Preferred for the frontend architecture: browser `/api/*` goes to Next.js BFF; direct backend routes remain internal or service-to-service.
- WebSocket `/ws/chat` goes to `ws-gateway`.
- LiveKit traffic goes to LiveKit.
- File download behavior is explicitly documented as BFF-proxied or gateway-direct.

Then update one deployment path so backend and frontend nginx definitions cannot diverge.

### Priority 2: Add Contract Drift Checks

Add automated checks that compare:

- OpenAPI paths in `aloqa-backend/shared/api/api-gateway/v1/`
- frontend route builders in `aloqa-frontend/packages/core/src/api/routes.ts`
- generated clients or BFF assumptions

This should fail CI when frontend references a backend path that is not in the OpenAPI contract, unless the route is intentionally marked as planned or app-local.

### Priority 3: Build Backend Integration Tests Around Core Workflows

The first backend tests should cover high-risk flows:

- register, login, refresh, logout
- company/workspace/channel creation with permissions
- send message and receive WebSocket event
- file upload, scan, metadata, share, revoke
- meeting join, waiting room, admin permission, LiveKit token
- outbox relay idempotency

### Priority 4: Stabilize Realtime Event Contracts

Document versioned event schemas for chat, notification, meeting, breakout, device, typing, pin, and reaction events. The frontend already has event mappings in `aloqa-frontend/packages/core/src/realtime/events.ts`; backend producers should be tested against that map.

### Priority 5: Reduce Legacy Frontend UI Package Exports

ADR-0023 says feature packages should be headless and app UI should be app-owned. Several packages still export platform UI entrypoints. Keep the rule, but migrate package by package rather than attempting a broad rewrite.

## What I Cannot Determine From Code Alone

I cannot determine the live production edge routing without server inspection.

I cannot determine production traffic volume, SLOs, alert coverage, or actual incident history from the repository.

I cannot determine whether every OpenAPI operation has a fully exercised frontend user flow.

I cannot determine whether Kafka, Redis, MinIO, OpenSearch, ClamAV, and LiveKit are hardened in the actual production environment beyond what compose files describe.

I cannot determine whether all migrations have been applied to production successfully.

## Bottom Line

Aloqa has a serious product architecture and a broad implemented feature surface. The main engineering problem is no longer "can this be built"; it is "can this be safely evolved." The next phase should focus on deployment ownership, contract drift prevention, backend integration coverage, and realtime observability.
