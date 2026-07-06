# 06. API

## What is an API?

An API is a set of doors the frontend can use to ask the backend for work.

Example:

```text
Frontend wants to log in
  -> calls login API

Frontend wants to send message
  -> calls send message API

Frontend wants to upload file
  -> calls upload file API
```

Real-life analogy:

The API is a service counter with labeled windows.

```text
Login window
Message window
File window
Meeting window
Search window
```

Each window accepts certain forms and returns certain answers.

## Why does Aloqa need APIs?

The frontend and backend are separate.

The frontend cannot directly open the database. It must ask the backend through approved doors.

This keeps the system safer because the backend can:

- check login
- check permission
- validate input
- store data correctly
- return only allowed information

Important term: API Gateway.

Plain English:

The API Gateway is the backend reception desk. Frontend apps send many public requests there, and it forwards the work to the right backend service.

Where it is used:

```text
aloqa-backend/api-gateway/
```

## Two API types in Aloqa

### Public HTTP API

This is the API used by frontend apps through the API Gateway.

Important term: HTTP.

Plain English:

HTTP is the normal request-and-response system used by web apps.

Example:

```text
POST /api/v1/auth/login/user
```

Where it is defined:

```text
aloqa-backend/shared/api/api-gateway/v1/api-gateway.openapi.yaml
aloqa-backend/shared/api/api-gateway/v1/paths/
```

### Internal gRPC API

Important term: gRPC.

Plain English:

gRPC is how backend services talk to each other. Think of it as the private phone system between departments.

Why Aloqa needs it:

The API Gateway receives public requests, then calls the right backend service through gRPC.

Where it is defined:

```text
aloqa-backend/shared/proto/
```

## What is OpenAPI?

OpenAPI is the written menu for the public HTTP API.

It explains:

- which paths exist
- which method is used, such as GET or POST
- what the request looks like
- what the response looks like

Why PMs should care:

If a feature needs a new screen, it may also need a new API door. If the door does not exist, frontend work alone is not enough.

## Example: login API journey

```text
User enters email and password
  -> frontend calls POST /api/v1/auth/login/user
  -> API Gateway receives it
  -> Auth Service checks the credentials
  -> response returns login success or failure
```

## Example: send message API journey

```text
User clicks Send
  -> frontend calls POST /api/v1/messaging/messages
  -> API Gateway forwards to Messaging Service
  -> message is saved
  -> response returns message data
  -> realtime event tells other users
```

## The API groups

The backend OpenAPI file defines 138 path items and 160 HTTP operations.

You do not need to memorize them. Think of them as product doors grouped by feature.

### Auth and security doors

Used for login, logout, sessions, password, email verification, 2FA, Google login, and magic links.

```text
POST /api/v1/auth/register/user
POST /api/v1/auth/login/user
POST /api/v1/auth/logout/user
POST /api/v1/auth/logout/all
POST /api/v1/auth/refresh/token
GET  /api/v1/auth/me
PUT  /api/v1/auth/me/settings
PATCH /api/v1/auth/me/profile
POST /api/v1/auth/verify/email
POST /api/v1/auth/verify/resend
POST /api/v1/auth/password/forgot
POST /api/v1/auth/password/reset
POST /api/v1/auth/google
POST /api/v1/auth/magic-link
GET  /api/v1/auth/magic-link/verify
GET  /api/v1/security/sessions
PATCH /api/v1/security/password/change
POST /api/v1/security/2fa/enable
POST /api/v1/security/2fa/enable/confirm
POST /api/v1/security/2fa/disable
POST /api/v1/security/2fa/disable/confirm
POST /api/v1/security/2fa/login/verify
POST /api/v1/security/2fa/login/resend
```

### User, company, workspace, and channel doors

Used for organization structure and access.

