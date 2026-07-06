# Database and Persistence Audit

## Persistence Overview

Aloqa uses multiple persistence and state systems:

- PostgreSQL: source of truth for users, sessions, companies, workspaces, channels, messages, files, meetings, notifications, permissions, and outbox rows.
- Redis: cache, session-adjacent state, presence, room state, pub/sub, typing, notification fanout, and ephemeral realtime state.
- MinIO: object storage for uploaded files.
- OpenSearch: search index for messages, files, users, or other searchable entities.
- Kafka: durable event transport between services.
- LiveKit: media/session system for calls, with Aloqa storing domain state separately.

Source paths:

- `aloqa-backend/platform/migrations/`
- `aloqa-backend/deploy/compose/core/docker-compose.yml`
- `aloqa-backend/deploy/prod/docker-compose.yml`
- `aloqa-backend/realtime-service/`
- `aloqa-backend/file-service/`
- `aloqa-backend/search-service/`

## PostgreSQL Schema Scope

The database schema is defined by timestamped migrations in `aloqa-backend/platform/migrations/`.

The schema covers:

- account identity and sessions
- company/workspace/channel membership
- custom roles and ABAC permissions
- workspace invites and invite links
- direct message channels and user blocks
- messages, threads, reactions, pins, read cursors, idempotency keys
- files, file shares, quotas, thumbnails, and processing state
- user settings and privacy
- notifications and channel mutes
- meetings, meeting participants, admins, room settings, device modes, permission requests
- meeting chat, meeting threads, meeting reactions, recipient targeting, file attachments
- breakout rooms, breakout participants, breakout chat, breakout waiting and join requests
- waiting room and participant status
- moderation state such as meeting bans
- outbox tables for messaging, channel, and meeting events

This is a mature schema for a collaboration product, not a toy model.

## Core Identity and Organization Tables

The initial migration creates core tables and enums:

- `users`
- `sessions`
- `companies`
- `company_members`
- `workspaces`
- `workspace_members`
- `channels`
- `channel_members`
- `messages`
- `reactions`
- `workspace_invites`
- `workspace_type`
- `channel_type`
- `invitation_status`

Source path: `aloqa-backend/platform/migrations/20260504074928_init.*`.

These tables establish the main hierarchy:

```text
company
  -> company_members
  -> workspace
       -> workspace_members
       -> channel
            -> channel_members
            -> messages
```

Direct-message support later extends the channel model with `dm_channels` and a `dm` channel type.

## Roles and Permissions

Role and permission support evolves through multiple migrations:

- initial custom roles: `custom_roles`, `role_permissions`, `user_roles`
- `action_scope` enum
- later `custom_roles_v2` style permission strings and model/resource indexes
- guest role support through `is_guest`
- invite links with role and role IDs

Source paths:

- `aloqa-backend/platform/migrations/20260520061940_add_abac_roles.*`
- `aloqa-backend/platform/migrations/20260604000001_custom_roles_v2.*`
- `aloqa-backend/platform/migrations/20260616000001_role_is_guest.*`

This shows the authorization model moved from a simpler action/scope representation toward a more flexible permission-string model.

## Messaging Model

Messaging tables evolve to support:

- JSONB message body
- body text extraction
- mentions
- replies and threads
- reactions
- pins
- read cursors and message sequence
- channel member mute state
- forwarded messages
- idempotency keys
- direct-message blocks and clears
- outbox events

Source paths:

- `aloqa-backend/platform/migrations/20260522130000_messages_body_jsonb.*`
- `aloqa-backend/platform/migrations/20260525000000_message_reactions.*`
- `aloqa-backend/platform/migrations/20260525000001_message_pinned.*`
- `aloqa-backend/platform/migrations/20260526000001_message_read_cursor.*`
- `aloqa-backend/platform/migrations/20260603000001_message_reply_to.*`
- `aloqa-backend/platform/migrations/20260603000002_message_idempotency_key.*`
- `aloqa-backend/platform/migrations/20260610000001_messaging_outbox.*`
- `aloqa-backend/platform/migrations/20260617000001_dm_chat_logic.*`
- `aloqa-backend/platform/migrations/20260617000002_dm_chat_clears.*`
- `aloqa-backend/platform/migrations/20260619102244_messages_body_text.*`

The presence of idempotency keys and outbox tables indicates the backend is designed to handle retries and asynchronous event publication.

## File Model

File persistence includes:

- file records
- file IDs attached to messages
- file shares
- workspace storage settings
- user quotas
- thumbnail and processing fields
- cascade behavior for shares
- meeting file-share type support

Source paths:

- `aloqa-backend/platform/migrations/20260608000003_files.*`
- `aloqa-backend/platform/migrations/20260609000001_message_file_ids.*`
- `aloqa-backend/platform/migrations/20260609000002_file_shares.*`
- `aloqa-backend/platform/migrations/20260610000002_workspace_storage_settings.*`
- `aloqa-backend/platform/migrations/20260626000001_meeting_chat_file_ids.*`
- `aloqa-backend/platform/migrations/20260626000002_file_shares_meeting_type.*`
- `aloqa-backend/platform/migrations/20260627000001_files_user_quota.*`

Uploaded bytes live in MinIO, while Postgres stores metadata, access, sharing, and quota state.

## Meeting Model

Meeting persistence is the most complex database area.

It includes:

