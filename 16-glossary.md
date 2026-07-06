# Glossary

## ABAC

Attribute-based access control. In Aloqa, ABAC appears in the organization service and platform permissions code to decide whether a user can perform an action in a company, workspace, channel, or related resource.

Source paths:

- `aloqa-backend/org-service/internal/core/abac/`
- `aloqa-backend/platform/pkg/permissions/`

## API Gateway

The backend HTTP entrypoint. It exposes OpenAPI routes and calls backend services through gRPC.

Source path: `aloqa-backend/api-gateway/`.

## BFF

Backend for frontend. In Aloqa web, the Next.js app has route handlers that receive browser `/api/*` calls and forward them to the backend gateway while managing sealed backend sessions.

Source paths:

- `aloqa-frontend/docs/adr/0037-web-bff-backend-session.md`
- `aloqa-frontend/apps/web/app/api/`

## Breakout Room

A meeting sub-room used during calls. Aloqa supports breakout rooms, participants, chat, invitations, waiting/join requests, and close/move flows.

Source paths:

- `aloqa-backend/shared/proto/meeting/`
- `aloqa-backend/platform/migrations/20260622080000_breakout_rooms.*`
- `aloqa-backend/platform/migrations/20260624100000_breakout_room_chat.*`

## Channel

A workspace conversation space. Channels can have members, messages, public join behavior, archive/unarchive behavior, mute state, and permissions.

Source paths:

- `aloqa-backend/org-service/`
- `aloqa-backend/messaging-service/`
- `aloqa-backend/platform/migrations/20260504074928_init.*`

## Company

Top-level organizational container. Companies contain members, roles, workspaces, and permissions.

Source path: `aloqa-backend/org-service/`.

## Contract Drift

When backend OpenAPI/protobuf contracts, frontend route helpers, generated clients, or documentation no longer agree.

Important paths:

- `aloqa-backend/shared/api/api-gateway/v1/`
- `aloqa-backend/shared/proto/`
- `aloqa-frontend/packages/core/src/api/routes.ts`

## CSRF

Cross-site request forgery. The web BFF uses CSRF checks for mutating cookie-authenticated requests.

Source path: `aloqa-frontend/apps/web/src/lib/auth/withCsrf.ts`.

## Direct Message

A private conversation channel between users. Aloqa models DMs alongside channel/chat behavior with additional DM-specific tables and privacy/blocking behavior.

Source paths:

- `aloqa-backend/platform/migrations/20260522064024_dm_channels.*`
- `aloqa-backend/platform/migrations/20260617000001_dm_chat_logic.*`
- `aloqa-backend/messaging-service/`

## Gateway

Usually refers to either the API gateway or WebSocket gateway. Be explicit when discussing architecture.

## gRPC

Backend service-to-service API protocol. Aloqa defines gRPC services in protobuf files under `shared/proto`.

Source path: `aloqa-backend/shared/proto/`.

## Kafka

Event broker used for asynchronous service communication and event fanout. Aloqa uses Kafka with outbox-style publishing for messaging, meeting, notification, and search-related flows.

Source paths:

- `aloqa-backend/deploy/prod/docker-compose.yml`
- `aloqa-backend/messaging-service/`
- `aloqa-backend/realtime-service/`
- `aloqa-backend/search-service/`

## LiveKit

Media infrastructure used for audio/video calls. Aloqa uses LiveKit rather than building custom WebRTC media infrastructure.

Source paths:

- `aloqa-frontend/docs/adr/0022-livekit-client-sdk.md`
- `aloqa-backend/realtime-service/`
- `aloqa-backend/deploy/prod/docker-compose.yml`

## Meeting

A video/call session with participants, room settings, admins, permissions, waiting room, chat, reactions, device requests, moderation, and breakout rooms.

Source paths:

- `aloqa-backend/realtime-service/`
- `aloqa-backend/shared/proto/meeting/`
- `aloqa-backend/platform/migrations/`

## MinIO

Object storage used for uploaded files.

Source paths:

- `aloqa-backend/file-service/`
- `aloqa-backend/deploy/prod/docker-compose.yml`

## OpenAPI

HTTP API contract format. Aloqa's API gateway OpenAPI source lives under `shared/api/api-gateway/v1`.

Source path: `aloqa-backend/shared/api/api-gateway/v1/api-gateway.openapi.yaml`.

## OpenSearch

Search engine used by the search service.

Source paths:

- `aloqa-backend/search-service/`
- `aloqa-backend/deploy/prod/docker-compose.yml`

## Outbox

A database table pattern where a service writes business data and an event row in the same transaction. A relay later publishes the event to Kafka.

Source paths:

- `aloqa-backend/platform/migrations/20260610000001_messaging_outbox.*`
- `aloqa-backend/platform/migrations/20260612000001_channels_outbox.*`
- `aloqa-backend/platform/migrations/20260622085254_meeting_outbox.*`

## Platform-First Frontend

Frontend architecture where shared packages reuse business logic, not platform UI. Web, desktop, and mobile own their UI surfaces.

Source path: `aloqa-frontend/docs/adr/0023-platform-first-architecture.md`.

## Redis

Fast key-value store used for cache, session-adjacent state, pub/sub, presence, room state, typing, and other realtime coordination.

Source paths:

- `aloqa-backend/realtime-service/internal/infrastructure/redis/`
- `aloqa-backend/ws-gateway/`
- `aloqa-backend/deploy/prod/docker-compose.yml`

## Role

A named set of permissions assigned to users in company/workspace contexts. Aloqa supports custom roles and user-role assignments.

Source paths:

- `aloqa-backend/platform/migrations/20260520061940_add_abac_roles.*`
- `aloqa-backend/platform/migrations/20260604000001_custom_roles_v2.*`
- `aloqa-backend/org-service/`

## Sealed Session

The web BFF's encrypted/authenticated cookie containing backend token material. It is stored in the `aloqa_bff_session` HttpOnly cookie.

Source path: `aloqa-frontend/apps/web/src/lib/auth/sessionCookie.ts`.

## WebSocket Gateway

Backend service that holds persistent WebSocket connections, accepts subscriptions, consumes Kafka events, and fans out realtime updates.

Source path: `aloqa-backend/ws-gateway/`.

## Workspace

A collaboration space inside a company. Workspaces contain members, channels, invites, storage settings, and quota behavior.

Source paths:

- `aloqa-backend/org-service/`
- `aloqa-backend/platform/migrations/20260504074928_init.*`

## WS Ticket

A short-lived token issued by the web BFF so browser clients can connect directly to the WebSocket gateway without exposing long-lived backend token material.

Source path: `aloqa-frontend/apps/web/app/api/realtime/ws-ticket/route.ts`.
