# Backend Audit

## Backend Summary

The backend is a Go workspace microservices system under `aloqa-backend`. It provides the durable business logic, API gateway, WebSocket gateway, internal gRPC contracts, database migrations, infrastructure compose files, and production deployment definitions.

Source paths:

- `aloqa-backend/go.work`
- `aloqa-backend/Taskfile.yml`
- `aloqa-backend/AGENTS.md`
- `aloqa-backend/shared/api/`
- `aloqa-backend/shared/proto/`
- `aloqa-backend/platform/migrations/`

The backend architecture is clean/layered by convention:

- `cmd/main.go` entrypoints
- `internal/core/app` composition
- feature packages with transport, service, repository, converter layers
- `platform` shared libraries
- `shared` contracts

Source path: `aloqa-backend/AGENTS.md`.

## Workspace Modules

The Go workspace includes:

- `api-gateway`
- `auth-service`
- `file-service`
- `messaging-service`
- `notification-service`
- `org-service`
- `platform`
- `realtime-service`
- `search-service`
- `shared`
- `ws-gateway`

Source path: `aloqa-backend/go.work`.

## Local Development and Operations

`Taskfile.yml` is the operational entrypoint. It includes tasks for:

- starting core infrastructure
- running backend services in development
- generating contracts
- running migrations
- installing local tools under `shared/bin`

Source path: `aloqa-backend/Taskfile.yml`.

The README describes local setup around dependency install, environment creation, infrastructure startup, and service development. Source path: `aloqa-backend/README.md`.

## API Gateway

The API gateway exposes the HTTP API. It translates HTTP/OpenAPI handler calls into backend gRPC service calls and applies gateway-level concerns such as middleware, auth extraction, cookie behavior, request parsing, and response conversion.

Source paths:

- `aloqa-backend/api-gateway/`
- `aloqa-backend/shared/api/api-gateway/v1/api-gateway.openapi.yaml`
- `aloqa-backend/shared/api/api-gateway/v1/paths/`

Important responsibilities:

- auth endpoints
- user/profile endpoints
- company/workspace/channel endpoints
- messaging endpoints
- file endpoints
- meeting endpoints
- search endpoints
- notification endpoints
- admin search reindex endpoint

The gateway has a large public surface: 160 HTTP operations across 138 OpenAPI path items.

## Auth Service

`auth-service` owns identity and account security:

- user registration
- login
- token refresh
- logout and logout-all
- session listing
- email verification and resend
- forgot/reset password
- Google login
- 2FA enable, confirm, disable, login verification, resend
- magic link request and verification
- profile, avatar, settings, and privacy operations where routed through auth/user flows

Source paths:

- `aloqa-backend/auth-service/`
- `aloqa-backend/shared/proto/auth/`
- `aloqa-backend/platform/pkg/utils/auth.go`

The service uses Postgres for durable user/session state and Redis for cache/session-adjacent behavior. It uses bcrypt for password hashing and JWT helpers in `platform`.

## Organization Service

`org-service` owns the organizational model:

- companies
- workspaces
- channels
- members
- public channel discovery
- workspace invites
- company/workspace kicks
- custom roles
- role permissions
- role assignment
- ABAC permission decisions

Source paths:

- `aloqa-backend/org-service/`
- `aloqa-backend/org-service/internal/core/abac/`
- `aloqa-backend/platform/pkg/permissions/`
- `aloqa-backend/shared/proto/org/`

The organization service is one of the most important correctness boundaries because many later operations depend on membership and permission decisions.

## Messaging Service

`messaging-service` owns chat and direct-message behavior:

- send message
- list channel messages
- edit messages
- delete and restore messages
- reactions
- pins and pinned lists
- channel read state and unread count
- threads and replies
- direct-message creation, listing, blocking, unblocking, and deletion
- message forwarding

Source paths:

- `aloqa-backend/messaging-service/`
- `aloqa-backend/shared/proto/messaging/`
- `aloqa-backend/platform/migrations/`

Messaging uses an outbox table to publish durable events to Kafka. This is the right pattern for avoiding "database write succeeded but event publish failed" inconsistencies, but it requires idempotent consumers and monitoring.

## File Service

`file-service` owns file upload and file metadata:

- upload
- file URL/content access
- delete public file
- file shares
- batch share/revoke
- list user files
- file metadata
- storage accounting
- workspace storage and quota
- scan/processing integrations

Source paths:

- `aloqa-backend/file-service/`
- `aloqa-backend/shared/proto/file/`
- `aloqa-backend/platform/migrations/`

The service integrates with MinIO, ClamAV, govips, and likely command-line processors such as ffmpeg or Ghostscript based on package and compose references. File systems are security-sensitive because they combine uploads, scanning, permissions, sharing, and download URLs.

## Notification Service

`notification-service` owns email and web notification behavior:

- verification email
- password reset email
- 2FA email
- magic link email
- web notification creation/listing/read state
- channel notification mutes
- Kafka event consumption for message-created-style events

Source paths:

