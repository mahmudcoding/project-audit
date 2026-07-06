# Feature Audit

## Feature Map

Aloqa implements a broad collaboration feature set:

- account and security
- companies and workspaces
- channels and membership
- custom roles and permissions
- direct messages
- channel messaging
- message threads, reactions, pins, read state, and forwarding
- files and file sharing
- notifications
- full-text search
- meetings and calls
- waiting rooms
- meeting chat
- meeting admins and permissions
- device requests
- breakout rooms
- realtime WebSocket updates
- web, desktop, and mobile client surfaces

The features are not evenly mature. Core backend contracts exist for many areas, but some frontend route helpers and app screens appear ahead of guaranteed backend/runtime validation.

## Account and Security

Capabilities:

- register user
- login
- logout and logout all
- refresh token
- current user lookup
- profile and settings update
- email verification and resend
- forgot/reset password
- Google login
- magic link login
- session listing
- password change
- 2FA enable/disable/login verification

Backend sources:

- `aloqa-backend/auth-service/`
- `aloqa-backend/shared/proto/auth/`
- `aloqa-backend/shared/api/api-gateway/v1/paths/`

Frontend sources:

- `aloqa-frontend/apps/web/app/`
- `aloqa-frontend/packages/core/src/api/routes.ts`
- `aloqa-frontend/apps/web/src/lib/auth/`

Product note: auth is a product-critical area because it affects every platform and every backend request.

## Companies, Workspaces, and Channels

Capabilities:

- company create/read/update/delete
- company members
- workspace create/read/update
- workspace members
- workspace invites and invite accept
- workspace storage/quota
- channel create/read/list
- public channels
- archive/unarchive
- join public channel
- channel add/remove/mute member
- user company/workspace/channel lists

Backend sources:

- `aloqa-backend/org-service/`
- `aloqa-backend/shared/proto/org/`
- `aloqa-backend/platform/migrations/`

Frontend sources:

- `aloqa-frontend/apps/web/app/w/[wsId]/`
- `aloqa-frontend/packages/core/src/api/routes.ts`
- `aloqa-frontend/packages/features/admin/`
- `aloqa-frontend/packages/features/settings/`

This is the structural foundation for all collaboration features.

## Roles, Permissions, and ABAC

Capabilities:

- company roles
- role permissions
- role assignment
- available permissions list
- company/workspace kick flows
- user role list
- ABAC-style permission checks inside backend services

Backend sources:

- `aloqa-backend/org-service/internal/core/abac/`
- `aloqa-backend/platform/pkg/permissions/`
- `aloqa-backend/platform/migrations/20260520061940_add_abac_roles.*`
- `aloqa-backend/platform/migrations/20260604000001_custom_roles_v2.*`

PM note: this is the area where product role naming must stay aligned with engineering permission strings. A mismatch here creates "UI says allowed, backend denies" bugs.

## Messaging and Direct Messages

Capabilities:

- send channel message
- list channel messages
- delete and restore messages
- edit message
- reactions
- pins and pinned lists
- read/unread
- channel members list
- forward message
- reply/thread
- create/list/delete DM
- block/unblock user

Backend sources:

- `aloqa-backend/messaging-service/`
- `aloqa-backend/shared/proto/messaging/`
- `aloqa-backend/platform/migrations/`

Frontend sources:

- `aloqa-frontend/packages/features/chat/`
- `aloqa-frontend/packages/core/src/realtime/events.ts`
- `aloqa-frontend/apps/web/app/w/[wsId]/`
- `aloqa-frontend/apps/mobile/app/`

Messaging also drives realtime complexity because a successful message write should usually fan out as an event to subscribed clients.

## Files

Capabilities:

- upload
- list user files
- get file metadata
- get file content
- delete file
- share/revoke/list shares
- batch sharing behavior in gRPC
- storage info and quotas
- file attachments in messages and meeting chat

Backend sources:

