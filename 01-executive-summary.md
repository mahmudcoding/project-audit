# 01. Executive Summary

## What is Aloqa?

Aloqa is a workplace communication product.

It helps people in a company:

- log in securely
- create workspaces
- create channels
- send messages
- send direct messages
- upload files
- search old content
- receive notifications
- join video meetings
- manage members, roles, and permissions

If you know Slack, Microsoft Teams, or Discord for work, Aloqa is in the same product family.

## What problem does it solve?

Companies need one place where employees can work together.

Without a tool like Aloqa, work spreads across:

- email
- private chats
- video-call links
- shared drives
- screenshots
- spreadsheets
- manual permission lists

Aloqa tries to bring these into one connected system.

## Who uses it?

Different people use Aloqa for different reasons:

| Person | What they need |
|---|---|
| Employee | Send messages, join meetings, upload files, search information |
| Team lead | Create channels, manage members, start meetings |
| Company admin | Manage workspaces, roles, permissions, storage |
| Support or operations team | See notifications, search history, manage access |
| Product manager | Plan features and understand what changes affect |
| Engineer | Build, fix, and operate the system |

## Imagine one company using Aloqa for one day

### Morning

An employee opens Aloqa.

```text
Employee
  -> opens web, desktop, or mobile app
  -> clicks Login
  -> enters email and password
  -> sees workspace list
```

Behind the scenes:

```text
Frontend screen
  -> login request
  -> backend checks password
  -> database checks user and session
  -> frontend receives success
```

Why this matters for a PM:

- Login must be reliable.
- If login breaks, every other feature becomes unreachable.
- Auth changes affect web, desktop, mobile, backend, and database sessions.

### Mid-morning

The employee sends a message in a channel.

```text
Employee types message
  -> clicks Send
  -> message appears
  -> teammates see it live
```

Behind the scenes:

```text
Frontend
  -> API Gateway
  -> Messaging Service
  -> Database
  -> Kafka event delivery
  -> WebSocket Gateway
  -> other users receive update
```

Real-life analogy:

- The database is the warehouse where the message is stored.
- Kafka is like an internal mail service that delivers "new message" notices.
- WebSocket is like a phone call that stays open so updates arrive instantly.

### Noon

The team joins a meeting.

```text
User clicks Join Meeting
  -> backend checks permission
  -> meeting service prepares room state
  -> LiveKit handles audio and video
  -> WebSocket sends meeting updates
```

Why this matters:

- Meetings are more complex than normal messages.
- They involve permissions, live video, waiting rooms, device requests, and realtime updates.
- This is an expensive area to modify.

### Afternoon

Someone uploads a file.

```text
User selects file
  -> frontend sends upload
  -> backend scans file
  -> backend stores file in MinIO
  -> database stores file metadata
  -> other users can access it if allowed
```

Real-life analogy:

- MinIO is the storage room for file contents.
- The database stores the label on the box: owner, file name, permissions, size, and status.
- ClamAV is the security guard that checks the file for malware.

### Evening

The user searches for an old message and logs out.

```text
User searches
  -> Search Service asks OpenSearch
  -> results return

User logs out
  -> Auth Service closes session
  -> frontend returns to login state
```

## The big architecture in one picture

```text
Users
  |
  v
Frontend apps
  |-- Web app
  |-- Desktop app
  |-- Mobile app
  |
  v
API Gateway and WebSocket Gateway
  |
  v
Backend services
  |-- Auth
  |-- Organization
  |-- Messaging
  |-- Files
  |-- Meetings
  |-- Notifications
  |-- Search
  |
  v
Storage and infrastructure
  |-- PostgreSQL database
  |-- Redis short-term memory
  |-- Kafka delivery service
  |-- MinIO file storage
  |-- OpenSearch search index
  |-- LiveKit video/audio
```

## The most important things to know first

### 1. Aloqa is two projects working together

The frontend project is here:

```text
aloqa-frontend/
```

The backend project is here:

```text
aloqa-backend/
```

The frontend cannot do much alone. It needs backend services to store data, check permissions, and send realtime updates.

### 2. The backend is split into departments

A backend "microservice" means a separate backend department.

Example:

- Auth Service handles login.
- Messaging Service handles messages.
- File Service handles files.
- Search Service handles search.

Why this project needs it:

- The product has many complex areas.
- Separate services keep responsibilities clearer.
- A bug in one area is easier to locate.

Where it is used:

```text
aloqa-backend/auth-service/
aloqa-backend/messaging-service/
aloqa-backend/file-service/
aloqa-backend/search-service/
```

### 3. The web app uses a BFF

BFF means "Backend for Frontend."

Plain English:

The browser does not send most requests straight to the backend. It first talks to a small server layer inside the web app.

Analogy:

The BFF is a receptionist.

```text
Browser
  -> BFF receptionist
  -> backend service desk
```

Why Aloqa needs it:

- It keeps sensitive login tokens safer.
- It lets the web app refresh login state without exposing everything to browser code.
- It gives one place to check browser request safety.

Where it is used:

```text
aloqa-frontend/apps/web/app/api/
aloqa-frontend/apps/web/src/lib/auth/
```

### 4. Realtime is a core feature, not a bonus

Messages, notifications, and meeting updates should appear quickly.

That means Aloqa uses:

- WebSocket for always-open client connections
- Kafka for backend event delivery
- Redis for short-term state
- LiveKit for meeting audio/video

## Main strengths

- The project has a real product structure, not just demo code.
- Backend services have clear responsibilities.
- Frontend has separate web, desktop, and mobile apps.
- API contracts are written down.
- The database model covers many real collaboration features.
- The web app has a thoughtful login/security design.

## Main risks

### Risk 1: Production routing is unclear

One frontend deployment file says browser API traffic should go to the web BFF.

One backend deployment file sends `/api/v1/` traffic directly to the backend gateway.

Important files:

```text
aloqa-frontend/deploy/nginx.prod.conf
aloqa-backend/deploy/prod/nginx/nginx.conf
```

Why a PM should care:

If the wrong route is active in production, login and security behavior may not match the design.

I cannot determine from the codebase which nginx file is the real production edge today.

### Risk 2: Backend tests look too thin

The backend has many services, but the backend guidance says tests are mostly absent except limited benchmark coverage.

Important file:

```text
aloqa-backend/AGENTS.md
```

Why a PM should care:

When a product has login, permissions, messaging, files, and meetings, missing tests can turn small changes into risky changes.

### Risk 3: Realtime has many moving parts

A message can pass through:

```text
Frontend
  -> API Gateway
  -> Messaging Service
  -> Database
  -> Kafka
  -> WebSocket Gateway
  -> other users
```

If one part breaks, the message may save correctly but not appear live.

### Risk 4: Meetings are expensive to change

Meetings touch:

- backend meeting service
- LiveKit
- WebSocket events
- Redis room state
- database tables
- frontend call screens
- mobile and desktop behavior

Changing meeting behavior is likely expensive.

## What you should remember

- Aloqa is a workplace communication platform.
- Users log in, chat, meet, upload files, search, and receive notifications.
- The frontend shows screens; the backend does the trusted work.
- The backend is split into service departments.
- The database stores the long-term truth.
- Redis is short-term memory.
- Kafka is internal delivery.
- WebSocket is a live phone line to users.
- LiveKit handles video and audio calls.
- The biggest risks are routing clarity, backend test coverage, and realtime complexity.
