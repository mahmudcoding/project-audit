# 10. Request Flows

## What is a request flow?

A request flow is the path a user action takes through the system.

It answers:

```text
User clicks something.
What happens next?
```

Why PMs should care:

Request flows help you estimate work and understand bugs.

If a bug happens after "Send message", the flow tells you which parts may be involved.

Quick terms before the flows:

- BFF means Backend for Frontend, the web app's helper server layer.
- API Gateway means the backend reception desk.
- Kafka means internal event delivery.
- WebSocket means a live connection that stays open.
- Redis means fast short-term memory.
- LiveKit means the audio/video meeting system.
- MinIO means file object storage.
- ClamAV means malware scanning for uploaded files.
- OpenSearch means the search engine.

## Flow 1: Web login

What it is:

A user signs in through the browser.

Why it exists:

The user needs to prove identity before seeing private company data.

Journey:

```text
User
  -> opens web app
  -> enters email and password
  -> BFF receives login request
  -> API Gateway forwards request
  -> Auth Service checks credentials
  -> Database checks user/session data
  -> BFF stores safe session cookie
  -> user enters workspace
```

Diagram:

```text
User
  -> Web app
  -> BFF
  -> API Gateway
  -> Auth Service
  -> PostgreSQL
  -> BFF
  -> Workspace screen
```

Important files:

```text
aloqa-frontend/apps/web/src/lib/auth/sessionCookie.ts
aloqa-frontend/apps/web/app/api/[...path]/route.ts
aloqa-backend/auth-service/
aloqa-backend/shared/api/api-gateway/v1/paths/login_user.yaml
```

What can break:

- login page fails
- token storage fails
- backend rejects valid user
- browser session is not saved
- user is immediately logged out

## Flow 2: Normal web API request

What it is:

A logged-in browser asks the backend for something.

Example:

User opens a channel list.

Journey:

```text
Browser
  -> calls /api/... on same website
  -> BFF reads safe session
  -> BFF forwards to backend
  -> API Gateway asks service
  -> service returns data
  -> BFF returns data to browser
  -> screen updates
```

Diagram:

```text
Browser
  -> BFF
  -> API Gateway
  -> Backend Service
  -> Database
  -> Backend Service
  -> API Gateway
  -> BFF
  -> Browser
```

Important files:

```text
aloqa-frontend/apps/web/app/api/[...path]/route.ts
aloqa-frontend/apps/web/src/lib/auth/sessionRefresh.ts
```

What can break:

- BFF cannot read session
- backend token expires
- refresh fails
- API route path is wrong
- edge routing bypasses BFF

## Flow 3: Send message

What it is:

A user sends a chat message and other users receive it live.

Journey:

```text
Alice types "Hello"
  -> clicks Send
  -> frontend calls messaging API
  -> API Gateway calls Messaging Service
  -> Messaging Service saves message
  -> database stores message
  -> outbox row is created
  -> Kafka carries event
  -> WebSocket Gateway sends event
  -> Bob sees "Hello"
```

Diagram:

```text
Alice
  -> Chat UI
  -> API Gateway
  -> Messaging Service
  -> PostgreSQL
  -> Kafka
  -> WebSocket Gateway
  -> Bob
```

Important files:

```text
aloqa-backend/messaging-service/
aloqa-backend/ws-gateway/
aloqa-backend/platform/migrations/20260610000001_messaging_outbox.*
aloqa-frontend/packages/core/src/realtime/events.ts
```

What can break:

- message save fails
- message saves but live event fails
- duplicate live event appears
- unread count is wrong
- blocked user can still message

## Flow 4: Upload file

What it is:

A user uploads a file to share or attach.

Journey:

```text
User selects file
  -> web upload route streams file
  -> API Gateway receives upload
  -> File Service scans file
  -> MinIO stores file bytes
  -> PostgreSQL stores file record
  -> frontend shows uploaded file
```

Diagram:

```text
User
  -> Upload UI
  -> BFF upload route
  -> API Gateway
  -> File Service
  -> ClamAV scan
  -> MinIO storage
  -> PostgreSQL metadata
```

Important files:

```text
aloqa-frontend/apps/web/app/api/upload/[...path]/route.ts
aloqa-backend/file-service/
aloqa-backend/shared/api/api-gateway/v1/paths/file/upload_file.yaml
```

What can break:

- large upload fails
- scan fails
- file stores but metadata fails
- metadata stores but file access is wrong
- user sees a file they should not see

## Flow 5: Join meeting

What it is:

A user enters a video/audio meeting.

Journey:

```text
User clicks Join
  -> frontend sends join request
  -> API Gateway calls Realtime Service
  -> Realtime Service checks permission
  -> database stores participant state
  -> Redis stores live room state
  -> LiveKit handles media
  -> WebSocket Gateway sends room updates
```

Diagram:

```text
User
  -> Call UI
  -> API Gateway
  -> Realtime Service
  -> PostgreSQL
  -> Redis
  -> LiveKit
  -> WebSocket Gateway
```

Important files:

```text
aloqa-backend/realtime-service/
aloqa-backend/shared/api/api-gateway/v1/paths/meeting/join_meeting.yaml
aloqa-frontend/packages/features/calls/
aloqa-frontend/docs/adr/0022-livekit-client-sdk.md
```

What can break:

- user cannot join
- waiting room does not update
- media token fails
- LiveKit connects but product state is wrong
- participant permissions are wrong

## Flow 6: Notification

What it is:

A user receives an alert about something.

Journey:

```text
Something happens
  -> backend creates notification event
  -> Notification Service stores notification
  -> Redis or Kafka helps deliver it
  -> WebSocket Gateway sends live update
  -> frontend shows badge or notification
```

Diagram:

```text
Backend event
  -> Notification Service
  -> PostgreSQL
  -> Redis/Kafka
  -> WebSocket Gateway
  -> User screen
```

Important files:

```text
aloqa-backend/notification-service/
aloqa-backend/ws-gateway/
aloqa-backend/shared/api/api-gateway/v1/paths/notifications/
```

## Flow 7: Search

What it is:

A user searches old content.

Journey:

```text
User types query
  -> frontend calls search API
  -> API Gateway calls Search Service
  -> Search Service asks OpenSearch
  -> results return to frontend
```

Diagram:

```text
User
  -> Search UI
  -> API Gateway
  -> Search Service
  -> OpenSearch
  -> Search results
```

Important files:

```text
aloqa-backend/search-service/
aloqa-backend/shared/api/api-gateway/v1/paths/search.yaml
aloqa-frontend/packages/features/search/
```

## How to use flows in planning

When planning a feature, ask:

- What user action starts the flow?
- Which frontend app is involved?
- Does it go through the BFF?
- Which API endpoint is involved?
- Which backend service owns the rule?
- Which database tables change?
- Does realtime event delivery happen?
- Does search or notification need an update?
- Which platforms need UI changes?

## What you should remember

- A request flow is the path from user action to system result.
- Login touches frontend, BFF, API Gateway, Auth Service, and database.
- Messages use both normal API and realtime delivery.
- Files use upload, scan, object storage, and metadata.
- Meetings use backend state plus LiveKit media.
- Notifications and search often happen after other features create events.
- Request flows help PMs estimate real feature cost.
