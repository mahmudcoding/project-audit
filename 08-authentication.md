# Authentication and Authorization Audit

## Authentication Summary

Aloqa authentication spans backend services, the API gateway, the web BFF, and platform clients.

Backend auth source paths:

- `aloqa-backend/auth-service/`
- `aloqa-backend/shared/proto/auth/`
- `aloqa-backend/platform/pkg/utils/auth.go`
- `aloqa-backend/platform/migrations/`
- `aloqa-backend/shared/api/api-gateway/v1/paths/`

Frontend auth source paths:

- `aloqa-frontend/apps/web/src/lib/auth/sessionCookie.ts`
- `aloqa-frontend/apps/web/src/lib/auth/sessionRefresh.ts`
- `aloqa-frontend/apps/web/src/lib/auth/withCsrf.ts`
- `aloqa-frontend/apps/web/app/api/[...path]/route.ts`
- `aloqa-frontend/apps/web/app/api/auth/refresh/route.ts`
- `aloqa-frontend/apps/web/app/api/realtime/ws-ticket/route.ts`

## Backend Auth Capabilities

Backend auth supports:

- user registration
- login
- refresh token
- logout current session
- logout all sessions
- session listing
- email verification
- resend verification code
- forgot password
- reset password
- Google login
- password change
- 2FA enable and confirmation
- 2FA disable and confirmation
- 2FA login verification and resend
- magic link request and verification
- settings and profile updates
- privacy updates
- avatar update

Source paths:

- `aloqa-backend/shared/proto/auth/`
- `aloqa-backend/shared/api/api-gateway/v1/paths/security/`
- `aloqa-backend/auth-service/`

## Token and Session Model

The backend uses JWT helpers in the platform module and stores session state in Postgres and Redis-adjacent infrastructure. Source paths:

- `aloqa-backend/platform/pkg/utils/auth.go`
- `aloqa-backend/auth-service/`
- `aloqa-backend/platform/migrations/20260504074928_init.*`

The API gateway sets access and refresh cookies for some auth flows. Source path: `aloqa-backend/api-gateway/`.

The web BFF seals backend token material into an app-owned HttpOnly cookie named `aloqa_bff_session`. Source path: `aloqa-frontend/apps/web/src/lib/auth/sessionCookie.ts`.

This creates two related but different session layers:

- backend session: the canonical server-side auth state
- web BFF session: sealed frontend-server cookie containing backend token material for browser requests

## Web BFF Security Model

For web, browser JavaScript should not manage backend refresh tokens directly. Instead:

1. Browser authenticates through web routes.
2. The web server stores backend token material in a sealed HttpOnly cookie.
3. Browser calls same-origin `/api/*`.
4. The BFF forwards to the backend gateway with backend cookies server-side.
5. If the access token is stale and the request is replayable, the BFF refreshes and retries.

Source paths:

- `aloqa-frontend/docs/adr/0037-web-bff-backend-session.md`
- `aloqa-frontend/apps/web/app/api/[...path]/route.ts`
- `aloqa-frontend/apps/web/src/lib/auth/sessionCookie.ts`
- `aloqa-frontend/apps/web/src/lib/auth/sessionRefresh.ts`

This is a sound browser security model if routing is correct.

## CSRF

The web BFF uses CSRF checks for mutating methods. Source path: `aloqa-frontend/apps/web/src/lib/auth/withCsrf.ts`.

This is necessary because the web model relies on cookies. Same-origin and CSRF behavior must be tested behind the actual production edge.

## Refresh Behavior

The BFF has server-side refresh behavior:

- transparent refresh for replayable API requests
- separate upload route because streaming uploads cannot safely be replayed
- explicit refresh route
- single-flight refresh logic keyed by session ID inside the process

Source paths:

- `aloqa-frontend/apps/web/app/api/[...path]/route.ts`
- `aloqa-frontend/apps/web/app/api/upload/[...path]/route.ts`
- `aloqa-frontend/apps/web/app/api/auth/refresh/route.ts`
- `aloqa-frontend/apps/web/src/lib/auth/sessionRefresh.ts`

