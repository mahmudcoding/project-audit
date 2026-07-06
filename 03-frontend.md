# Frontend Audit

## Frontend Summary

The frontend is a platform-first TypeScript monorepo under `aloqa-frontend`. It targets:

- Web: `aloqa-frontend/apps/web`
- Desktop: `aloqa-frontend/apps/desktop`
- Mobile: `aloqa-frontend/apps/mobile`

The frontend strategy is not "one UI everywhere." ADR-0023 defines the intended model as shared headless logic plus app-owned UI. Source path: `aloqa-frontend/docs/adr/0023-platform-first-architecture.md`.

This is the right direction for a product with browser, Electron, and mobile surfaces. The tradeoff is that shared business logic must be very well separated from rendering assumptions.

## Tooling and Package Model

The root package manifest declares:

- pnpm workspace package manager
- Turbo task orchestration
- TypeScript strict posture
- shared scripts for build, dev, lint, typecheck, test, verification, API codegen, smoke checks, mobile checks, and release workflows

Source path: `aloqa-frontend/package.json`.

The workspace is organized around apps and packages:

- `apps/*`: deployable applications
- `packages/*`: shared packages and feature packages
- `tooling/*`: supporting tools where present

The frontend AGENTS file documents a strict engineering posture: avoid `any`, avoid default exports, use adapters, preserve state boundaries, and keep CI budgets under control. Source path: `aloqa-frontend/AGENTS.md`.

## Web App

The web app is a Next.js app in `aloqa-frontend/apps/web`.

Important responsibilities:

- Browser UI for auth, workspace, chat, calls, files, search, settings, and admin areas.
- BFF route handlers under `apps/web/app/api/`.
- Session sealing, refresh, and CSRF handling under `apps/web/src/lib/auth/`.
- Same-origin API behavior for browser REST calls.
- Upload streaming route for large file uploads.

Key paths:

- `aloqa-frontend/apps/web/app/`
- `aloqa-frontend/apps/web/app/api/[...path]/route.ts`
- `aloqa-frontend/apps/web/app/api/upload/[...path]/route.ts`
- `aloqa-frontend/apps/web/app/api/realtime/ws-ticket/route.ts`
- `aloqa-frontend/apps/web/src/lib/auth/sessionCookie.ts`
- `aloqa-frontend/apps/web/src/lib/auth/sessionRefresh.ts`
- `aloqa-frontend/apps/web/src/lib/auth/withCsrf.ts`

The web app has an important security role. It is not just a static client; it owns backend-session sealing and controls how browser-originated REST calls reach the backend.

## Desktop App

The desktop app is an Electron application under `aloqa-frontend/apps/desktop`.

Important responsibilities:

- Native desktop shell.
- Renderer app with workspace/channel/settings/call screens.
- Desktop bridges for call surfaces and deep links.
- Electron-specific packaging and runtime concerns.

Key paths:

- `aloqa-frontend/apps/desktop/`
- `aloqa-frontend/apps/desktop/src/main/`
- `aloqa-frontend/apps/desktop/src/preload/`
- `aloqa-frontend/apps/desktop/src/renderer/`

The desktop app likely uses more direct API/client behavior than the web BFF, depending on its runtime token storage. I cannot fully determine every desktop auth storage path without a dedicated deep dive through renderer auth screens and adapters.

## Mobile App

The mobile app is an Expo/React Native app under `aloqa-frontend/apps/mobile`.

Important responsibilities:

- Mobile auth flows.
- Workspace tab navigation.
- Chat, channel, DM, thread, calls, files, settings, and admin surfaces.
- Native realtime/media integrations such as LiveKit React Native.
- Mobile storage and security concerns such as secure store, MMKV, SQLite, SSL pinning, notifications, and native sharing.

Key paths:

- `aloqa-frontend/apps/mobile/`
- `aloqa-frontend/apps/mobile/app/`
- `aloqa-frontend/apps/mobile/package.json`

The mobile app has the highest platform-specific integration burden because it combines navigation, secure storage, push/native notifications, LiveKit, and offline-adjacent storage.

## Shared Core

`packages/core` is the most important frontend package. It contains reusable, platform-neutral behavior:

- API client: `aloqa-frontend/packages/core/src/api/client.ts`
- API routes: `aloqa-frontend/packages/core/src/api/routes.ts`
- API endpoint builders: `aloqa-frontend/packages/core/src/api/endpoints.ts`
- WebSocket path helpers: `aloqa-frontend/packages/core/src/api/wsPath.ts`
- Realtime client: `aloqa-frontend/packages/core/src/realtime/client.ts`
- Realtime events: `aloqa-frontend/packages/core/src/realtime/events.ts`
- Realtime bridges: `aloqa-frontend/packages/core/src/realtime/bridges.ts`

The API client supports JSON and FormData bodies, authorization tokens, cookies, 401 refresh behavior, blob requests, and beacon-style requests. Source path: `aloqa-frontend/packages/core/src/api/client.ts`.