- `aloqa-backend/notification-service/`
- `aloqa-backend/shared/proto/notification/`

The service uses SMTP/go-mail and integrates with Redis/Kafka paths.

## Realtime Service

`realtime-service` owns meeting domain behavior and LiveKit integration:

- create/update/end meeting
- active meeting lookup
- join meeting and token issuance
- waiting room flows
- meeting chat
- replies and threads
- meeting reactions
- meeting admins
- room settings
- participant permissions
- device permission requests
- pin state
- moderation: mute, kick, ban, unban
- participants list
- breakout rooms
- breakout chat
- breakout waiting/join request flows

Source paths:

- `aloqa-backend/realtime-service/`
- `aloqa-backend/shared/proto/meeting/`
- `aloqa-backend/platform/migrations/`

This service is complex enough to deserve its own test plan. Meeting behavior combines persistent state, ephemeral Redis state, WebSocket events, LiveKit state, admin permissions, and user-specific waiting-room flows.

## WebSocket Gateway

`ws-gateway` owns persistent WebSocket connections and event fanout:

- authenticating WebSocket clients
- subscribing/unsubscribing channels and meetings
- typing events
- device-state events
- reaction events
- ping/pong and room sync
- Kafka consumers for chat and meeting events
- Redis pub/sub for user notifications
- presence and gateway lease behavior

Source paths:

- `aloqa-backend/ws-gateway/`
- `aloqa-backend/ws-gateway/internal/ws/messages.go`
- `aloqa-backend/ws-gateway/internal/ws/client.go`
- `aloqa-backend/shared/proto/presence/`

The WebSocket gateway is a scaling boundary. It must handle reconnects, duplicate subscriptions, auth expiry, sequence/resume behavior, and backpressure.

## Search Service

`search-service` owns OpenSearch-backed search:

- search API
- admin reindex
- Kafka consumers for index updates

Source paths:

- `aloqa-backend/search-service/`
- `aloqa-backend/shared/proto/search/`

Search is operationally separate from source-of-truth Postgres. It should be treated as eventually consistent unless code or product requirements prove otherwise.

## Platform Module

The `platform` module contains shared backend building blocks:

- migrations
- auth/JWT/password helpers
- middleware
- logging
- database helpers
- Redis helpers
- permissions
- common infrastructure code

Source paths:

- `aloqa-backend/platform/`
- `aloqa-backend/platform/migrations/`
- `aloqa-backend/platform/pkg/`

This is a useful shared layer, but it should be protected from becoming a dumping ground. Cross-service abstractions should stay small and stable.

## Shared Contracts

The `shared` module contains source contracts:

- OpenAPI source: `aloqa-backend/shared/api/api-gateway/v1/`
- protobuf source: `aloqa-backend/shared/proto/`

Generated outputs under `shared/pkg` and tools under `shared/bin` are not manual-read/manual-edit targets according to backend instructions. Source path: `aloqa-backend/AGENTS.md`.

## Backend Deployment

Production compose includes infrastructure and services:

- Postgres
- Redis
- MinIO
- ClamAV
- OpenSearch
- Kafka
- LiveKit
- migrations
- notification service
- auth service
- org service
- messaging service
- realtime service
- file service
- search service
- WebSocket gateway
- API gateway
- frontend
- nginx

Source path: `aloqa-backend/deploy/prod/docker-compose.yml`.

The production nginx routes:

- `/api/v1/` to `api-gateway`
- `/files/` to `api-gateway`
- `/docs/` to `api-gateway`
- exact `/ws` to `ws_gateway/ws/chat`
- `/` to frontend

Source path: `aloqa-backend/deploy/prod/nginx/nginx.conf`.

This conflicts with the frontend BFF deployment model if that nginx is the browser-facing edge.

## Backend Risks

### Test Coverage

The backend repo instructions say tests are mostly absent except limited logger benchmark coverage. Source path: `aloqa-backend/AGENTS.md`.

For a service surface this broad, that is the biggest backend engineering risk.

### Migration Complexity

The migration list is long and touches core identity, messaging, files, and meetings. Source path: `aloqa-backend/platform/migrations/`.

The meeting subsystem alone has many migration files. Migration ordering, rollback behavior, and production data compatibility need careful process.

### Security-Critical Flows

Auth, permissions, file access, WebSocket auth, meeting moderation, invite links, and storage quotas are security-sensitive. These need tests beyond unit coverage.

### Operational Ambiguity

Compose files describe a single-host deployment style, but I cannot determine actual production topology, secrets handling, backup policy, or observability coverage from the codebase alone.

### Minor Compose Risk

The backend production compose contains a duplicate `POSTGRES_HOST: postgres` line in the `messaging-service` environment. Source path: `aloqa-backend/deploy/prod/docker-compose.yml`.

This is probably harmless YAML override behavior, but it signals that deployment files need linting.

## Backend Assessment

The backend has a coherent microservice architecture and a substantial feature implementation. The biggest gap is verification. The next engineering investment should be integration tests and contract tests around auth, permissions, chat, files, meetings, and realtime fanout.
