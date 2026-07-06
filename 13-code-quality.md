# 13. Code Quality

## What does code quality mean here?

Code quality means how safe and easy the system is to change.

It is not only about whether code looks clean.

For Aloqa, quality means:

- can engineers understand where a feature lives?
- can tests catch mistakes?
- do frontend and backend contracts match?
- can deployment be trusted?
- can bugs be found quickly?
- can a small change avoid breaking unrelated features?

## Why PMs should care

Quality affects planning.

Bad quality turns small work into risky work.

Good quality makes estimates more reliable.

Quick terms used in this chapter:

- Kafka means internal event delivery.
- WebSocket means a live connection that stays open.
- OpenAPI means the written public HTTP API menu.
- Protobuf means a written contract for backend service messages.
- ADR means Architecture Decision Record, a note explaining an architecture choice.
- BFF means Backend for Frontend, the web app's helper server layer.
- Redis means fast short-term memory.
- Outbox means a database tray of events to send later.

Example:

```text
Change button text
  -> likely low risk

Change message delivery flow
  -> backend, database, Kafka, WebSocket, frontend
  -> higher risk
```

## What looks strong

### Clear backend departments

Backend services are separated by responsibility.

```text
auth-service
org-service
messaging-service
file-service
realtime-service
notification-service
search-service
ws-gateway
```

Why this is good:

When a bug happens, engineers can usually narrow the first place to inspect.

### Written API contracts

OpenAPI and protobuf files define how systems talk.

Why this is good:

Frontend and backend can agree on expected paths and data shapes.

Important files:

```text
aloqa-backend/shared/api/
aloqa-backend/shared/proto/
```

### Frontend architecture decisions are documented

The frontend explains important choices in ADR files.

ADR means Architecture Decision Record.

Plain English:

An ADR is a note explaining why the team chose a design.

Important files:

```text
aloqa-frontend/docs/adr/
```

### Web BFF security design is thoughtful

The web app has a BFF layer for safer browser session handling.

Why this is good:

It avoids exposing sensitive login material directly to browser code when server-side handling is safer.

Important files:

```text
aloqa-frontend/apps/web/app/api/
aloqa-frontend/apps/web/src/lib/auth/
```

## What looks risky

### Backend tests appear too thin

The backend instructions say tests are mostly absent except limited benchmark coverage.

Important file:

```text
aloqa-backend/AGENTS.md
```

Why PMs should care:

The backend owns login, permissions, files, meetings, and message delivery. These areas need automated tests.

### Frontend and backend API maps can drift

The backend has OpenAPI routes.

The frontend has route helpers.

If they disagree, a screen may call the wrong backend path.

Important files:

```text
aloqa-backend/shared/api/api-gateway/v1/
aloqa-frontend/packages/core/src/api/routes.ts
```

### Realtime has many moving parts

Realtime involves:

```text
database
outbox rows
Kafka
WebSocket Gateway
Redis
frontend realtime client
```

Why this is risky:

The message may save correctly but not appear live.

### Deployment routing is unclear

Frontend and backend deployment files do not tell the same story for `/api/*`.

Important files:

```text
aloqa-frontend/deploy/nginx.prod.conf
aloqa-backend/deploy/prod/nginx/nginx.conf
```

Why this is risky:

The web BFF security model depends on browser API requests going to the right place.

## Quality by product area

| Area | Quality concern | PM risk |
|---|---|---|
| Auth | session, token, 2FA, BFF behavior | users blocked or security issue |
| Permissions | role and access checks | wrong users get access |
| Messaging | save plus live delivery | chat feels broken |
| Files | upload, scan, share, quota | security or data access bug |
| Meetings | many state transitions | live call failures |
| Search | indexing and query behavior | stale or missing results |
| Notifications | stored plus live delivery | users miss important updates |

## What tests should exist first

Recommended first backend tests:

```text
register -> login -> refresh -> logout
create company -> workspace -> channel
assign role -> deny unauthorized action
send message -> receive WebSocket event
upload file -> share -> revoke
join meeting -> waiting room -> admit
```

Recommended frontend tests:

```text
login page
BFF refresh behavior
chat send and receive
file upload screen
call join screen
mobile navigation
desktop call surface
```

## What PMs should ask during planning

- Is this covered by tests today?
- Does this change an API contract?
- Does this affect more than one platform?
- Does this affect permissions?
- Does this affect realtime events?
- Does this affect database migrations?
- Is production routing involved?
- What smoke test proves it works?

## Unknowns from code alone

I cannot determine from static code inspection whether production has complete dashboards and alerts for all services.

## What you should remember

- Code quality means safe and predictable change.
- Aloqa has strong architecture separation.
- Backend test coverage is the biggest quality concern.
- API drift between frontend and backend is a real risk.
- Realtime bugs can be partial and hard to see.
- Deployment routing must be clarified.
- PMs should ask about tests, contracts, platforms, permissions, and realtime before accepting estimates.
