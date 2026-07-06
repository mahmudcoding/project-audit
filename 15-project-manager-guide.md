# Project Manager Guide

## Plain-English Summary

Aloqa is a multi-platform collaboration product. It is similar in category to Slack plus calls/files/search/admin controls.

It has:

- a backend made of multiple Go services
- a web app with a security-focused BFF
- a desktop app
- a mobile app
- realtime chat and meeting events
- LiveKit-based video/audio calls
- PostgreSQL, Redis, Kafka, MinIO, OpenSearch, and other infrastructure

This is a complex product. The main management risk is not whether there is code. There is a lot of code. The risk is making sure releases are safe across many services and platforms.

## Major Product Areas

### Account and Security

Includes login, sessions, password reset, email verification, 2FA, Google login, magic links, profile, settings, and privacy.

Engineering owners likely touch:

- backend auth service
- API gateway
- web BFF
- platform-specific clients

### Organization Model

Includes companies, workspaces, channels, invites, members, roles, permissions, and admin flows.

This is the backbone of access control.

### Messaging

Includes channel chat, direct messages, threads, reactions, pins, edits, deletes, read state, and blocking.

This is one of the core daily-use features.

### Files

Includes upload, storage, shares, download/content access, quotas, thumbnails/processing, and scan behavior.

This area has security and storage-cost implications.

### Meetings and Calls

Includes meeting lifecycle, LiveKit join, waiting room, meeting chat, admins, room settings, device requests, moderation, breakout rooms, and participant state.

This is the most complex single feature area.

### Search

Includes full-text search and admin reindex.

Search is likely eventually consistent because it depends on OpenSearch and background indexing.

### Notifications

Includes email and in-app/web notifications, read state, and channel mute behavior.

Notifications cross product boundaries because many features generate them.

## How To Think About Readiness

For each feature, track readiness across these columns:

| Area | Meaning |
|---|---|
| Backend contract | OpenAPI/gRPC route exists |
| Backend implementation | service logic exists and persists correctly |
| Web UI | browser feature works |
| Desktop UI | desktop feature works |
| Mobile UI | mobile feature works |
| Realtime | live updates work where expected |
| Permissions | unauthorized users are blocked |
| Tests | automated tests cover happy path and key failures |
| Smoke | production-like smoke test exists |
| Observability | failures can be detected |

Do not mark a feature "done" just because one route or one screen exists.

## Highest Release Risks

### Edge Routing

The web app expects browser API traffic to go through the web BFF. One backend nginx file routes `/api/v1/` directly to the backend gateway, while the frontend nginx routes `/api/*` to the web app. This must be resolved before relying on web auth/security behavior.

### Backend Coverage

Backend test coverage appears thin compared with the number of services and workflows.

### Realtime Reliability

Chat and meetings use asynchronous events through outbox tables, Kafka, Redis, WebSocket gateway, and frontend clients. These flows can fail partially.

### Meeting Complexity

Meetings include LiveKit plus many product rules. Bugs here are likely to be user-visible and hard to diagnose.

### File Security

Uploads, shares, scans, and downloads need strong tests.

## Questions PMs Should Ask Before Shipping a Feature

- Which platforms support this feature: web, desktop, mobile?
- Does the backend route exist in OpenAPI?
- Does the service implementation exist?
- Is there a permission check?
- Does the feature generate realtime events?
- Does the frontend listen for those events?
- What happens after reconnect?
- What happens if the request is retried?
- What happens if the user loses permission mid-flow?
- What is the failure message shown to the user?
- Is there an automated test?
- Is there a production smoke check?
- Is there a dashboard or alert if this fails?

## Suggested Milestones

### Milestone 1: Release Safety Baseline

Goals:

- decide production edge routing
- add contract drift checks
- add backend migration check
- add smoke test for web BFF, gateway, WebSocket, and LiveKit routing

### Milestone 2: Core Collaboration Confidence

Goals:

- integration tests for auth, workspace/channel, messaging, file upload
- WebSocket message fanout test
- file permission tests
- role/permission matrix tests

### Milestone 3: Meeting Hardening

Goals:

- meeting join/waiting/admit tests
- LiveKit token tests
- meeting admin permission tests
- device request tests
- breakout room tests
- meeting realtime event tests

### Milestone 4: Platform Parity

Goals:

- feature readiness matrix for web/desktop/mobile
- mobile-specific auth/reconnect tests
- desktop-specific call/deep-link tests
- shared core conformance tests

### Milestone 5: Operational Readiness

Goals:

- outbox/Kafka dashboards
- WebSocket connection metrics
- LiveKit join metrics
- file scan metrics
- search indexing lag metrics
- incident runbooks

## What Is Strong Already

- The backend has clear service boundaries.
- The frontend has clear architecture decisions.
- The API surface is documented in OpenAPI/protobuf.
- The database schema covers the real product domain.
- There are production deployment artifacts and CI/CD docs.

## What Needs Management Attention

- Prevent shipping features without backend tests.
- Prevent platform parity assumptions without checking all apps.
- Keep architecture docs current.
- Make one team or owner accountable for production edge routing.
- Treat meetings and files as high-risk product areas.
- Track contract drift as a release blocker.

## PM Assessment

Aloqa should be managed as a real multi-platform SaaS product, not as a simple web app. The codebase has enough architecture to support that, but the release process needs stronger gates before the product surface grows further.