```text
POST /api/v1/users/me/avatar
GET  /api/v1/users/me/files
GET  /api/v1/users/me/companies
GET  /api/v1/users/me/workspaces
GET  /api/v1/users/me/channels
GET  /api/v1/users/me/channels/archived
GET  /api/v1/users/me/privacy/{company_id}
PUT  /api/v1/users/me/privacy/{company_id}/dm
PUT  /api/v1/users/me/privacy/{company_id}/invite
GET  /api/v1/users/{user_id}/roles
POST /api/v1/companies
GET/PATCH/DELETE /api/v1/companies/{company_id}
GET  /api/v1/companies/{company_id}/workspaces
GET  /api/v1/companies/{company_id}/members
GET  /api/v1/companies/{company_id}/roles
GET  /api/v1/companies/{company_id}/permissions/available
POST /api/v1/companies/kick
POST /api/v1/companies/roles
GET/PATCH/DELETE /api/v1/companies/roles/{role_id}
POST /api/v1/companies/roles/assign
POST /api/v1/workspaces
GET/PATCH /api/v1/workspaces/{workspace_id}
GET  /api/v1/workspaces/{workspace_id}/channels
GET  /api/v1/workspaces/{workspace_id}/public-channels
GET  /api/v1/workspaces/{workspace_id}/members
GET  /api/v1/workspaces/{workspace_id}/invites
GET  /api/v1/workspaces/{workspace_id}/storage
PATCH /api/v1/workspaces/{workspace_id}/quota
POST /api/v1/workspaces/kick
POST /api/v1/workspaces/invites
POST /api/v1/workspaces/invites/accept
POST /api/v1/channels
GET  /api/v1/channels/{channel_id}
GET  /api/v1/channels/{channel_id}/members
POST /api/v1/channels/{channel_id}/archive
POST /api/v1/channels/{channel_id}/unarchive
POST /api/v1/channels/{channel_id}/join
POST /api/v1/channels/members/add
POST /api/v1/channels/members/remove
POST /api/v1/channels/members/mute
```

### Messaging doors

Used for channel chat, direct messages, reactions, pins, read state, threads, and blocking.

```text
GET  /api/v1/messaging/me/direct
POST /api/v1/messaging/dm
DELETE /api/v1/messaging/dm/{channel_id}
POST /api/v1/messaging/messages
POST /api/v1/messaging/messages/forward
GET/DELETE /api/v1/messaging/channels/{channel_id}/messages
POST /api/v1/messaging/channels/{channel_id}/messages/restore
PATCH /api/v1/messaging/channels/{channel_id}/messages/{message_id}
POST /api/v1/messaging/channels/{channel_id}/messages/{message_id}/reactions
POST /api/v1/messaging/channels/{channel_id}/messages/{message_id}/pin
GET  /api/v1/messaging/channels/{channel_id}/messages/pinned
GET  /api/v1/messaging/channels/{channel_id}/members
POST /api/v1/messaging/channels/{channel_id}/read
GET  /api/v1/messaging/channels/{channel_id}/unread
GET  /api/v1/messaging/messages/{message_id}/thread
POST /api/v1/messaging/messages/{message_id}/reply
POST /api/v1/messaging/users/block
POST /api/v1/messaging/users/unblock
```

### File doors

Used for upload, download, shares, and storage information.

```text
GET  /api/v1/users/me/storage
POST /api/v1/files/upload
GET/DELETE /api/v1/files/{file_id}
GET  /api/v1/files/{file_id}/content
GET/POST/DELETE /api/v1/files/{file_id}/shares
```

### Meeting doors

Used for meetings, waiting rooms, meeting chat, admins, permissions, device requests, moderation, and breakout rooms.

