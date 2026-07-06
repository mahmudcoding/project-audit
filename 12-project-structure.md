# 12. Project Structure

## What is project structure?

Project structure is the folder map.

It answers:

- Where is the frontend?
- Where is the backend?
- Where are API contracts?
- Where are migrations?
- Where are deployment files?

Real-life analogy:

It is the building directory near an elevator.

```text
Floor 1 -> lobby
Floor 2 -> engineering
Floor 3 -> finance
Warehouse -> storage
```

## Why PMs should care

You do not need to write code, but knowing the map helps you:

- ask the right team
- understand estimates
- follow bug reports
- know whether a feature is frontend-only or backend-heavy
- understand why one change touches many files

Quick terms used in this chapter:

- BFF means Backend for Frontend, the web app's helper server layer.
- OpenAPI means the written public HTTP API menu.
- gRPC means the private phone system between backend services.
- CI/CD means automated checking and delivery workflows.
- Kafka means internal event delivery.
- WebSocket means a live connection that stays open.
- MinIO means file object storage.
- ClamAV means malware scanning for uploaded files.
- Redis means short-term memory.
- LiveKit means the audio/video meeting system.

## Top-level map

```text
/Users/mahmud/Projects/aloqa/
  aloqa-frontend/
  aloqa-backend/
  docs/
    project-audit/
```

## Frontend map

```text
aloqa-frontend/
  apps/
    web/
    desktop/
    mobile/
  packages/
    core/
    features/
    ui-kit-*/
  docs/
    adr/
  deploy/
  .github/
```

What each part means:

| Folder | Plain meaning |
|---|---|
| `apps/web` | browser app |
| `apps/desktop` | installed desktop app |
| `apps/mobile` | phone app |
| `packages/core` | shared frontend logic |
| `packages/features` | shared feature areas |
| `packages/ui-kit-*` | UI building blocks |
| `docs/adr` | architecture decisions |
| `deploy` | deployment config |
| `.github` | automation workflows |

## Backend map

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
  deploy/
  .github/
```

What each part means:

| Folder | Plain meaning |
|---|---|
| `api-gateway` | backend reception desk |
| `auth-service` | login and sessions |
| `org-service` | companies, workspaces, channels, roles |
| `messaging-service` | chat and DMs |
| `file-service` | uploads, files, shares |
| `realtime-service` | meeting rules and room state |
| `notification-service` | email and in-app notifications |
| `search-service` | search |
| `ws-gateway` | live WebSocket updates |
| `platform` | shared backend tools and migrations |
| `shared` | API contracts |
| `deploy` | local and production deployment files |

## Where API contracts live

HTTP/OpenAPI contracts:

```text
aloqa-backend/shared/api/api-gateway/v1/
```

gRPC contracts:

```text
aloqa-backend/shared/proto/
```

Plain English:

These folders define the promises between parts of the system.

## Where database changes live

```text
aloqa-backend/platform/migrations/
```

These files define changes to database tables over time.

## Where deployment files live

Backend deployment:

```text
aloqa-backend/deploy/
aloqa-backend/deploy/prod/docker-compose.yml
aloqa-backend/deploy/prod/nginx/nginx.conf
```

Frontend deployment:

```text
aloqa-frontend/deploy/nginx.prod.conf
aloqa-frontend/deploy/smoke-live.sh
```

CI/CD:

```text
aloqa-backend/.github/workflows/
aloqa-frontend/.github/workflows/
```

## How to find the owner of a bug

If bug says "cannot log in":

```text
auth-service
api-gateway
web BFF
frontend auth screen
```

If bug says "message sent but teammate does not see it":

```text
messaging-service
messaging_outbox
Kafka
ws-gateway
frontend realtime client
```

If bug says "file upload fails":

```text
web upload route
api-gateway
file-service
ClamAV
MinIO
files database tables
```

If bug says "meeting join fails":

```text
realtime-service
LiveKit
Redis
ws-gateway
call UI
meeting tables
```

## Important caution about generated code

Generated code means code created automatically from contracts.

Backend instructions say not to manually read or edit generated code under:

```text
aloqa-backend/shared/pkg/
aloqa-backend/shared/bin/
```

Source of truth is:

```text
aloqa-backend/shared/api/
aloqa-backend/shared/proto/
```

Why PMs should care:

If engineers say "we need to regenerate contracts", this is what they mean.

## What you should remember

- The project has separate frontend and backend repos.
- Frontend apps live under `aloqa-frontend/apps`.
- Shared frontend logic lives under `aloqa-frontend/packages`.
- Backend services live as separate folders under `aloqa-backend`.
- API contracts live under `aloqa-backend/shared`.
- Database migrations live under `aloqa-backend/platform/migrations`.
- Deployment files exist in both frontend and backend repos.
- For bugs, start by identifying the user action, then follow the folder map.
