# API Audit

## API Summary

Aloqa has two main API layers:

- Public/client HTTP API through the backend API gateway, specified by OpenAPI.
- Internal service-to-service gRPC APIs, specified by protobuf.

The frontend also adds a web BFF layer. Browser clients are intended to call same-origin Next.js route handlers under `/api/*`; those route handlers forward to the backend gateway. Desktop and mobile clients can use backend routes more directly depending on their auth adapters.

Source paths:

- `aloqa-backend/shared/api/api-gateway/v1/api-gateway.openapi.yaml`
- `aloqa-backend/shared/api/api-gateway/v1/paths/`
- `aloqa-backend/shared/proto/`
- `aloqa-frontend/apps/web/app/api/[...path]/route.ts`
- `aloqa-frontend/packages/core/src/api/routes.ts`
- `aloqa-frontend/packages/core/src/api/endpoints.ts`

## HTTP Surface

The OpenAPI root file defines 138 path items and 160 HTTP operations. The path items are split into small YAML files under `aloqa-backend/shared/api/api-gateway/v1/paths/`.

Source path: `aloqa-backend/shared/api/api-gateway/v1/api-gateway.openapi.yaml`.

### Endpoint Inventory

This list is extracted from the OpenAPI root and its referenced path files.

| Methods | Path | Source path item |
|---|---|---|
| POST | `/api/v1/auth/register/user` | `paths/register_user.yaml` |
| POST | `/api/v1/auth/login/user` | `paths/login_user.yaml` |
| POST | `/api/v1/auth/logout/user` | `paths/logout_user.yaml` |
| POST | `/api/v1/auth/logout/all` | `paths/logout_all.yaml` |
| POST | `/api/v1/auth/refresh/token` | `paths/refresh_token.yaml` |
| GET | `/api/v1/auth/me` | `paths/get_me.yaml` |
| PUT | `/api/v1/auth/me/settings` | `paths/update_user_settings.yaml` |
| PATCH | `/api/v1/auth/me/profile` | `paths/update_user_profile.yaml` |
| POST | `/api/v1/auth/verify/email` | `paths/verify_email.yaml` |
| POST | `/api/v1/auth/verify/resend` | `paths/resend_verification_code.yaml` |
| POST | `/api/v1/auth/password/forgot` | `paths/forgot_password.yaml` |
| POST | `/api/v1/auth/password/reset` | `paths/reset_password.yaml` |
| POST | `/api/v1/auth/google` | `paths/google_login.yaml` |
| POST | `/api/v1/auth/magic-link` | `paths/request_magic_link.yaml` |
| GET | `/api/v1/auth/magic-link/verify` | `paths/verify_magic_link.yaml` |
| GET | `/api/v1/security/sessions` | `paths/list_sessions.yaml` |
| PATCH | `/api/v1/security/password/change` | `paths/security/change_password.yaml` |
| POST | `/api/v1/security/2fa/enable` | `paths/security/two_fa_enable.yaml` |
| POST | `/api/v1/security/2fa/enable/confirm` | `paths/security/two_fa_enable_confirm.yaml` |
| POST | `/api/v1/security/2fa/disable` | `paths/security/two_fa_disable.yaml` |
| POST | `/api/v1/security/2fa/disable/confirm` | `paths/security/two_fa_disable_confirm.yaml` |
| POST | `/api/v1/security/2fa/login/verify` | `paths/security/two_fa_login_verify.yaml` |
| POST | `/api/v1/security/2fa/login/resend` | `paths/security/two_fa_login_resend.yaml` |
| POST | `/api/v1/users/me/avatar` | `paths/user/update_avatar.yaml` |
| GET | `/api/v1/users/me/files` | `paths/file/list_my_files.yaml` |
| GET | `/api/v1/users/me/companies` | `paths/list_my_companies.yaml` |
| GET | `/api/v1/users/me/workspaces` | `paths/list_my_workspaces.yaml` |
| GET | `/api/v1/users/me/channels` | `paths/list_my_channels.yaml` |
| GET | `/api/v1/users/me/channels/archived` | `paths/list_archived_channels.yaml` |
| GET | `/api/v1/users/me/privacy/{company_id}` | `paths/get_user_privacy.yaml` |
| PUT | `/api/v1/users/me/privacy/{company_id}/dm` | `paths/set_dm_privacy.yaml` |
| PUT | `/api/v1/users/me/privacy/{company_id}/invite` | `paths/set_invite_privacy.yaml` |
| GET | `/api/v1/users/{user_id}/roles` | `paths/list_user_company_roles.yaml` |
| POST | `/api/v1/companies` | `paths/create_company.yaml` |
| GET, PATCH, DELETE | `/api/v1/companies/{company_id}` | `paths/get_company.yaml` |
| GET | `/api/v1/companies/{company_id}/workspaces` | `paths/list_company_workspaces.yaml` |
| GET | `/api/v1/companies/{company_id}/members` | `paths/list_company_members.yaml` |
| GET | `/api/v1/companies/{company_id}/roles` | `paths/list_company_roles.yaml` |
| GET | `/api/v1/companies/{company_id}/permissions/available` | `paths/list_available_permissions.yaml` |
| POST | `/api/v1/companies/kick` | `paths/kick_company.yaml` |
| POST | `/api/v1/companies/roles` | `paths/create_company_role.yaml` |
| GET, PATCH, DELETE | `/api/v1/companies/roles/{role_id}` | `paths/company_role_by_id.yaml` |
| POST | `/api/v1/companies/roles/assign` | `paths/assign_company_role.yaml` |
| POST | `/api/v1/workspaces` | `paths/create_workspace.yaml` |
| GET, PATCH | `/api/v1/workspaces/{workspace_id}` | `paths/workspace_by_id.yaml` |
| GET | `/api/v1/workspaces/{workspace_id}/channels` | `paths/list_workspace_channels.yaml` |
| GET | `/api/v1/workspaces/{workspace_id}/public-channels` | `paths/list_public_channels.yaml` |
| GET | `/api/v1/workspaces/{workspace_id}/members` | `paths/list_workspace_members.yaml` |
| GET | `/api/v1/workspaces/{workspace_id}/invites` | `paths/list_workspace_invites.yaml` |
| GET | `/api/v1/workspaces/{workspace_id}/storage` | `paths/get_workspace_storage.yaml` |
| PATCH | `/api/v1/workspaces/{workspace_id}/quota` | `paths/update_workspace_quota.yaml` |
| POST | `/api/v1/workspaces/kick` | `paths/kick_workspace.yaml` |
| POST | `/api/v1/workspaces/invites` | `paths/create_workspace_invite.yaml` |
| POST | `/api/v1/workspaces/invites/accept` | `paths/accept_workspace_invite.yaml` |
| POST | `/api/v1/channels` | `paths/create_channel.yaml` |
| GET | `/api/v1/channels/{channel_id}` | `paths/get_channel.yaml` |
| GET | `/api/v1/channels/{channel_id}/members` | `paths/list_channel_members.yaml` |
| POST | `/api/v1/channels/{channel_id}/archive` | `paths/archive_channel.yaml` |
| POST | `/api/v1/channels/{channel_id}/unarchive` | `paths/unarchive_channel.yaml` |
| POST | `/api/v1/channels/{channel_id}/join` | `paths/join_public_channel.yaml` |
| POST | `/api/v1/channels/members/add` | `paths/add_channel_member.yaml` |
| POST | `/api/v1/channels/members/remove` | `paths/remove_channel_member.yaml` |
| POST | `/api/v1/channels/members/mute` | `paths/mute_channel_member.yaml` |
| GET | `/api/v1/messaging/me/direct` | `paths/list_my_direct.yaml` |
| POST | `/api/v1/messaging/dm` | `paths/messaging/create_dm.yaml` |
| DELETE | `/api/v1/messaging/dm/{channel_id}` | `paths/messaging/delete_dm.yaml` |
| POST | `/api/v1/messaging/messages` | `paths/messaging/send_message.yaml` |
| POST | `/api/v1/messaging/messages/forward` | `paths/messaging/forward_message.yaml` |
| DELETE, GET | `/api/v1/messaging/channels/{channel_id}/messages` | `paths/messaging/get_channel_messages.yaml` |
| POST | `/api/v1/messaging/channels/{channel_id}/messages/restore` | `paths/messaging/restore_messages.yaml` |
| PATCH | `/api/v1/messaging/channels/{channel_id}/messages/{message_id}` | `paths/messaging/edit_message.yaml` |
| POST | `/api/v1/messaging/channels/{channel_id}/messages/{message_id}/reactions` | `paths/messaging/message_reactions.yaml` |
| POST | `/api/v1/messaging/channels/{channel_id}/messages/{message_id}/pin` | `paths/messaging/pin_message.yaml` |
| GET | `/api/v1/messaging/channels/{channel_id}/messages/pinned` | `paths/messaging/get_pinned_messages.yaml` |
| GET | `/api/v1/messaging/channels/{channel_id}/members` | `paths/messaging/get_channel_members.yaml` |
| POST | `/api/v1/messaging/channels/{channel_id}/read` | `paths/messaging/mark_channel_read.yaml` |
| GET | `/api/v1/messaging/channels/{channel_id}/unread` | `paths/messaging/get_unread_count.yaml` |
| GET | `/api/v1/messaging/messages/{message_id}/thread` | `paths/messaging/get_thread_messages.yaml` |
| POST | `/api/v1/messaging/messages/{message_id}/reply` | `paths/messaging/reply_to_message.yaml` |
| POST | `/api/v1/messaging/users/block` | `paths/messaging/block_user.yaml` |
| POST | `/api/v1/messaging/users/unblock` | `paths/messaging/unblock_user.yaml` |
| GET | `/api/v1/users/me/storage` | `paths/file/get_user_storage.yaml` |
| POST | `/api/v1/files/upload` | `paths/file/upload_file.yaml` |
| GET, DELETE | `/api/v1/files/{file_id}` | `paths/file/get_file.yaml` |
| GET | `/api/v1/files/{file_id}/content` | `paths/file/get_file_content.yaml` |
| POST, DELETE, GET | `/api/v1/files/{file_id}/shares` | `paths/file/file_shares.yaml` |
| POST | `/api/v1/meeting` | `paths/meeting/create_meeting.yaml` |
| GET, PATCH | `/api/v1/meeting/{meeting_id}` | `paths/meeting/get_meeting.yaml` |
| POST | `/api/v1/meeting/{meeting_id}/end` | `paths/meeting/end_meeting.yaml` |
| POST | `/api/v1/meeting/{meeting_id}/join` | `paths/meeting/join_meeting.yaml` |
| GET | `/api/v1/meeting/{meeting_id}/waiting` | `paths/meeting/list_waiting_participants.yaml` |
| POST | `/api/v1/meeting/{meeting_id}/waiting/cancel` | `paths/meeting/cancel_waiting.yaml` |
| POST | `/api/v1/meeting/{meeting_id}/participants/{participant_id}/admit` | `paths/meeting/admit_participant.yaml` |
| POST | `/api/v1/meeting/{meeting_id}/participants/{participant_id}/reject` | `paths/meeting/reject_participant.yaml` |
| POST | `/api/v1/meeting/{meeting_id}/participants/admit-all` | `paths/meeting/admit_all_participants.yaml` |
| GET | `/api/v1/meeting/channel/{channel_id}/active` | `paths/meeting/get_active_meeting.yaml` |
| GET, POST | `/api/v1/meeting/{meeting_id}/messages` | `paths/meeting/meeting_chat_messages.yaml` |
| GET, POST | `/api/v1/meeting/messages/{message_id}/thread` | `paths/meeting/meeting_chat_thread.yaml` |
| POST | `/api/v1/meeting/messages/{message_id}/reactions` | `paths/meeting/meeting_chat_reactions.yaml` |
| GET | `/api/v1/meeting/{meeting_id}/admins` | `paths/meeting/meeting_admins.yaml` |
| POST, DELETE | `/api/v1/meeting/{meeting_id}/admins/{user_id}` | `paths/meeting/meeting_admin_item.yaml` |
| GET, PATCH | `/api/v1/meeting/{meeting_id}/settings` | `paths/meeting/meeting_room_settings.yaml` |
| GET, PUT | `/api/v1/meeting/{meeting_id}/participants/{user_id}/permissions` | `paths/meeting/meeting_participant_permissions.yaml` |
| GET, POST | `/api/v1/meeting/{meeting_id}/permission-requests` | `paths/meeting/meeting_permission_requests.yaml` |
| POST | `/api/v1/meeting/{meeting_id}/permission-requests/{request_id}/approve` | `paths/meeting/meeting_permission_request_approve.yaml` |
| POST | `/api/v1/meeting/{meeting_id}/permission-requests/{request_id}/reject` | `paths/meeting/meeting_permission_request_reject.yaml` |
| POST | `/api/v1/meeting/{meeting_id}/participants/{user_id}/device-requests` | `paths/meeting/meeting_device_requests.yaml` |
| POST | `/api/v1/meeting/{meeting_id}/device-requests/{request_id}/accept` | `paths/meeting/meeting_device_request_accept.yaml` |
| POST | `/api/v1/meeting/{meeting_id}/device-requests/{request_id}/decline` | `paths/meeting/meeting_device_request_decline.yaml` |
| GET, POST, DELETE | `/api/v1/meeting/{meeting_id}/pin` | `paths/meeting/meeting_pin.yaml` |
| POST | `/api/v1/meeting/{meeting_id}/participants/{user_id}/mute` | `paths/meeting/meeting_participant_mute.yaml` |
| POST | `/api/v1/meeting/{meeting_id}/participants/{user_id}/kick` | `paths/meeting/meeting_participant_kick.yaml` |
| POST, DELETE | `/api/v1/meeting/{meeting_id}/participants/{user_id}/ban` | `paths/meeting/meeting_participant_ban.yaml` |
| GET | `/api/v1/meeting/{meeting_id}/bans` | `paths/meeting/meeting_participant_bans.yaml` |
| GET, POST | `/api/v1/meeting/{meeting_id}/breakout-rooms` | `paths/meeting/create_breakout_room.yaml` |
| POST | `/api/v1/meeting/{meeting_id}/breakout-rooms/close-all` | `paths/meeting/close_all_breakout_rooms.yaml` |
| POST | `/api/v1/meeting/{meeting_id}/breakout-rooms/leave` | `paths/meeting/leave_breakout_room.yaml` |
| POST | `/api/v1/meeting/{meeting_id}/breakout-rooms/move` | `paths/meeting/move_participant.yaml` |
| POST | `/api/v1/meeting/{meeting_id}/breakout-rooms/return-request` | `paths/meeting/request_return_to_main.yaml` |
| POST | `/api/v1/meeting/breakout-rooms/{breakout_room_id}/close` | `paths/meeting/close_breakout_room.yaml` |
| POST | `/api/v1/meeting/breakout-rooms/{breakout_room_id}/join` | `paths/meeting/join_breakout_room.yaml` |
| GET | `/api/v1/meeting/breakout-rooms/{breakout_room_id}/participants` | `paths/meeting/breakout_room_participants.yaml` |
| POST | `/api/v1/meeting/breakout-rooms/{breakout_room_id}/invite` | `paths/meeting/invite_participant_to_breakout_room.yaml` |
| PATCH | `/api/v1/meeting/breakout-rooms/{breakout_room_id}/name` | `paths/meeting/rename_breakout_room.yaml` |
| PATCH | `/api/v1/meeting/breakout-rooms/{breakout_room_id}` | `paths/meeting/update_breakout_room.yaml` |
| GET, POST | `/api/v1/meeting/breakout-rooms/{breakout_room_id}/chat` | `paths/meeting/breakout_chat_messages.yaml` |
| POST | `/api/v1/meeting/breakout-rooms/chat/{message_id}/reactions` | `paths/meeting/breakout_chat_reactions.yaml` |
| GET | `/api/v1/meeting/{meeting_id}/participants` | `paths/meeting/meeting_participants.yaml` |
| POST | `/api/v1/meeting/breakout-rooms/{breakout_room_id}/join-request` | `paths/meeting/accept_breakout_invitation.yaml` |
| POST | `/api/v1/meeting/breakout-join-requests/{request_id}/approve` | `paths/meeting/approve_breakout_join.yaml` |
| POST | `/api/v1/meeting/breakout-join-requests/{request_id}/reject` | `paths/meeting/reject_breakout_join.yaml` |
| POST | `/api/v1/meeting/breakout-rooms/{breakout_room_id}/join-request/cancel` | `paths/meeting/cancel_breakout_waiting.yaml` |
| GET | `/api/v1/meeting/breakout-rooms/{breakout_room_id}/waiting` | `paths/meeting/list_breakout_waiting.yaml` |
| GET | `/api/v1/search` | `paths/search.yaml` |
| POST | `/api/v1/admin/search/reindex` | `paths/admin_search_reindex.yaml` |
| GET | `/api/v1/notifications` | `paths/notifications/list_notifications.yaml` |
| POST | `/api/v1/notifications/read` | `paths/notifications/mark_read.yaml` |
| POST, DELETE | `/api/v1/notifications/channels/{channel_id}/mute` | `paths/notifications/mute_notifications.yaml` |

