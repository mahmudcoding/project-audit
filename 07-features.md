# 07. Features

## What is this chapter?

This chapter explains Aloqa by product feature.

For each feature, it answers:

- What is it?
- Why does the business need it?
- Who probably requested it?
- What user journey does it support?
- Which backend services are affected?
- Which frontend screens are affected?
- Which database tables are affected?
- What could break if it changes?
- How expensive is it likely to modify?

## Feature map

```text
Aloqa
  |-- Account and security
  |-- Companies and workspaces
  |-- Channels and roles
  |-- Messaging and direct messages
  |-- Files
  |-- Meetings and calls
  |-- Notifications
  |-- Search
  |-- Realtime updates
```

Quick terms used in this chapter:

- BFF means Backend for Frontend, the web app's helper server layer.
- MinIO means storage for uploaded file contents.
- ClamAV means malware scanning for uploaded files.
- LiveKit means the audio/video meeting system.
- WebSocket means a live connection for instant updates.
- Kafka means internal event delivery.
- Redis means short-term live state storage.
- OpenSearch means the search engine.

## Account and security

What it is:

This is how users prove who they are.

It includes login, logout, password reset, email verification, 2FA, Google login, magic links, sessions, profile, and privacy settings.

Why the business needs it:

Companies will not trust a collaboration tool unless accounts are secure.

Who probably requested it:

- security
- compliance
- company admins
- product leadership
- users who need account recovery

User journey:

```text
User opens Aloqa
  -> clicks Login
  -> enters credentials
  -> backend checks identity
  -> user enters workspace
```

Affected backend:

```text
auth-service
api-gateway
platform auth utilities
```

Affected frontend:

```text
web auth pages
desktop auth flow
mobile auth flow
web BFF session code
```

Affected database:

```text
users
sessions
user_settings
user_privacy
two-factor related fields
```

What could break:

- users cannot log in
- sessions expire incorrectly
- 2FA blocks valid users
- password reset links fail
- web and mobile behave differently

Modification cost:

High. Auth touches every platform and has security risk.

## Companies, workspaces, and channels

What it is:

This is the structure of the workplace inside Aloqa.

```text
Company
  -> Workspace
      -> Channel
          -> Messages and members
```

Why the business needs it:

Companies need to separate teams, departments, projects, and permissions.

Who probably requested it:

- company admins
- team leads
- operations managers
- enterprise customers

User journey:

```text
Admin creates company
  -> creates workspace
  -> creates channel
  -> invites members
  -> members start chatting
```

Affected backend:

```text
org-service
api-gateway
platform permissions
```

Affected frontend:

```text
workspace screens
channel sidebar
admin screens
settings screens
invite screens
```

Affected database:

```text
companies
company_members
workspaces
workspace_members
channels
channel_members
workspace_invites
```

What could break:

- users see the wrong workspace
- private channels become visible
- invites stop working
- channel membership becomes wrong

Modification cost:

Medium to high. Structure changes affect many screens and permission checks.

## Roles and permissions

What it is:

Permissions decide what a user is allowed to do.

Analogy:

A permission is an access badge.

Some badges open normal doors. Admin badges open more doors.

Why the business needs it:

Companies need control over who can invite users, remove members, create channels, manage storage, or moderate meetings.

Who probably requested it:

- company admins
- security
- enterprise customers
- support teams

User journey:

```text
Admin opens role settings
  -> creates custom role
  -> selects allowed actions
  -> assigns role to user
  -> backend enforces permissions
```

Affected backend:

```text
org-service
platform permissions
api-gateway
```

Affected frontend:

```text
admin role screens
member management screens
settings screens
```

Affected database:

```text
custom_roles
role_permissions
user_roles
company_members
workspace_members
```

What could break:

- users get too much access
- admins lose access
- frontend shows a button that backend rejects
- backend allows action frontend hides

Modification cost:

High. Permission changes are security-sensitive.

## Messaging and direct messages

What it is:

This is chat.

It includes channel messages, direct messages, replies, reactions, pins, edits, deletes, read state, unread counts, and blocking.

Why the business needs it:

Messaging is the daily core of the product.

Who probably requested it:

- all users
- team leads
- support teams
- product leadership

User journey:

