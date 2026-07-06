# Aloqa Project Audit Index

This audit covers the local project rooted at `/Users/mahmud/Projects/aloqa`.
That directory contains two primary source repositories:

- `aloqa-backend`: Go workspace with backend services, infrastructure, API contracts, database migrations, and production compose files.
- `aloqa-frontend`: platform-first frontend monorepo for web, desktop, and mobile clients.

The audit intentionally treats Aloqa as a system made of both repositories. A claim about "the project" usually depends on both halves.

## Documents

1. `01-executive-summary.md`: system-level status, strengths, risks, and recommended priorities.
2. `02-system-architecture.md`: runtime topology, repository ownership, service boundaries, and deployment shape.
3. `03-frontend.md`: frontend monorepo, app surfaces, BFF, feature packages, state, realtime, and testing.
4. `04-backend.md`: Go services, layered architecture, service responsibilities, workers, and operational behavior.
5. `05-database.md`: relational model, migrations, Redis, MinIO, OpenSearch, outbox tables, and data risks.
6. `06-api.md`: HTTP OpenAPI surface, gRPC service surface, BFF routing, and contract risks.
7. `07-features.md`: user-facing capabilities and how they map to frontend/backend implementation.
8. `08-authentication.md`: authentication, sessions, cookies, 2FA, OAuth, magic links, CSRF, and auth risks.
9. `09-realtime.md`: WebSocket gateway, LiveKit, Kafka outbox, Redis room state, events, and risks.
10. `10-request-flows.md`: end-to-end flows for login, BFF requests, chat, files, meetings, notifications, and search.
11. `11-technology-stack.md`: languages, frameworks, libraries, infrastructure, tooling, and CI/CD.
12. `12-project-structure.md`: directory maps for the backend, frontend, contracts, migrations, and deploy files.
13. `13-code-quality.md`: maintainability, testing posture, coupling, strengths, and quality risks.
14. `14-technical-debt.md`: prioritized debt list with impact and suggested remediation.
15. `15-project-manager-guide.md`: non-engineering guide to scope, milestones, risks, and useful questions.
16. `16-glossary.md`: shared vocabulary used across the codebase and this audit.

## Source Paths Used Heavily

- `aloqa-backend/AGENTS.md`
- `aloqa-backend/README.md`
- `aloqa-backend/go.work`
- `aloqa-backend/Taskfile.yml`
- `aloqa-backend/shared/api/api-gateway/v1/api-gateway.openapi.yaml`
- `aloqa-backend/shared/api/api-gateway/v1/paths/`
- `aloqa-backend/shared/proto/`
- `aloqa-backend/platform/migrations/`
- `aloqa-backend/deploy/compose/core/docker-compose.yml`
- `aloqa-backend/deploy/prod/docker-compose.yml`
- `aloqa-backend/deploy/prod/nginx/nginx.conf`
- `aloqa-frontend/AGENTS.md`
- `aloqa-frontend/package.json`
- `aloqa-frontend/apps/web/`
- `aloqa-frontend/apps/desktop/`
- `aloqa-frontend/apps/mobile/`
- `aloqa-frontend/packages/core/`
- `aloqa-frontend/packages/features/`
- `aloqa-frontend/docs/adr/`
- `aloqa-frontend/docs/CICD.md`
- `aloqa-frontend/docs/infrastructure-architecture.md`
- `aloqa-frontend/deploy/nginx.prod.conf`

## Audit Limits

I inspected source files, configuration, contracts, migrations, deployment files, and package manifests. I did not run the full application stack, run migrations, or execute all test suites. Where runtime-only behavior cannot be proven from the codebase, the relevant document says so explicitly.