- `meetings`
- `meeting_participants`
- meeting settings
- join requests
- optional channel association
- meeting chat messages
- meeting chat threads and mentions
- meeting chat reactions
- meeting admins and admin permissions
- room settings
- participant permissions
- permission requests
- device requests
- pin state
- meeting outbox
- meeting bans
- waiting room participant statuses
- breakout rooms
- breakout room participants
- breakout-room visibility/allowed participants
- breakout-room chat
- breakout join requests
- breakout waiting

Source paths:

- `aloqa-backend/platform/migrations/20260522150000_add_video_meetings.*`
- `aloqa-backend/platform/migrations/20260618101959_meeting_settings.*`
- `aloqa-backend/platform/migrations/20260618113454_meeting_join_requests.*`
- `aloqa-backend/platform/migrations/20260619085238_meeting_chat_messages.*`
- `aloqa-backend/platform/migrations/20260619120000_meeting_chat_threads.*`
- `aloqa-backend/platform/migrations/20260622072718_meeting_admins.*`
- `aloqa-backend/platform/migrations/20260622072719_meeting_admin_permissions.*`
- `aloqa-backend/platform/migrations/20260622072736_meeting_room_settings.*`
- `aloqa-backend/platform/migrations/20260622074215_meeting_participant_permissions.*`
- `aloqa-backend/platform/migrations/20260622075916_meeting_permission_requests.*`
- `aloqa-backend/platform/migrations/20260622080000_breakout_rooms.*`
- `aloqa-backend/platform/migrations/20260622081959_meeting_device_requests.*`
- `aloqa-backend/platform/migrations/20260622082932_meeting_pin.*`
- `aloqa-backend/platform/migrations/20260622085254_meeting_outbox.*`
- `aloqa-backend/platform/migrations/20260624051928_meeting_bans.*`
- `aloqa-backend/platform/migrations/20260624100000_breakout_room_chat.*`
- `aloqa-backend/platform/migrations/20260624110000_waiting_room.*`
- `aloqa-backend/platform/migrations/20260625120000_breakout_join_requests.*`

This database area requires careful integration tests because meeting behavior is state-machine-heavy.

## Outbox Tables

The migrations include outbox tables for asynchronous events:

- `messaging_outbox`
- `channels_outbox`
- `meeting_outbox`

Source paths:

- `aloqa-backend/platform/migrations/20260610000001_messaging_outbox.*`
- `aloqa-backend/platform/migrations/20260612000001_channels_outbox.*`
- `aloqa-backend/platform/migrations/20260622085254_meeting_outbox.*`

The outbox pattern is a strong reliability choice. It lets services commit business data and event intent in the same database transaction, then relay events to Kafka after commit.

The risk is operational: stuck outbox rows, duplicate event publication, poison events, or consumer lag need monitoring.

## Redis Usage

Redis appears in auth/session-adjacent behavior, realtime room state, WebSocket notification pub/sub, presence, typing, and cache behavior.

Source paths:

- `aloqa-backend/auth-service/`
- `aloqa-backend/realtime-service/internal/infrastructure/redis/`
- `aloqa-backend/ws-gateway/`
- `aloqa-backend/deploy/prod/docker-compose.yml`

Realtime Redis keys include room state, pin state, online users, participant state, typing, presence, and reaction-limit-style state. Source path: `aloqa-backend/realtime-service/internal/infrastructure/redis/room_state.go`.

Redis data should be treated as reconstructable or ephemeral unless a specific key is proven durable.

## MinIO and File Storage

MinIO stores uploaded file bytes. Postgres stores metadata and access state. ClamAV scans uploaded content. File processing uses media/document tooling in the file service.

Source paths:

- `aloqa-backend/file-service/`
- `aloqa-backend/deploy/compose/core/docker-compose.yml`
- `aloqa-backend/deploy/prod/docker-compose.yml`

Critical questions for production:

- Are buckets private by default?
- Are signed URLs short-lived?
- Are failed scans quarantined?
- Are large uploads streamed safely?
- Are thumbnails derived only from safe, scanned content?

I cannot determine all production storage policies from the codebase alone.

## OpenSearch

OpenSearch is used by `search-service` for full-text search and reindex behavior.

Source paths:

- `aloqa-backend/search-service/`
- `aloqa-backend/shared/proto/search/`
- `aloqa-backend/deploy/prod/docker-compose.yml`

Search should be assumed eventually consistent unless product requirements explicitly guarantee read-after-write search behavior.

## Database Risks

### Migration Volume and State Machines

The schema has many migrations, especially for meeting features. Complex state machines need both migration tests and behavior tests.

### Cross-Service Ownership

Many services use the same Postgres database. That can be pragmatic, but it weakens strict microservice data isolation. Table ownership should be documented so one service does not casually depend on another service's internals.

### Outbox Monitoring

Outbox tables are good, but they need operational dashboards:

- oldest unprocessed row age
- retry count
- event publish failures
- consumer lag
- dead-letter behavior

I cannot determine from the codebase whether those dashboards exist.

### Data Retention

The codebase shows soft-delete-like fields and deletion/restore flows in places, but I cannot determine a complete retention policy for users, messages, files, meeting recordings, notifications, or audit logs from the codebase.

## Database Assessment

The database design matches a feature-rich collaboration system. The schema is broad and pragmatic. The most important improvements are not new tables; they are migration discipline, table ownership documentation, outbox observability, and integration tests for stateful workflows.
