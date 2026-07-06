# Request Flows

## Purpose

This document describes important end-to-end flows across frontend, gateway, backend services, persistence, realtime, and infrastructure.

Source paths used throughout:

- `aloqa-frontend/apps/web/app/api/`
- `aloqa-frontend/packages/core/src/api/`
- `aloqa-frontend/packages/core/src/realtime/`
- `aloqa-backend/api-gateway/`
- `aloqa-backend/auth-service/`
- `aloqa-backend/org-service/`
- `aloqa-backend/messaging-service/`
- `aloqa-backend/file-service/`
- `aloqa-backend/realtime-service/`
- `aloqa-backend/ws-gateway/`
- `aloqa-backend/platform/migrations/`

## Web Login Flow

```mermaid
sequenceDiagram
  participant User
  participant Web as Next.js Web/BFF
  participant Gateway as API Gateway
  participant Auth as Auth Service
  participant DB as Postgres
  User->>Web: submit login
  Web->>Gateway: POST /api/v1/auth/login/user
  Gateway->>Auth: Login gRPC
  Auth->>DB: validate user/session
  Auth-->>Gateway: tokens/session
  Gateway-->>Web: auth result and backend cookies
  Web->>Web: seal backend token material
  Web-->>User: set aloqa_bff_session
```

Source paths:

- `aloqa-backend/shared/api/api-gateway/v1/paths/login_user.yaml`
- `aloqa-backend/auth-service/`
- `aloqa-frontend/apps/web/src/lib/auth/sessionCookie.ts`

The exact web route for login UI depends on the app route implementation. The BFF session sealing path is explicit.

## Web BFF REST Request Flow

```mermaid
sequenceDiagram
  participant Browser
  participant BFF as Next.js /api catch-all
  participant Gateway as API Gateway
  participant Service as Backend Service
  Browser->>BFF: /api/... same-origin request
  BFF->>BFF: read sealed session
  BFF->>Gateway: forward request with access_token cookie
  Gateway->>Service: gRPC call
  Service-->>Gateway: response
  Gateway-->>BFF: HTTP response
  BFF-->>Browser: response
```

If the backend returns 401 and the request is replayable, the BFF can refresh and retry. Source paths:

- `aloqa-frontend/apps/web/app/api/[...path]/route.ts`
- `aloqa-frontend/apps/web/src/lib/auth/sessionRefresh.ts`

This flow depends on edge routing sending browser `/api/*` traffic to Next.js.

## Upload Flow

```mermaid
sequenceDiagram
  participant Browser
  participant UploadBFF as Next.js upload proxy
  participant Gateway as API Gateway
  participant File as File Service
  participant MinIO
  participant ClamAV
  participant DB as Postgres
  Browser->>UploadBFF: upload stream
  UploadBFF->>Gateway: stream upload
  Gateway->>File: upload request
  File->>ClamAV: scan
  File->>MinIO: store object
  File->>DB: write metadata
  File-->>Gateway: file response
  Gateway-->>UploadBFF: response
  UploadBFF-->>Browser: response
```

Source paths:

- `aloqa-frontend/apps/web/app/api/upload/[...path]/route.ts`
- `aloqa-backend/shared/api/api-gateway/v1/paths/file/upload_file.yaml`
- `aloqa-backend/file-service/`
- `aloqa-backend/deploy/prod/docker-compose.yml`

Upload uses a separate BFF route because large streams are not safely replayable after a refresh.

## Send Message Flow

```mermaid
sequenceDiagram
  participant Client
  participant Gateway as API Gateway
  participant Messaging as Messaging Service
  participant DB as Postgres
  participant Kafka
  participant WS as WebSocket Gateway
  participant Other as Subscribed Clients
  Client->>Gateway: POST /api/v1/messaging/messages
  Gateway->>Messaging: SendMessage gRPC
  Messaging->>DB: insert message and outbox row
  Messaging-->>Gateway: message result
  Gateway-->>Client: HTTP response
  Messaging->>Kafka: publish outbox event
  Kafka->>WS: message event
  WS-->>Other: realtime message_created
```

Source paths:

- `aloqa-backend/shared/api/api-gateway/v1/paths/messaging/send_message.yaml`
- `aloqa-backend/messaging-service/`
- `aloqa-backend/platform/migrations/20260610000001_messaging_outbox.*`
- `aloqa-backend/ws-gateway/`
- `aloqa-frontend/packages/core/src/realtime/events.ts`

Important correctness requirement: the HTTP success response and realtime event should converge on the same message identity and ordering semantics.

## WebSocket Connection Flow

```mermaid
sequenceDiagram
  participant Browser
  participant BFF as ws-ticket route
  participant WS as WebSocket Gateway
  Browser->>BFF: POST /api/realtime/ws-ticket
  BFF->>BFF: validate sealed session and refresh if needed
  BFF-->>Browser: short-lived token
  Browser->>WS: connect /ws/chat with token
  WS->>WS: authenticate
  WS-->>Browser: connected
```

Source paths:

