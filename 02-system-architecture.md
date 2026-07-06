# 02. System Architecture

## What is system architecture?

System architecture is the map of how the whole product is put together.

It answers:

- Where does the user enter?
- Which app receives the request?
- Which backend service does the work?
- Where is data stored?
- How do live updates reach other users?

Quick translations before we start:

- Redis means fast short-term memory.
- Kafka means internal event delivery.
- WebSocket means a live connection that stays open.
- LiveKit means the audio/video meeting system.
- BFF means Backend for Frontend, a helper layer used by the web app.

Real-life analogy:

Aloqa is like an office building.

```text
Front lobby       -> frontend apps
Reception desk    -> API gateway
Departments       -> backend services
Warehouse         -> database
Short-term notes  -> Redis
Mailroom          -> Kafka
Phone line        -> WebSocket
Meeting rooms     -> LiveKit
```

## Why does this architecture exist?

Aloqa has many jobs to do:

- show screens
- protect login sessions
- store messages
- check permissions
- scan files
- run meetings
- send notifications
- search content
- update users live

Putting all of that into one large program would be hard to maintain.

So Aloqa separates work into parts.

## The simplest picture

```text
User
  |
  v
Frontend app
  |
  v
Gateway
  |
  v
Backend service
  |
  v
Database or infrastructure
```

Example:

```text
User sends message
  -> frontend app sends request
  -> API Gateway receives it
  -> Messaging Service saves it
  -> PostgreSQL stores it
  -> WebSocket Gateway sends live update
```

## The main parts

### Frontend apps

What this is:

The frontend is what users see and click.

Why it exists:

Users need screens, buttons, forms, menus, call controls, message lists, and file views.

Where it is used:

```text
aloqa-frontend/apps/web/
aloqa-frontend/apps/desktop/
aloqa-frontend/apps/mobile/
```

User journey:

```text
User opens app
  -> frontend loads screen
  -> user clicks something
  -> frontend sends request
  -> frontend shows result
```

### API Gateway

What this is:

The API Gateway is the main backend reception desk for normal requests.

Why it exists:

The frontend should not need to know every backend department. It sends requests to one main desk. That desk forwards the request to the right service.

Example:

```text
Frontend says: "Create channel"
  -> API Gateway receives it
  -> API Gateway asks Org Service
  -> Org Service creates channel
```

Where it is used:

```text
aloqa-backend/api-gateway/
```

### Backend services

What this is:

Backend services are separate departments.

Why Aloqa needs them:

Each department owns one major job.

```text
Auth Service          -> login and sessions
Org Service           -> companies, workspaces, channels, roles
Messaging Service     -> chat and direct messages
File Service          -> upload, storage, shares
Realtime Service      -> meeting logic
Notification Service  -> email and in-app notifications
Search Service        -> search
WebSocket Gateway     -> live updates
```

Where they are used:

```text
aloqa-backend/auth-service/
aloqa-backend/org-service/
aloqa-backend/messaging-service/
aloqa-backend/file-service/
aloqa-backend/realtime-service/
aloqa-backend/notification-service/
aloqa-backend/search-service/
aloqa-backend/ws-gateway/
```

### Database

What this is:

The database is the long-term warehouse.

Why it exists:

Messages, users, workspaces, files, roles, meetings, and sessions must survive after a server restarts.

Where it is used:

```text
aloqa-backend/platform/migrations/
```

### Redis

What this is:

Redis is fast short-term memory.

Why it exists:

Some data changes quickly and does not need to live forever in the same way as database records.

Examples:

- who is online
- room state during a meeting
- typing state
- short-lived notification delivery

Where it is used:

```text
aloqa-backend/realtime-service/internal/infrastructure/redis/
aloqa-backend/ws-gateway/
```

### Kafka

What this is:

Kafka is an internal delivery service.

Analogy:

Imagine a company mailroom. One department drops a note into the mailroom. Another department picks it up later.

Why it exists:

When a message is saved, other systems need to know:

- WebSocket should notify users
- Search may need to index it
- Notifications may need to alert someone

Kafka helps those systems hear about the change.

Where it is used:

```text
aloqa-backend/messaging-service/
aloqa-backend/realtime-service/
aloqa-backend/search-service/
aloqa-backend/ws-gateway/
```

### WebSocket Gateway

What this is:

WebSocket is a connection that stays open.

Analogy:

Normal HTTP is like sending a letter and waiting for a reply. WebSocket is like keeping a phone call open.

Why Aloqa needs it:

Users expect messages, notifications, typing, and meeting updates to appear live.

Where it is used:

```text
aloqa-backend/ws-gateway/
aloqa-frontend/packages/core/src/realtime/
```

### LiveKit

What this is:

LiveKit is the video and audio meeting system.

Why Aloqa needs it:

Building video/audio infrastructure from scratch is difficult. LiveKit handles the media part while Aloqa handles product rules like who can join, who is admin, and who is in the waiting room.

Where it is used:

```text
aloqa-backend/realtime-service/
aloqa-frontend/docs/adr/0022-livekit-client-sdk.md
```

## A full example: sending a message

```text
1. User types "Hello" in a channel.
2. Frontend sends the message.
3. API Gateway receives the request.
4. Messaging Service checks and saves the message.
5. PostgreSQL stores the message.
6. Messaging Service creates a delivery note.
7. Kafka carries the note.
8. WebSocket Gateway receives the note.
9. Other users see "Hello" appear live.
```

Diagram:

```text
User
  -> Frontend
  -> API Gateway
  -> Messaging Service
  -> PostgreSQL
  -> Kafka
  -> WebSocket Gateway
  -> Other users
```

## A full example: joining a meeting

```text
1. User clicks Join Meeting.
2. Frontend asks backend to join.
3. API Gateway forwards to Realtime Service.
4. Realtime Service checks meeting rules.
5. Database stores participant state.
6. Redis stores short-term room state.
7. LiveKit handles audio/video connection.
8. WebSocket Gateway sends room updates.
```

Diagram:

```text
User
  -> Frontend
  -> API Gateway
  -> Realtime Service
  -> Database and Redis
  -> LiveKit
  -> WebSocket Gateway
```

## Important implementation files

Do not start here when learning the project. Use these only after you understand the story above.

Backend:

```text
aloqa-backend/go.work
aloqa-backend/Taskfile.yml
aloqa-backend/shared/api/api-gateway/v1/api-gateway.openapi.yaml
aloqa-backend/shared/proto/
aloqa-backend/platform/migrations/
aloqa-backend/deploy/prod/docker-compose.yml
```

Frontend:

```text
aloqa-frontend/apps/
aloqa-frontend/packages/core/
aloqa-frontend/docs/adr/
aloqa-frontend/deploy/nginx.prod.conf
```

## Important risk: two deployment stories

The frontend deployment file says browser API traffic should go to the web BFF:

```text
aloqa-frontend/deploy/nginx.prod.conf
```

The backend deployment file sends `/api/v1/` traffic directly to the API Gateway:

```text
aloqa-backend/deploy/prod/nginx/nginx.conf
```

Why you should care:

If production uses the wrong path, the login/security design can behave differently from what the frontend expects.

I cannot determine from the codebase which routing file is active in production today.

## What you should remember

- Architecture is the map of how requests move through Aloqa.
- Frontend apps are the user's screens.
- API Gateway is the backend reception desk.
- Backend services are separate departments.
- PostgreSQL is the long-term warehouse.
- Redis is short-term memory.
- Kafka is the internal mailroom.
- WebSocket is a phone call that stays connected.
- LiveKit handles audio and video calls.
- The biggest architecture risk is unclear production routing.