## HTTP API Themes

### Auth and Security

Auth endpoints cover registration, login, refresh, logout, profile, verification, password reset, Google login, magic links, sessions, password change, and 2FA. Source paths:

- `aloqa-backend/shared/api/api-gateway/v1/paths/register_user.yaml`
- `aloqa-backend/shared/api/api-gateway/v1/paths/login_user.yaml`
- `aloqa-backend/shared/api/api-gateway/v1/paths/security/`

### Organization and Workspace

Organization endpoints cover companies, workspaces, members, channels, roles, available permissions, invites, kicks, public channels, storage, and quotas. Source paths:

- `aloqa-backend/shared/api/api-gateway/v1/paths/create_company.yaml`
- `aloqa-backend/shared/api/api-gateway/v1/paths/create_workspace.yaml`
- `aloqa-backend/shared/api/api-gateway/v1/paths/create_channel.yaml`

### Messaging

Messaging endpoints cover channel messages, direct messages, edits, deletion, restore, reactions, pins, channel read state, unread count, threads, replies, forward, block, and unblock. Source path: `aloqa-backend/shared/api/api-gateway/v1/paths/messaging/`.

### Files

File endpoints cover upload, file metadata, file content redirect/download, shares, user storage, and user file lists. Source path: `aloqa-backend/shared/api/api-gateway/v1/paths/file/`.