- `aloqa-backend/file-service/`
- `aloqa-backend/shared/proto/file/`
- `aloqa-backend/platform/migrations/20260608000003_files.*`

Frontend sources:

- `aloqa-frontend/packages/features/files/`
- `aloqa-frontend/apps/web/app/api/upload/[...path]/route.ts`
- `aloqa-frontend/packages/core/src/api/routes.ts`

Risk: files combine upload size, malware scanning, object storage, share permissions, signed URLs, thumbnails, and quotas. This area should have dedicated security and integration tests.

## Meetings and Calls

Capabilities:

- create meeting
- get/update/end meeting
- join meeting
- active meeting by channel
- waiting room
- admit/reject/cancel waiting
- meeting chat
- threads and reactions
- admins and admin permissions
- room settings
- participant permissions
- permission requests
- device requests
- pin target
- mute/kick/ban/unban
- participant list
- breakout rooms
- breakout room chat
- breakout room invites
- breakout join/waiting flows

Backend sources:

- `aloqa-backend/realtime-service/`
- `aloqa-backend/shared/proto/meeting/`
- `aloqa-backend/platform/migrations/`

Frontend sources:

- `aloqa-frontend/packages/features/calls/`
- `aloqa-frontend/packages/core/src/realtime/events.ts`
- `aloqa-frontend/docs/adr/0022-livekit-client-sdk.md`
- `aloqa-frontend/apps/desktop/`
- `aloqa-frontend/apps/mobile/`

This is the largest single feature domain. It should be managed as a product area with its own acceptance test suite.

## Notifications

Capabilities:

- email verification
- password reset email
- 2FA email
- magic link email
- web notifications
- list notifications
- mark notifications read
- channel mute/unmute notifications

Backend sources:

- `aloqa-backend/notification-service/`
- `aloqa-backend/shared/proto/notification/`
- `aloqa-backend/shared/api/api-gateway/v1/paths/notifications/`

Frontend sources:

- `aloqa-frontend/packages/core/src/realtime/events.ts`
- `aloqa-frontend/packages/core/src/realtime/bridges.ts`

Notifications depend on both stored notification records and realtime/user-specific delivery.

## Search

Capabilities:

- search
- admin reindex
- asynchronous indexing from events

Backend sources:

- `aloqa-backend/search-service/`
- `aloqa-backend/shared/proto/search/`
- `aloqa-backend/shared/api/api-gateway/v1/paths/search.yaml`
- `aloqa-backend/shared/api/api-gateway/v1/paths/admin_search_reindex.yaml`

Frontend sources:

- `aloqa-frontend/packages/features/search/`
- `aloqa-frontend/packages/core/src/api/routes.ts`

Search should be documented as eventually consistent unless product requirements say otherwise.

## Realtime Product Behavior

Realtime features include:

- chat message events
- message edit/delete/restore/reaction/pin events
- typing indicators
- notifications
- meeting room updates
- meeting participant updates
- meeting chat and reaction updates
- breakout room updates
- waiting room and permission request updates
- device state updates

Backend sources:

- `aloqa-backend/ws-gateway/`
- `aloqa-backend/messaging-service/`
- `aloqa-backend/realtime-service/`

Frontend source: `aloqa-frontend/packages/core/src/realtime/events.ts`.

## Feature Gaps and Unknowns

I cannot determine from static code inspection which features are fully shippable in production UX. The presence of routes, screens, and contracts does not prove end-to-end runtime completeness.

Areas that need product/QA verification:

- all meeting moderation edge cases
- file scan failure behavior
- quota enforcement UX
- role and permission matrix
- mobile parity with web
- desktop parity with web
- offline or reconnect behavior after long disconnection
- search freshness and reindex behavior
- notification delivery on each platform

## Feature Assessment

Aloqa has enough implemented surface to be managed as a mature product with explicit feature ownership. The next PM/engineering step should be feature readiness matrices: contract present, backend implementation present, web UI present, desktop UI present, mobile UI present, integration tests present, production smoke present.
