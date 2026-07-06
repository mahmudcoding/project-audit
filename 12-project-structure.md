# Project Structure

## Parent Directory

The local project root is:

```text
/Users/mahmud/Projects/aloqa
```

Important children:

```text
aloqa/
  aloqa-backend/
  aloqa-frontend/
  docs/
    project-audit/
```

`aloqa-backend` and `aloqa-frontend` are the primary source repositories. The audit docs are placed at the parent level to cover both without modifying either child repo source tree.

## Backend Root

Source path: `aloqa-backend/`

Important backend root files and directories:

```text
aloqa-backend/
  AGENTS.md
  README.md
  Taskfile.yml
  go.work
  api-gateway/
  auth-service/
  file-service/
  messaging-service/
  notification-service/
  org-service/
  platform/
  realtime-service/
  search-service/
  shared/
  ws-gateway/
  deploy/
  .github/
```

## Backend Services

Each backend service is a Go module. The consistent service shape is:

```text
service-name/
  cmd/
  internal/
  go.mod
```

The backend repository instructions describe a layered convention:

```text
cmd/main.go
internal/core/app
internal/core/config
internal/core/domain
internal/core/errors
internal/features/v1/<feature>/
  transport/
  service/
  repository/
  converter/
```

Source path: `aloqa-backend/AGENTS.md`.

Actual service directories:

```text
aloqa-backend/api-gateway/
aloqa-backend/auth-service/
aloqa-backend/file-service/
aloqa-backend/messaging-service/
aloqa-backend/notification-service/
aloqa-backend/org-service/
aloqa-backend/realtime-service/
aloqa-backend/search-service/
aloqa-backend/ws-gateway/
```

## Backend Platform Module

Source path: `aloqa-backend/platform/`

Important areas:

```text
platform/
  migrations/
  pkg/
```

`platform/migrations/` is the authoritative migration location for the shared Postgres schema.

`platform/pkg/` contains shared runtime helpers such as auth utilities, permissions, logging, middleware, database/Redis helpers, and other cross-service code.

## Backend Shared Contracts

Source path: `aloqa-backend/shared/`

Important areas:

```text
shared/
  api/
    api-gateway/
      v1/
        api-gateway.openapi.yaml
        paths/
  proto/
    auth/
    file/
    meeting/
    messaging/
    notification/
    org/
    presence/
    search/
```

Backend instructions say generated code under `shared/pkg` and tools under `shared/bin` should not be read or edited manually. Source path: `aloqa-backend/AGENTS.md`.

## Backend Deploy Structure

Source path: `aloqa-backend/deploy/`

Important areas:

```text
deploy/
  compose/
    core/
      docker-compose.yml
  prod/
    docker-compose.yml
    nginx/
      nginx.conf
```

The backend production compose describes infrastructure, backend services, frontend, and nginx in one deployment.

## Frontend Root

Source path: `aloqa-frontend/`

Important frontend root files and directories:

```text
aloqa-frontend/
  AGENTS.md
  package.json
  pnpm-lock.yaml
  apps/
  packages/
  docs/
  deploy/
  .github/
```

The root package file defines workspace scripts and package manager behavior. Source path: `aloqa-frontend/package.json`.

## Frontend Apps

Source path: `aloqa-frontend/apps/`

```text
apps/
  web/
  desktop/
  mobile/
```

`apps/web`:

- Next.js app
- browser UI
- BFF route handlers
- auth session sealing and refresh

`apps/desktop`:

- Electron app
- main/preload/renderer layers
- desktop shell and call surfaces

`apps/mobile`:

- Expo/React Native app
- mobile navigation
- mobile chat/calls/files/settings surfaces
- native storage and media integrations

## Frontend Packages

Source path: `aloqa-frontend/packages/`

Important categories:

```text
packages/
  core/
  features/
    admin/
    calendar/
    calls/
    chat/
    files/
    search/
    settings/
  ui-kit-*/
  eslint-config/
```

`packages/core` is the shared headless core. It includes API clients, route builders, realtime client logic, and shared domain behavior.

Feature packages are intended to be headless, although some still expose legacy platform UI entrypoints.

## Frontend Docs and ADRs

Source path: `aloqa-frontend/docs/`

Important docs:

- `docs/adr/0022-livekit-client-sdk.md`
- `docs/adr/0023-platform-first-architecture.md`
- `docs/adr/0037-web-bff-backend-session.md`
- `docs/adr/0038-single-aloqa-backend-target.md`
- `docs/CICD.md`
- `docs/infrastructure-architecture.md`

These docs are important because they explain architecture decisions that are not obvious from package names alone.

## Frontend Deploy Structure

Source path: `aloqa-frontend/deploy/`

Important files:

```text
deploy/
  nginx.prod.conf
  smoke-live.sh
```

The frontend production nginx file routes browser `/api/*` to the web app/BFF, `/ws/chat` to the WebSocket gateway, `/livekit/*` to LiveKit, and `/files/*` to the API gateway. Source path: `aloqa-frontend/deploy/nginx.prod.conf`.

## CI/CD Structure

Backend:

```text
aloqa-backend/.github/workflows/deploy.yml
```

Frontend:

```text
aloqa-frontend/.github/workflows/release-pr.yml
aloqa-frontend/.github/workflows/release-cd.yml
aloqa-frontend/.github/workflows/mobile-ci.yml
```

Source paths:

- `aloqa-backend/.github/workflows/`
- `aloqa-frontend/.github/workflows/`
- `aloqa-frontend/docs/CICD.md`

## Structural Assessment

The project structure is understandable and mostly well separated:

- backend services are explicit modules
- backend contracts are centralized
- frontend apps are separated by platform
- frontend shared logic has a clear home
- deployment files are present in both repos

The main structural weakness is cross-repo deployment ownership. Both backend and frontend contain production edge config, and those configs encode different assumptions about `/api/*`.