```text
User opens channel
  -> types message
  -> clicks Send
  -> message saves
  -> teammates see it live
  -> unread counts update
```

Affected backend:

```text
messaging-service
ws-gateway
api-gateway
search-service
notification-service
```

Affected frontend:

```text
chat screen
DM screen
thread screen
reaction UI
unread badges
realtime client
```

Affected database:

```text
messages
reactions
channel_members
dm_channels
user_blocks
messaging_outbox
```

What could break:

- messages do not save
- messages save but do not appear live
- unread counts become wrong
- blocked users can still message
- search misses messages

Modification cost:

Medium to high. Simple UI changes can be low, but backend or realtime changes are higher risk.

## Files

What it is:

File upload, file storage, file sharing, file access, and quotas.

Why the business needs it:

Teams need to share documents, images, PDFs, spreadsheets, and meeting files.

Who probably requested it:

- users
- team leads
- operations
- company admins
- security

User journey:

```text
User uploads file
  -> backend scans it
  -> file is stored
  -> metadata is saved
  -> allowed users can open it
```

Affected backend:

```text
file-service
api-gateway
MinIO
ClamAV
```

Affected frontend:

```text
file upload UI
chat attachment UI
file browser
meeting file UI
web upload BFF route
```

Affected database:

```text
files
file_shares
workspace_storage_settings
message file IDs
meeting chat file IDs
```

What could break:

- upload fails
- unsafe file is accepted
- user sees a file they should not see
- quota becomes wrong
- file preview fails

Modification cost:

High when permissions, scanning, quotas, or storage are involved.

## Meetings and calls

What it is:

Video/audio meetings plus product rules around them.

It includes:

- create meeting
- join meeting
- waiting room
- meeting chat
- admins
- room settings
- device requests
- mute, kick, ban
- breakout rooms

Why the business needs it:

Teams need live discussion, not only text chat.

Who probably requested it:

- users
- team leads
- remote teams
- enterprise customers

User journey:

```text
User clicks Join
  -> backend checks rules
  -> LiveKit connects audio/video
  -> WebSocket sends room updates
  -> meeting controls update live
```

Affected backend:

```text
realtime-service
ws-gateway
api-gateway
LiveKit
Redis
```

Affected frontend:

```text
call screens
meeting chat
waiting room UI
device request UI
breakout room UI
desktop call surface
mobile call surface
```

Affected database:

```text
meetings
meeting_participants
meeting_admins
meeting_room_settings
meeting_permission_requests
meeting_device_requests
meeting_chat_messages
breakout_rooms
breakout_room_participants
```

What could break:

- users cannot join calls
- waiting room gets stuck
- wrong user becomes admin
- microphone/camera rules fail
- breakout rooms behave incorrectly
- mobile and desktop diverge

Modification cost:

High. Meetings touch many backend services, database tables, realtime events, and platform clients.

## Notifications

What it is:

Notifications tell users something happened.

Examples:

- verification email
- password reset email
- 2FA email
- magic link email
- in-app notification
- channel mute/unmute

Why the business needs it:

Users need to know when something needs attention.

Affected backend:

```text
notification-service
ws-gateway
Kafka
Redis
```

Affected frontend:

```text
notification UI
badges
settings for muted channels
realtime client
```

Affected database:

```text
notifications
channel mute records
```

Modification cost:

Medium. Email changes can be lower risk; realtime notification changes are higher.

## Search

What it is:

Search helps users find old messages, files, or people.

Why the business needs it:

Collaboration tools become less useful if users cannot find past work.

User journey:

```text
User types search text
  -> frontend sends query
  -> Search Service asks OpenSearch
  -> results return
```

Affected backend:

```text
search-service
OpenSearch
Kafka consumers
```

Affected frontend:

```text
search screens
search result UI
```

Modification cost:

Medium. Ranking and indexing changes may be higher.

## What you should remember

- Features are not isolated. Most touch frontend, backend, database, and sometimes realtime.
- Auth and permissions are high risk because they affect access.
- Messaging is core daily usage.
- Files are security-sensitive.
- Meetings are the most complex product area.
- Search may be eventually consistent.
- Notifications depend on both stored data and live delivery.
- A PM should ask which services, screens, and tables a feature touches before estimating it.