The route registry uses `/api/v1` as its backend API prefix. Source path: `aloqa-frontend/packages/core/src/api/routes.ts`.

## Feature Packages

Feature packages include:

- `@aloqa/admin`
- `@aloqa/calendar`
- `@aloqa/calls`
- `@aloqa/chat`
- `@aloqa/files`
- `@aloqa/search`
- `@aloqa/settings`

Source path: `aloqa-frontend/packages/features/`.

The intended direction is headless feature logic, but several packages still expose platform UI entrypoints such as `ui-web`, `ui-desktop`, or `ui-mobile`. That is a known architectural transition risk against ADR-0023. `@aloqa/files` appears closer to the intended headless model.

This should not be fixed by a broad rewrite. Migrate one feature package at a time when changing that feature anyway.

## Routing and Screens

The web app contains routes for:

- auth: login, signup, forgot password, reset password, magic link
- BFF API route handlers
- docs/openapi
- guest join
- workspace shell under `w/[wsId]`
- chat, DMs, channels, calls, files, calendar, search, settings, admin, directories

Source path: `aloqa-frontend/apps/web/app`.

The mobile app contains routes for:

- auth
- protected workspace tabs
- chat, calls, more
- channel, DM, thread
- calendar, files, settings/admin
- guest join and native sharing

Source path: `aloqa-frontend/apps/mobile/app`.

The desktop app contains renderer screens for workspace/channel/settings/call flows. Source path: `aloqa-frontend/apps/desktop/src/renderer`.

## Web BFF Behavior

The web BFF is one of the most important frontend design decisions.

The catch-all REST route:

- runs in Node runtime
- reads the sealed backend session
- forwards requests to the backend gateway
- attaches backend `access_token` as a Cookie header server-side
- buffers normal request bodies up to a replay limit for refresh retry
- handles stale-token refresh
- uses CSRF protection for mutating requests

Source path: `aloqa-frontend/apps/web/app/api/[...path]/route.ts`.

The upload route is separate because large streaming uploads are not safely replayable after refresh. Source path: `aloqa-frontend/apps/web/app/api/upload/[...path]/route.ts`.

The WebSocket ticket route issues a short-lived token for direct WebSocket connection. Source path: `aloqa-frontend/apps/web/app/api/realtime/ws-ticket/route.ts`.

## Frontend Realtime

The frontend realtime client is robust relative to the backend complexity:

- idempotent connect behavior
- reconnect with exponential backoff and jitter
- heartbeat and missed-pong tracking
- optional resume key and resume sequence
- injected token refresh
- support for backend envelope and flat-frame event formats

Source path: `aloqa-frontend/packages/core/src/realtime/client.ts`.

Event names are mapped in `aloqa-frontend/packages/core/src/realtime/events.ts`. That file is a critical integration contract with `aloqa-backend/ws-gateway` and backend event producers.

## Frontend Testing and CI

The frontend repo contains stronger test and CI structure than the backend:

- root scripts for lint, typecheck, test, and verify
- release PR workflow
- release CD workflow
- mobile CI workflow
- smoke checks for production behavior
- security/performance related scripts

Source paths:

- `aloqa-frontend/package.json`
- `aloqa-frontend/.github/workflows/`
- `aloqa-frontend/deploy/smoke-live.sh`
- `aloqa-frontend/docs/CICD.md`

The release policy says CI is primarily for PRs targeting `main`. Source path: `aloqa-frontend/docs/CICD.md`.

## Frontend Risks

### BFF Routing Risk

The web BFF security model only holds if browser `/api/*` traffic reaches Next.js. The frontend production nginx file supports this model; the backend production nginx file may not. Source paths:

- `aloqa-frontend/deploy/nginx.prod.conf`
- `aloqa-backend/deploy/prod/nginx/nginx.conf`

I cannot determine from the codebase which edge configuration is live.

### Shared Route Drift

The frontend route registry must stay aligned with backend OpenAPI. Source paths:

- `aloqa-frontend/packages/core/src/api/routes.ts`
- `aloqa-backend/shared/api/api-gateway/v1/api-gateway.openapi.yaml`

The registry includes comments for superseded routes and routes that appear to be future-facing or not clearly represented in backend OpenAPI. This should become a CI check, not a manual review task.

### Transitional UI Package Shape

Several feature packages still expose platform-specific UI entrypoints. That conflicts with the cleanest reading of ADR-0023. It is acceptable as migration debt, but it should be explicitly tracked.

### Multi-Platform Behavior Drift

Web, desktop, and mobile can easily diverge in auth storage, realtime reconnection behavior, media behavior, file handling, and error display. The architecture needs shared conformance tests around `packages/core`, plus app-specific smoke tests.

## Frontend Assessment

The frontend has a strong architectural foundation and serious platform ambition. The main risk is not poor structure; it is keeping that structure true as feature work continues. The web BFF and realtime client deserve special protection because they sit on security and product-critical boundaries.
