# Technology Stack

## Backend Stack

Backend language and runtime:

- Go
- Go workspace
- Go modules per service

Source paths:

- `aloqa-backend/go.work`
- `aloqa-backend/*/go.mod`

Backend service communication:

- HTTP through API gateway
- gRPC between gateway and services
- protobuf contracts under `shared/proto`
- OpenAPI contracts under `shared/api`

Source paths:

- `aloqa-backend/shared/proto/`
- `aloqa-backend/shared/api/api-gateway/v1/`

Backend libraries and infrastructure visible from module manifests and source:

- `pgx` for PostgreSQL
- Redis client libraries
- `kafka-go`
- MinIO Go SDK
- OpenSearch Go client
- LiveKit server SDK/protocol packages
- Gorilla WebSocket
- zap logging
- caarlos0/env and godotenv for configuration
- bcrypt/JWT helpers through platform dependencies
- squirrel SQL builder
- ogen for OpenAPI generation
- protobuf/gRPC toolchain
- Taskfile for workflow automation

Source paths:

- `aloqa-backend/*/go.mod`
- `aloqa-backend/Taskfile.yml`

## Backend Infrastructure

Infrastructure components:

- PostgreSQL
- Redis
- Kafka
- MinIO
- ClamAV
- OpenSearch
- LiveKit
- nginx

Source paths:

- `aloqa-backend/deploy/compose/core/docker-compose.yml`
- `aloqa-backend/deploy/prod/docker-compose.yml`
- `aloqa-backend/deploy/prod/nginx/nginx.conf`

## Frontend Stack

Frontend language/runtime:

- TypeScript
- React
- Next.js for web
- Electron for desktop
- Expo/React Native for mobile
- pnpm workspaces
- Turbo

Source paths:

- `aloqa-frontend/package.json`
- `aloqa-frontend/apps/web/package.json`
- `aloqa-frontend/apps/desktop/package.json`
- `aloqa-frontend/apps/mobile/package.json`

Frontend UI and state:

- Tailwind CSS
- Radix UI
- platform-specific UI kits
- TanStack Query
- Zustand
- Zod

Source paths:

- `aloqa-frontend/package.json`
- `aloqa-frontend/packages/`

Realtime and media:

- WebSocket client in `packages/core`
- LiveKit client SDK by ADR
- LiveKit React Native dependencies for mobile

Source paths:

- `aloqa-frontend/packages/core/src/realtime/`
- `aloqa-frontend/docs/adr/0022-livekit-client-sdk.md`
- `aloqa-frontend/apps/mobile/package.json`

Local/mobile persistence and platform features:

- Dexie in web dependencies
- op-sqlite in mobile dependencies
- MMKV
- Expo Secure Store
- native notifications
- SSL pinning
- CallKit/Telecom-style integrations where mobile dependencies support them

Source paths:

- `aloqa-frontend/apps/web/package.json`
- `aloqa-frontend/apps/mobile/package.json`

## Testing and Quality Tooling

Frontend:

- TypeScript typecheck
- ESLint
- Prettier
- Vitest
- Playwright
- Detox/mobile checks where configured
- CI workflows for release PR, release CD, and mobile CI

Source paths:

- `aloqa-frontend/package.json`
- `aloqa-frontend/.github/workflows/`

Backend:

- Taskfile tasks
- Go toolchain
- buf/protobuf generation
- ogen generation
- migrations
- limited tests noted by backend instructions

Source paths:

- `aloqa-backend/Taskfile.yml`
- `aloqa-backend/AGENTS.md`

## Deployment Stack

Backend deployment:

- Docker Compose
- nginx
- migration container
- SSH-based deployment workflow in GitHub Actions

Source paths:

- `aloqa-backend/deploy/prod/docker-compose.yml`
- `aloqa-backend/deploy/prod/nginx/nginx.conf`
- `aloqa-backend/.github/workflows/deploy.yml`

Frontend deployment:

- Dockerfile(s)
- nginx production config
- release PR workflow
- release CD workflow
- smoke script
- production docs referencing `https://airion-cargo.online/`

Source paths:

- `aloqa-frontend/docs/CICD.md`
- `aloqa-frontend/docs/infrastructure-architecture.md`
- `aloqa-frontend/deploy/nginx.prod.conf`
- `aloqa-frontend/deploy/smoke-live.sh`
- `aloqa-frontend/.github/workflows/`

## Version Policy Notes

The frontend AGENTS file says the stack was last verified around a date and lists modern versions such as Next.js 16, React 19.2, Tailwind v4, Electron 41, Expo SDK 55, React Native 0.83.6, TypeScript 6, pnpm 10, and Turbo 2. Source path: `aloqa-frontend/AGENTS.md`.

The backend AGENTS file lists Go 1.26.2 and infrastructure versions such as PostgreSQL 18.3 and Redis 8.6.3. Source path: `aloqa-backend/AGENTS.md`.

These are version-policy claims from repository instructions. They should be verified against actual `go.mod`, Docker images, package lockfiles, and CI runtime before release decisions.

## Stack Assessment

The stack is modern and appropriate for a real-time collaboration product. The main stack risk is not obsolete technology; it is integration complexity:

- many services
- many contracts
- multiple client platforms
- multiple deployment configs
- multiple state systems

The project needs strong verification and deployment ownership to match the ambition of its stack.