- `aloqa-frontend/apps/web/app/api/realtime/ws-ticket/route.ts`
- `aloqa-frontend/packages/core/src/realtime/client.ts`
- `aloqa-backend/ws-gateway/`

Desktop and mobile may use direct token adapters rather than the web ticket route.

## Meeting Join Flow

```mermaid
sequenceDiagram
  participant Client
  participant Gateway as API Gateway
  participant Meeting as Realtime Service
  participant DB as Postgres
  participant Redis
  participant LiveKit
  participant WS as WebSocket Gateway
  Client->>Gateway: POST /api/v1/meeting/{id}/join
  Gateway->>Meeting: JoinMeeting gRPC
  Meeting->>DB: participant/join state
  Meeting->>Redis: room/participant state
  Meeting->>LiveKit: issue or validate media token
  Meeting-->>Gateway: join result
  Gateway-->>Client: meeting token/state
  Meeting->>WS: event via outbox/Kafka path
  WS-->>Client: room update
```

Source paths:

- `aloqa-backend/shared/api/api-gateway/v1/paths/meeting/join_meeting.yaml`
- `aloqa-backend/realtime-service/`
- `aloqa-backend/platform/migrations/20260522150000_add_video_meetings.*`
- `aloqa-frontend/docs/adr/0022-livekit-client-sdk.md`

Exact media token details should be verified in the realtime service implementation before changing client behavior.

## Waiting Room Flow

```mermaid
sequenceDiagram
  participant User
  participant Moderator
  participant Gateway
  participant Meeting as Realtime Service
  participant DB as Postgres
  participant WS as WebSocket Gateway
  User->>Gateway: join meeting
  Gateway->>Meeting: JoinMeeting
  Meeting->>DB: waiting participant/request state
  Meeting-->>User: waiting response
  Meeting->>WS: waiting event
  Moderator->>Gateway: admit or reject participant
  Gateway->>Meeting: Admit/Reject
  Meeting->>DB: update participant status
  Meeting->>WS: participant status event
```

Source paths:

- `aloqa-backend/shared/api/api-gateway/v1/paths/meeting/list_waiting_participants.yaml`
- `aloqa-backend/shared/api/api-gateway/v1/paths/meeting/admit_participant.yaml`
- `aloqa-backend/shared/api/api-gateway/v1/paths/meeting/reject_participant.yaml`
- `aloqa-backend/platform/migrations/20260624110000_waiting_room.*`

## Notification Flow

```mermaid
sequenceDiagram
  participant Producer as Service producing event
  participant Kafka
  participant Notification as Notification Service
  participant DB as Postgres
  participant Redis
  participant WS as WebSocket Gateway
  participant User
  Producer->>Kafka: publish event
  Kafka->>Notification: consume event
  Notification->>DB: store notification
  Notification->>Redis: publish user notification
  Redis->>WS: notif:user event
  WS-->>User: realtime notification
```

Source paths:

- `aloqa-backend/notification-service/`
- `aloqa-backend/ws-gateway/`
- `aloqa-backend/shared/api/api-gateway/v1/paths/notifications/`

## Search Flow

```mermaid
sequenceDiagram
  participant User
  participant Gateway
  participant Search as Search Service
  participant OpenSearch
  User->>Gateway: GET /api/v1/search
  Gateway->>Search: Search gRPC
  Search->>OpenSearch: query
  OpenSearch-->>Search: hits
  Search-->>Gateway: results
  Gateway-->>User: response
```

Source paths:

- `aloqa-backend/shared/api/api-gateway/v1/paths/search.yaml`
- `aloqa-backend/search-service/`

Indexing is likely asynchronous through Kafka consumers. Search should be treated as eventually consistent unless tests prove otherwise.

## Role Assignment Flow

```mermaid
sequenceDiagram
  participant Admin
  participant Gateway
  participant Org as Org Service
  participant DB as Postgres
  Admin->>Gateway: POST /api/v1/companies/roles/assign
  Gateway->>Org: assign role
  Org->>Org: check ABAC/permissions
  Org->>DB: update user role
  Org-->>Gateway: assignment result
  Gateway-->>Admin: response
```

Source paths:

- `aloqa-backend/shared/api/api-gateway/v1/paths/assign_company_role.yaml`
- `aloqa-backend/org-service/internal/core/abac/`
- `aloqa-backend/platform/migrations/20260604000001_custom_roles_v2.*`

## Request Flow Risks

### Replay Safety

Normal BFF requests can be retried after refresh; upload streams cannot. That split is explicit in web routes. Source paths:

- `aloqa-frontend/apps/web/app/api/[...path]/route.ts`
- `aloqa-frontend/apps/web/app/api/upload/[...path]/route.ts`

### Eventual Consistency

Message search, notification delivery, and realtime fanout can lag behind database writes. Product UX should account for that.

### Multi-Path Auth

Web, desktop, and mobile may not use identical token flows. A bug can affect one platform only.

### Deployment Routing

The web BFF request flow only works if edge routing is configured correctly.

## Flow Assessment

The request flows are coherent and match the architecture of a collaboration product. The main risk is that many workflows are cross-service and asynchronous, so integration tests should be prioritized over isolated unit tests.