### Meetings

Meeting endpoints are extensive. They cover create, read, update, end, join, waiting room, chat, threads, reactions, admins, settings, permissions, device requests, pinning, moderation, bans, breakout rooms, breakout chat, breakout join/waiting flows, and participant lists. Source path: `aloqa-backend/shared/api/api-gateway/v1/paths/meeting/`.

### Search and Notifications

Search endpoints cover search and admin reindex. Notifications endpoints cover list, mark read, and channel mute/unmute. Source paths:

- `aloqa-backend/shared/api/api-gateway/v1/paths/search.yaml`
- `aloqa-backend/shared/api/api-gateway/v1/paths/admin_search_reindex.yaml`
- `aloqa-backend/shared/api/api-gateway/v1/paths/notifications/`

## gRPC Surface

The backend uses protobuf/gRPC internally. Source path: `aloqa-backend/shared/proto/`.

Services identified from proto source:

- `AuthService`: registration, login, refresh, logout, sessions, verification, password reset, Google login, 2FA, settings, privacy, avatar/profile, magic links.
- `OrgService`: companies, workspaces, channels, members, invites, roles, ABAC permission flows.
- `MessagingService`: send/list/edit/delete/restore messages, DMs, reactions, pins, read state, threads, block/unblock.
- `FileService`: upload, URLs, deletion, shares, metadata, storage, quotas.
- `MeetingService`: meeting lifecycle, join/waiting, chat, admins, room settings, permissions, device requests, moderation, pins, breakout rooms, breakout chat, waiting/join flows.
- `NotificationService`: email sends, web notifications, read state, channel mute/unmute.
- `PresenceService`: connect, disconnect, gateway lease, single/bulk presence.
- `SearchService`: search, ping, reindex.