```text
POST /api/v1/meeting
GET/PATCH /api/v1/meeting/{meeting_id}
POST /api/v1/meeting/{meeting_id}/end
POST /api/v1/meeting/{meeting_id}/join
GET  /api/v1/meeting/{meeting_id}/waiting
POST /api/v1/meeting/{meeting_id}/waiting/cancel
POST /api/v1/meeting/{meeting_id}/participants/{participant_id}/admit
POST /api/v1/meeting/{meeting_id}/participants/{participant_id}/reject
POST /api/v1/meeting/{meeting_id}/participants/admit-all
GET  /api/v1/meeting/channel/{channel_id}/active
GET/POST /api/v1/meeting/{meeting_id}/messages
GET/POST /api/v1/meeting/messages/{message_id}/thread
POST /api/v1/meeting/messages/{message_id}/reactions
GET  /api/v1/meeting/{meeting_id}/admins
POST/DELETE /api/v1/meeting/{meeting_id}/admins/{user_id}
GET/PATCH /api/v1/meeting/{meeting_id}/settings
GET/PUT /api/v1/meeting/{meeting_id}/participants/{user_id}/permissions
GET/POST /api/v1/meeting/{meeting_id}/permission-requests
POST /api/v1/meeting/{meeting_id}/permission-requests/{request_id}/approve
POST /api/v1/meeting/{meeting_id}/permission-requests/{request_id}/reject
POST /api/v1/meeting/{meeting_id}/participants/{user_id}/device-requests
POST /api/v1/meeting/{meeting_id}/device-requests/{request_id}/accept
POST /api/v1/meeting/{meeting_id}/device-requests/{request_id}/decline
GET/POST/DELETE /api/v1/meeting/{meeting_id}/pin
POST /api/v1/meeting/{meeting_id}/participants/{user_id}/mute
POST /api/v1/meeting/{meeting_id}/participants/{user_id}/kick
POST/DELETE /api/v1/meeting/{meeting_id}/participants/{user_id}/ban
GET  /api/v1/meeting/{meeting_id}/bans
GET/POST /api/v1/meeting/{meeting_id}/breakout-rooms
POST /api/v1/meeting/{meeting_id}/breakout-rooms/close-all
POST /api/v1/meeting/{meeting_id}/breakout-rooms/leave
POST /api/v1/meeting/{meeting_id}/breakout-rooms/move
POST /api/v1/meeting/{meeting_id}/breakout-rooms/return-request
POST /api/v1/meeting/breakout-rooms/{breakout_room_id}/close
POST /api/v1/meeting/breakout-rooms/{breakout_room_id}/join
GET  /api/v1/meeting/breakout-rooms/{breakout_room_id}/participants
POST /api/v1/meeting/breakout-rooms/{breakout_room_id}/invite
PATCH /api/v1/meeting/breakout-rooms/{breakout_room_id}/name
PATCH /api/v1/meeting/breakout-rooms/{breakout_room_id}
GET/POST /api/v1/meeting/breakout-rooms/{breakout_room_id}/chat
POST /api/v1/meeting/breakout-rooms/chat/{message_id}/reactions
GET  /api/v1/meeting/{meeting_id}/participants
POST /api/v1/meeting/breakout-rooms/{breakout_room_id}/join-request
POST /api/v1/meeting/breakout-join-requests/{request_id}/approve
POST /api/v1/meeting/breakout-join-requests/{request_id}/reject
POST /api/v1/meeting/breakout-rooms/{breakout_room_id}/join-request/cancel
GET  /api/v1/meeting/breakout-rooms/{breakout_room_id}/waiting
```

### Search and notification doors

```text
GET  /api/v1/search
POST /api/v1/admin/search/reindex
GET  /api/v1/notifications
POST /api/v1/notifications/read
POST/DELETE /api/v1/notifications/channels/{channel_id}/mute
```

## Frontend API files

The frontend keeps its own map of API paths:

```text
aloqa-frontend/packages/core/src/api/routes.ts
aloqa-frontend/packages/core/src/api/endpoints.ts
aloqa-frontend/packages/core/src/api/client.ts
```

Why PMs should care:

If backend and frontend route maps disagree, screens can call APIs that do not exist or use old paths.

## API change cost

| Change | Cost | Why |
|---|---:|---|
| Add optional response field | Medium | Backend and frontend both need updates |
| Add new endpoint | Medium to high | Contract, backend, frontend, tests |
| Rename endpoint | High | Breaks clients unless migration path exists |
| Change auth behavior | High | Security and all platforms |
| Change meeting endpoint | High | Many screens and services depend on it |

## Important files

```text
aloqa-backend/shared/api/api-gateway/v1/api-gateway.openapi.yaml
aloqa-backend/shared/api/api-gateway/v1/paths/
aloqa-backend/shared/proto/
aloqa-frontend/packages/core/src/api/routes.ts
aloqa-frontend/apps/web/app/api/[...path]/route.ts
```

## Unknowns from code alone

This documentation lists the API paths and purpose. It does not prove every response shape works in a running system.

I cannot determine runtime compatibility without generating clients and running contract tests.

## What you should remember

- An API is a set of doors from frontend to backend.
- OpenAPI is the public HTTP API menu.
- gRPC is the internal phone system between backend services.
- The API Gateway receives most public backend requests.
- The frontend keeps its own route map and must stay in sync.
- Aloqa has 160 HTTP operations in the backend OpenAPI surface.
- API changes often affect backend, frontend, tests, and documentation.
