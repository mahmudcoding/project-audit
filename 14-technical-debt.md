# 14. Technical Debt

## What is technical debt?

Technical debt is work the team still needs to do to make the product safer, easier, or cheaper to change.

Real-life analogy:

It is like building maintenance.

The building may still work today, but if you ignore wiring, plumbing, locks, or fire alarms, future repairs become more expensive and dangerous.

## Why PMs should care

Technical debt affects:

- delivery speed
- bug risk
- release safety
- engineering estimates
- customer trust
- incident risk

Technical debt is not always bad. Sometimes teams take debt to move faster. The important thing is to know which debt is dangerous.

## Debt priority scale

```text
P0 -> fix soon, high risk
P1 -> important, plan into roadmap
P2 -> cleanup or hardening, lower urgency
```

Quick terms used in this chapter:

- BFF means Backend for Frontend, the web app's helper server layer.
- OpenAPI means the written public HTTP API menu.
- CI means automated checks.
- Kafka means internal event delivery.
- Outbox means a database tray of events to send later.

## P0: Decide the production edge

What this means:

The "edge" is the public entrance to production traffic.

Analogy:

It is the front door of the building. Everyone must agree which door visitors use.

The issue:

The frontend deployment says browser API requests should go to the web BFF.

The backend deployment sends `/api/v1/` requests directly to the API Gateway.

Important files:

```text
aloqa-frontend/deploy/nginx.prod.conf
aloqa-backend/deploy/prod/nginx/nginx.conf
aloqa-frontend/docs/adr/0037-web-bff-backend-session.md
```

Why business should care:

If requests enter through the wrong door, web login and security behavior may not match the design.

Who depends on it:

- web frontend
- backend gateway
- auth service
- DevOps/deployment
- security

Modification cost:

Medium to high. It may be mostly configuration, but the impact is security-critical and needs careful smoke testing.

Recommended fix:

Pick one official production routing model and remove or clearly mark the other as inactive.

## P0: Add backend integration tests

What this means:

Integration tests check that multiple parts work together.

Analogy:

Testing a car engine alone is useful. But integration testing checks whether the engine, brakes, steering, and dashboard work together on the road.

The issue:

Backend guidance says tests are mostly absent except limited benchmark coverage.

Important file:

```text
aloqa-backend/AGENTS.md
```

Why business should care:

The backend owns the highest-risk product rules: auth, permissions, files, messaging, meetings, and realtime.

Who depends on it:

- all frontend platforms
- QA
- support
- customers

Recommended first tests:

```text
login and refresh
workspace/channel creation
permission denial
send message and realtime event
file upload and share revoke
meeting join and waiting room
```

Modification cost:

High to start, but it reduces future release risk.

## P0: Add contract drift checks

What this means:

Contract drift happens when two parts of the system disagree about how they talk.

Example:

```text
Frontend calls /api/v1/example
Backend only supports /api/v1/examples
```

The issue:

Backend OpenAPI, backend protobuf, and frontend route helpers are separate files.

Important files:

```text
aloqa-backend/shared/api/api-gateway/v1/
aloqa-backend/shared/proto/
aloqa-frontend/packages/core/src/api/routes.ts
```

Why business should care:

Screens can break even when both frontend and backend code "look correct" separately.

Recommended fix:

Add CI checks that compare frontend routes with backend OpenAPI and fail if they drift.

Modification cost:

Medium.

## P1: Version realtime events

What this means:

Realtime events are messages sent over live connections.

Example:

```text
message_created
reaction_updated
meeting_participant_joined
```

The issue:

Backend and frontend must agree on event names and fields.

Important files:

```text
aloqa-backend/ws-gateway/
aloqa-frontend/packages/core/src/realtime/events.ts
```

Why business should care:

If events drift, chat or meetings can break without normal page errors.

Recommended fix:

Create versioned event schemas and tests that replay backend events through frontend parsers.

Modification cost:

Medium to high.

## P1: Add outbox and Kafka observability

What this means:

Observability means the team can see what the system is doing.

The outbox and Kafka move events behind the scenes.

Analogy:

If Kafka is the mailroom, observability is the tracking screen showing delayed or lost mail.

The issue:

This audit found outbox tables, but cannot prove production dashboards or alerts exist.

Important files:

```text
aloqa-backend/platform/migrations/20260610000001_messaging_outbox.*
aloqa-backend/platform/migrations/20260612000001_channels_outbox.*
aloqa-backend/platform/migrations/20260622085254_meeting_outbox.*
```

Why business should care:

Messages may save but not deliver live. Notifications may lag. Search may become stale.

Recommended metrics:

- oldest unsent event
- retry count
- failed event count
- Kafka lag
- dead-letter count

Modification cost:

Medium.

## P1: Clarify database table ownership

What this means:

Each table should have an owning service.

Analogy:

Every filing cabinet should have a department responsible for it.

The issue:

Many backend services use the same PostgreSQL database.

Important files:

```text
aloqa-backend/platform/migrations/
```

Why business should care:

If ownership is unclear, one feature can accidentally break another.

Recommended fix:

Create a table ownership document:

```text
table -> owner service -> allowed readers -> allowed writers
```

Modification cost:

Medium.

## P1: Complete frontend platform-first migration

What this means:

The frontend architecture wants shared logic, not shared UI.

The issue:

Some feature packages still expose platform-specific UI entrypoints.

Important files:

```text
aloqa-frontend/docs/adr/0023-platform-first-architecture.md
aloqa-frontend/packages/features/
```

Why business should care:

Web, desktop, and mobile users may need different UI. Over-sharing UI can slow platform-specific improvements.

Recommended fix:

Migrate feature packages gradually when those areas are touched.

Modification cost:

Medium, but should be done incrementally.

## P1: Harden file security tests

What this means:

Files require extra safety because users upload unknown content.

Important files:

```text
aloqa-backend/file-service/
aloqa-frontend/apps/web/app/api/upload/[...path]/route.ts
```

Test cases:

```text
unauthorized file access
revoked share
workspace boundary access
scan failure
quota enforcement
deleted file access
meeting attachment access
```

Modification cost:

Medium to high.

## P1: Add meeting state-machine tests

What this means:

A state machine is a set of allowed states and transitions.

Plain English:

A meeting participant may be waiting, admitted, rejected, muted, banned, or moved to a breakout room. The system must handle those changes in the right order.

Important files:

```text
aloqa-backend/realtime-service/
aloqa-backend/shared/proto/meeting/
aloqa-backend/platform/migrations/
aloqa-frontend/packages/features/calls/
```

Why business should care:

Meeting bugs are highly visible during live calls.

Modification cost:

High, but important.

## P2: Clean up deployment file linting

The backend production compose includes a duplicate `POSTGRES_HOST: postgres` entry under `messaging-service`.

Important file:

```text
aloqa-backend/deploy/prod/docker-compose.yml
```

This is likely not the biggest issue, but deployment files should be linted.

Modification cost:

Low to medium.

## P2: Reduce documentation drift

Some repo guidance appears older than current implementation.

Why business should care:

Old documentation makes onboarding slower and causes wrong assumptions.

Modification cost:

Low to medium.

## What you should remember

- Technical debt is future maintenance risk.
- The highest debt is production routing clarity.
- Backend integration tests are urgently needed.
- Contract drift checks would prevent frontend/backend mismatch.
- Realtime events need clearer versioning.
- Outbox and Kafka need visible monitoring.
- File and meeting areas deserve extra safety tests.
- Not all debt is equal; prioritize by business risk.
