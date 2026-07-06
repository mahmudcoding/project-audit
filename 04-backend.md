# 04. Backend

## What is the backend?

The backend is the trusted part of Aloqa that users do not see directly.

It:

- checks login
- stores messages
- checks permissions
- manages workspaces and channels
- stores file records
- sends notifications
- creates meeting state
- talks to the database
- talks to other infrastructure

Real-life analogy:

The frontend is the customer counter. The backend is the office behind the counter where employees check records, approve requests, and update the warehouse.

## Why does it exist?

The frontend should not be trusted to decide important things.

For example, the frontend can show a "Delete channel" button, but the backend must decide:

- Is the user logged in?
- Is the user allowed to delete this channel?
- Does the channel exist?
- What database records must change?
- Should other users be notified?

Quick terms used in this chapter:

- BFF means Backend for Frontend. It is the web app's helper server layer.
- Redis means fast short-term memory.
- Kafka means internal event delivery.
- WebSocket means a live connection that stays open.
- MinIO means file object storage.
- ClamAV means malware scanning for uploaded files.
- OpenSearch means the search engine.
- LiveKit means the audio/video meeting system.

## Backend as company departments

Aloqa backend is split into services.

A service is a separate backend department with one main responsibility.

```text
Auth Service          -> identity and sessions
Org Service           -> companies, workspaces, channels, roles
Messaging Service     -> chat and direct messages
File Service          -> uploads, shares, storage
Realtime Service      -> meeting rules and room state
Notification Service  -> email and web notifications
Search Service        -> search and reindex
API Gateway           -> reception desk for HTTP requests
WebSocket Gateway     -> live connection manager
Platform              -> shared backend tools
Shared                -> API contracts
```

Where the services live:

```text
aloqa-backend/
  api-gateway/
  auth-service/
  org-service/
  messaging-service/
  file-service/
  realtime-service/
  notification-service/
  search-service/
  ws-gateway/
  platform/
  shared/
```

## User journey: login

```text
User clicks Login
  -> frontend sends email and password
  -> API Gateway receives request
  -> Auth Service checks user
  -> database checks password/session records
  -> Auth Service creates tokens/session
  -> response goes back to frontend
```

Services affected:

- API Gateway
- Auth Service
- PostgreSQL
- Redis, depending on session/cache behavior
- Web BFF for browser users

Important files:

```text
aloqa-backend/api-gateway/
aloqa-backend/auth-service/
aloqa-backend/shared/proto/auth/
aloqa-backend/platform/pkg/utils/auth.go
```

## User journey: create a workspace channel

```text
User clicks Create Channel
  -> frontend sends channel name
  -> API Gateway receives request
  -> Org Service checks workspace permission
  -> Org Service creates channel
  -> database stores channel and membership
  -> channel appears in frontend
```

Services affected:

- API Gateway
- Org Service
- PostgreSQL
- possibly WebSocket or notification paths if users need updates

Important files:

```text
aloqa-backend/org-service/
aloqa-backend/shared/proto/org/
aloqa-backend/platform/pkg/permissions/
```

## User journey: send a message

```text
User clicks Send
  -> API Gateway receives message
  -> Messaging Service saves it
  -> database stores message
  -> outbox row is created
  -> Kafka delivers event
  -> WebSocket Gateway notifies other users
```

Important term: outbox row.

Plain English:

An outbox row is a saved "task note" in the database. It says, "This message was created; please tell other systems."

Why Aloqa needs it:

If the message saves but Kafka is temporarily unavailable, the note remains in the database and can be sent later.

Important files:

```text
aloqa-backend/messaging-service/
aloqa-backend/platform/migrations/20260610000001_messaging_outbox.*
aloqa-backend/ws-gateway/
```

## User journey: upload a file

```text
User chooses file
  -> frontend sends upload
  -> API Gateway receives upload
  -> File Service scans file
  -> MinIO stores file content
  -> database stores file metadata
  -> frontend shows uploaded file
```

Real-life analogy:

- MinIO is a storage room.
- Database metadata is the label on each box.
- ClamAV is a security guard checking files.

Important files:

```text
aloqa-backend/file-service/
aloqa-backend/shared/proto/file/
aloqa-backend/platform/migrations/20260608000003_files.*
```

## User journey: join a meeting

```text
User clicks Join
  -> API Gateway receives request
  -> Realtime Service checks meeting rules
  -> database stores participant state
  -> Redis stores short-term room state
  -> LiveKit handles audio/video
  -> WebSocket Gateway sends updates
```

Important files:

```text
aloqa-backend/realtime-service/
aloqa-backend/shared/proto/meeting/
aloqa-backend/platform/migrations/
```

## User journey: search

```text
User types search query
  -> API Gateway receives request
  -> Search Service asks OpenSearch
  -> OpenSearch returns matching records
  -> frontend shows results
```

Important files:

```text
aloqa-backend/search-service/
aloqa-backend/shared/proto/search/
```

## What is gRPC?

gRPC is a way for backend services to talk to each other.

Plain English:

If HTTP is how the public front desk receives requests, gRPC is the internal phone system between departments.

Why Aloqa needs it:

The API Gateway can ask the Auth Service, Org Service, Messaging Service, and other services to do work using clear internal contracts.

Where it is defined:

```text
aloqa-backend/shared/proto/
```

## What is OpenAPI?

OpenAPI is a written menu of the public HTTP API.

Plain English:

It lists the doors the frontend can knock on.

Example:

```text
POST /api/v1/auth/login/user
GET  /api/v1/auth/me
POST /api/v1/messaging/messages
```

Why Aloqa needs it:

It helps frontend and backend agree on request paths, request bodies, and responses.

Where it is defined:

```text
aloqa-backend/shared/api/api-gateway/v1/api-gateway.openapi.yaml
```

## Important backend files

```text
aloqa-backend/go.work
aloqa-backend/Taskfile.yml
aloqa-backend/README.md
aloqa-backend/AGENTS.md
aloqa-backend/shared/api/
aloqa-backend/shared/proto/
aloqa-backend/platform/migrations/
aloqa-backend/deploy/prod/docker-compose.yml
```

## What can break if backend changes?

Auth changes can break:

- login
- session refresh
- logout
- web BFF behavior
- desktop/mobile login

Org changes can break:

- workspace access
- channel membership
- roles
- permissions
- admin screens

Messaging changes can break:

- sending messages
- editing messages
- reactions
- threads
- live updates
- unread counts

File changes can break:

- upload
- download
- file shares
- quotas
- malware scanning

Meeting changes can break:

- join flow
- waiting room
- LiveKit token
- call controls
- breakout rooms
- live meeting updates

## Change cost guide

| Change area | Likely cost | Why |
|---|---:|---|
| Simple response text | Low | Usually one service and one screen |
| Add small API field | Medium | Backend, frontend, contracts, tests |
| Change login/session behavior | High | Security and every platform |
| Change permissions | High | Many features depend on access checks |
| Change meeting behavior | High | Database, LiveKit, WebSocket, clients |
| Change file access | High | Security and storage concerns |

## What you should remember

- The backend is the trusted work area behind the frontend.
- The API Gateway is the public backend reception desk.
- Backend services are separate departments.
- gRPC is the internal phone system between services.
- OpenAPI is the written menu of public HTTP routes.
- PostgreSQL stores long-term truth.
- Kafka helps services tell each other about events.
- Auth, permissions, files, and meetings are high-risk backend areas.
- Backend test coverage appears thin, so changes need care.