The gRPC surface is larger than the HTTP surface in some areas because backend services also expose internal operations that the gateway composes.

## Frontend API Client

The frontend API client:

- builds paths from a route registry
- sends JSON or FormData
- supports `Authorization` and `Cookie` headers when configured
- handles 401 refresh logic
- parses JSON responses
- supports blob and beacon-style requests

Source path: `aloqa-frontend/packages/core/src/api/client.ts`.

The frontend route registry is a static list of expected backend routes. Source path: `aloqa-frontend/packages/core/src/api/routes.ts`.

## Web BFF API

The web BFF route handler is a same-origin proxy for browser REST calls. It:

- reads the sealed app session
- forwards the backend `access_token` as a Cookie header
- refreshes stale tokens for replayable requests
- applies CSRF checks for mutating requests
- separates upload proxying to avoid replaying non-replayable streams

Source paths:

- `aloqa-frontend/apps/web/app/api/[...path]/route.ts`
- `aloqa-frontend/apps/web/app/api/upload/[...path]/route.ts`
- `aloqa-frontend/apps/web/src/lib/auth/withCsrf.ts`

## Contract Risks

### OpenAPI vs Frontend Route Registry

The backend OpenAPI source and frontend route registry must be kept in lockstep:

- `aloqa-backend/shared/api/api-gateway/v1/api-gateway.openapi.yaml`
- `aloqa-frontend/packages/core/src/api/routes.ts`

This should be automated in CI.

### BFF vs Direct Gateway Paths

The frontend BFF architecture wants browser `/api/*` to route to Next.js. The backend nginx routes `/api/v1/` to the API gateway. This is a deployment contract conflict unless a higher-level edge config resolves it.

Source paths:

- `aloqa-frontend/docs/adr/0037-web-bff-backend-session.md`
- `aloqa-frontend/deploy/nginx.prod.conf`
- `aloqa-backend/deploy/prod/nginx/nginx.conf`

### Schema-Level Details

This audit lists paths and service capabilities, but it does not inline every request and response schema. The authoritative schemas are the individual OpenAPI path files and component schemas under `aloqa-backend/shared/api/api-gateway/v1/`. I cannot determine runtime compatibility without generating clients and running contract tests.
