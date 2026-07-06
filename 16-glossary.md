# 16. Glossary

This glossary explains project terms in plain language.

## ABAC

ABAC means attribute-based access control.

Plain English:

The system checks facts before allowing an action.

Example facts:

- who is the user?
- what company are they in?
- what role do they have?
- what action are they trying to do?
- what resource are they touching?

Why Aloqa needs it:

Aloqa has companies, workspaces, channels, roles, files, and meetings. Access rules need more than "is admin or not."

Where it is used:

```text
aloqa-backend/org-service/internal/core/abac/
aloqa-backend/platform/pkg/permissions/
```

## API

An API is a doorway for one part of software to ask another part to do work.

Example:

```text
Frontend calls login API.
Backend checks login.
```

## API Gateway

The backend reception desk.

Frontend apps send many requests here. The gateway forwards work to the right backend service.

Where it is used:

```text
aloqa-backend/api-gateway/
```

## BFF

BFF means Backend for Frontend.

Plain English:

It is a helper server layer for one frontend app.

In Aloqa web, the BFF sits between the browser and backend.

Analogy:

The BFF is a receptionist for browser requests.

Where it is used:

```text
aloqa-frontend/apps/web/app/api/
```

## Backend

The trusted server-side part of Aloqa.

It stores data, checks permissions, and performs business rules.

## Breakout room

A smaller room inside a meeting.

Example:

A large meeting splits into three smaller group discussions.

## Channel

A conversation space inside a workspace.

Example:

`#engineering`, `#support`, or `#announcements`.

## CI

CI means continuous integration.

Plain English:

Automated checks that run before code is merged or released.

## Cookie

A small piece of data stored by the browser.

Aloqa web uses cookies for session behavior.

## CSRF

CSRF means cross-site request forgery.

Plain English:

A bad website tries to trick a logged-in browser into sending a request.

Aloqa uses CSRF checks for safer browser requests.

## Database

The long-term storage system.

In Aloqa, PostgreSQL is the main database.

## Direct message

A private conversation between users.

Often called DM.

## Docker Compose

A tool for running many services together.

Aloqa uses it for local and production-like service setup.

## Event

A record that something happened.

Example:

```text
message_created
meeting_joined
notification_created
```

## Frontend

The part users see and click.

In Aloqa, frontend includes web, desktop, and mobile apps.

## gRPC

A private communication method between backend services.

Analogy:

The internal phone system between backend departments.

Where it is defined:

```text
aloqa-backend/shared/proto/
```

## Kafka

An internal event delivery system.

Analogy:

The company mailroom.

One service sends an event. Another service picks it up.

## LiveKit

The audio/video meeting system.

Aloqa uses it so the team does not need to build video calls from scratch.

## Migration

An ordered database change.

Example:

```text
Create files table.
Add meeting settings column.
```

Where migrations live:

```text
aloqa-backend/platform/migrations/
```

## Microservice

A separate backend service with its own responsibility.

Analogy:

A department inside a company.

Example:

```text
Auth Service handles login.
Messaging Service handles messages.
```

## MinIO

Object storage for uploaded files.

Analogy:

The file storage room.

## OpenAPI

The written menu of public HTTP API routes.

Where it lives:

```text
aloqa-backend/shared/api/api-gateway/v1/
```

## OpenSearch

The search engine used to find content quickly.

## Outbox

A database table that stores events to send later.

Analogy:

An outgoing mail tray.

Why it matters:

It helps avoid losing events if a delivery system is temporarily unavailable.

## Permission

A rule that says whether a user can do something.

Analogy:

An access badge.

## PostgreSQL

A relational database used as Aloqa's main long-term storage.

## Redis

Fast short-term storage.

Analogy:

Sticky notes or a whiteboard.

Used for live state, presence, typing, and other temporary data.

## Realtime

Updates that arrive without refreshing.

Example:

New messages appear while a user is still looking at the screen.

## Role

A named set of permissions.

Example:

Admin, member, guest, or custom company role.

## Sealed cookie

A protected browser cookie.

Analogy:

A locked envelope carried by the browser, opened safely by the server.

## Session

A login state.

Analogy:

A visitor pass showing the user is currently signed in.

## WebSocket

A live connection that stays open.

Analogy:

A phone call that stays connected.

Why Aloqa needs it:

Messages, notifications, typing, and meeting updates need to arrive live.

## WebSocket Gateway

The backend service that manages live WebSocket connections.

Where it is used:

```text
aloqa-backend/ws-gateway/
```

## Workspace

A work area inside a company.

Workspaces contain channels, members, settings, storage, and invites.

## What you should remember

- API means a doorway between software parts.
- Gateway means reception desk.
- BFF means browser-facing helper server.
- Database means long-term storage.
- Redis means short-term memory.
- Kafka means internal delivery service.
- WebSocket means live connection.
- LiveKit means audio/video meeting system.
- Permission means access badge.
- Session means visitor pass.