Risk: single-flight behavior is in-process. If production runs multiple web replicas, duplicate refresh races can still happen unless the backend refresh design tolerates them or a distributed lock/session design is added.

## WebSocket Auth

The web app has a WebSocket ticket route. Source path: `aloqa-frontend/apps/web/app/api/realtime/ws-ticket/route.ts`.

The route:

- requires a valid sealed session
- proactively refreshes when the access token is close to expiry
- returns a token for direct WebSocket connection

The WebSocket gateway authenticates clients and manages connection subscriptions. Source paths:

- `aloqa-backend/ws-gateway/`
- `aloqa-backend/ws-gateway/internal/ws/client.go`
- `aloqa-backend/ws-gateway/internal/ws/messages.go`

## Authorization Model

Authorization is broader than login. Aloqa has organization-level and workspace/channel-level permissions:

- company roles
- workspace roles or memberships
- channel membership
- custom roles
- role permissions
- user roles
- ABAC checks

Backend source paths:

- `aloqa-backend/org-service/internal/core/abac/`
- `aloqa-backend/platform/pkg/permissions/`
- `aloqa-backend/platform/migrations/20260520061940_add_abac_roles.*`
- `aloqa-backend/platform/migrations/20260604000001_custom_roles_v2.*`

Authorization risk is high because chat, files, meeting access, invites, roles, and storage administration all depend on correct permission checks.

## File Authorization

File access combines:

- file ownership
- workspace/company context
- file shares
- message or meeting attachments
- signed URL/content access behavior
- delete/revoke behavior
- storage quotas

Source paths:

- `aloqa-backend/file-service/`
- `aloqa-backend/platform/migrations/20260608000003_files.*`
- `aloqa-backend/platform/migrations/20260609000002_file_shares.*`

This area should be explicitly tested for privilege escalation.

## Meeting Authorization

Meeting authorization includes:

- meeting creator/owner
- channel/workspace access
- meeting admins
- admin permissions
- room settings
- participant permissions
- device permission requests
- waiting room admission
- moderation: mute, kick, ban
- breakout-room visibility and invites

Source paths:

- `aloqa-backend/realtime-service/`
- `aloqa-backend/shared/proto/meeting/`
- `aloqa-backend/platform/migrations/20260622072718_meeting_admins.*`
- `aloqa-backend/platform/migrations/20260622072719_meeting_admin_permissions.*`
- `aloqa-backend/platform/migrations/20260622074215_meeting_participant_permissions.*`

Meeting auth is more complex than normal chat auth and should have dedicated tests.

## Auth Risks

### Edge Routing Can Bypass BFF Assumptions

If browser `/api/v1/*` goes directly to `api-gateway`, the web BFF protections and sealed session model may be bypassed. Source paths:

- `aloqa-frontend/deploy/nginx.prod.conf`
- `aloqa-backend/deploy/prod/nginx/nginx.conf`

This is the highest auth architecture risk.

### Distributed Refresh

BFF refresh single-flight appears process-local. If web scales horizontally, refresh race behavior must be validated.

### Test Coverage

The backend has limited test coverage according to its own instructions. Auth and authorization need integration tests, not just unit tests.

### Cookie Details

The exact production behavior depends on domains, TLS, SameSite, Secure flags, reverse proxy headers, and edge routing. I cannot determine the live cookie behavior from the repo alone.

## Recommended Auth Tests

Add tests for:

- register -> verify email -> login
- login -> refresh -> retry request
- logout current session
- logout all sessions
- password change invalidates old credentials as expected
- forgot/reset password token expiry
- 2FA enable/disable/login
- magic link replay prevention
- web BFF CSRF rejection for mutating requests
- web BFF refresh under concurrent requests
- file access across workspace boundaries
- meeting admin and participant permission edge cases
- WebSocket connect with expired, invalid, and refreshed tokens

## Auth Assessment

The auth design has the right pieces: sessions, JWT, HttpOnly web session sealing, CSRF, 2FA, OAuth, magic links, and ABAC. The risk is integration correctness across gateway, BFF, cookies, permissions, and WebSocket authentication.
