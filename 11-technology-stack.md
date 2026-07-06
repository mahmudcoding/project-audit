# 11. Technology Stack

## What is a technology stack?

A technology stack is the list of tools used to build and run the product.

Real-life analogy:

If Aloqa is a building, the stack is the set of materials and machines used to build it:

- concrete
- wiring
- elevators
- phones
- security system
- warehouse system

In software, these are programming languages, frameworks, databases, and infrastructure tools.

## Why PMs should care

The stack affects:

- hiring
- feature cost
- release risk
- performance
- security
- mobile support
- infrastructure cost
- time needed for fixes

## Backend stack

### Go

What it is:

Go is the programming language used by the backend.

Why Aloqa uses it:

Go is common for backend services that need concurrency, networking, and deployment simplicity.

Where it is used:

```text
aloqa-backend/
aloqa-backend/go.work
aloqa-backend/*/go.mod
```

### PostgreSQL

What it is:

The main database.

Why Aloqa uses it:

It stores users, messages, workspaces, files, permissions, and meetings.

Where it is used:

```text
aloqa-backend/platform/migrations/
```

### Redis

What it is:

Fast short-term memory.

Why Aloqa uses it:

Presence, room state, typing, notifications, and some cache-like behavior need fast temporary storage.

### Kafka

What it is:

Internal event delivery.

Why Aloqa uses it:

When a message or meeting update happens, other systems need to hear about it.

### MinIO

What it is:

Object storage for uploaded files.

Why Aloqa uses it:

Files are better stored in object storage than directly inside the database.

### ClamAV

What it is:

Malware scanning software.

Why Aloqa uses it:

Uploaded files should be checked before they are trusted.

### OpenSearch

What it is:

Search engine.

Why Aloqa uses it:

Users expect fast search over content.

### LiveKit

What it is:

Audio/video meeting platform.

Why Aloqa uses it:

Building reliable calls from scratch is expensive and risky.

## Frontend stack

### TypeScript

What it is:

JavaScript with type checking.

Why Aloqa uses it:

It helps catch mistakes before runtime.

Where it is used:

```text
aloqa-frontend/
```

### React

What it is:

A UI library for building screens.

Why Aloqa uses it:

Web, desktop renderer, and mobile UI can share React-style thinking.

### Next.js

What it is:

The web app framework.

Why Aloqa uses it:

It supports pages, server-side behavior, and special web route handlers.

Where it is used:

```text
aloqa-frontend/apps/web/
```

### BFF

What it is:

BFF means Backend for Frontend.

Plain English:

It is a helper server layer inside the web app.

Why Aloqa uses it:

It lets the browser send safer same-website requests while the web server handles sensitive backend session details.

Where it is used:

```text
aloqa-frontend/apps/web/app/api/
```

### Electron

What it is:

A way to build desktop apps using web technology.

Where it is used:

```text
aloqa-frontend/apps/desktop/
```

### Expo and React Native

What it is:

Tools for building mobile apps.

Where it is used:

```text
aloqa-frontend/apps/mobile/
```

### TanStack Query, Zustand, and Zod

Simple explanations:

- TanStack Query helps fetch and cache server data.
- Zustand helps manage app state.
- Zod helps validate data shapes.

Why Aloqa needs them:

Frontend apps need reliable data loading, local state, and validation.

## Build and workflow tools

### pnpm

Package manager for frontend dependencies.

### Turbo

Runs frontend monorepo tasks efficiently.

### Taskfile

Runs backend development tasks.

Where it is used:

```text
aloqa-backend/Taskfile.yml
```

### Docker Compose

Runs multiple services together.

Where it is used:

```text
aloqa-backend/deploy/compose/core/docker-compose.yml
aloqa-backend/deploy/prod/docker-compose.yml
```

## CI/CD

CI means automated checks before or during release.

CD means automated delivery/deployment steps.

Important files:

```text
aloqa-frontend/.github/workflows/
aloqa-backend/.github/workflows/deploy.yml
aloqa-frontend/docs/CICD.md
```

## Stack diagram

```text
User apps
  |-- Next.js web
  |-- Electron desktop
  |-- Expo mobile
        |
        v
Backend Go services
        |
        v
Infrastructure
  |-- PostgreSQL
  |-- Redis
  |-- Kafka
  |-- MinIO
  |-- ClamAV
  |-- OpenSearch
  |-- LiveKit
```

## What can make stack changes expensive?

- changing the database version
- changing auth libraries
- changing LiveKit behavior
- changing frontend framework versions
- changing mobile native dependencies
- changing Kafka event behavior
- changing production deployment tools

## Important files

```text
aloqa-backend/go.work
aloqa-backend/*/go.mod
aloqa-backend/deploy/prod/docker-compose.yml
aloqa-frontend/package.json
aloqa-frontend/apps/web/package.json
aloqa-frontend/apps/desktop/package.json
aloqa-frontend/apps/mobile/package.json
```

## What you should remember

- The backend is mainly Go.
- The frontend is mainly TypeScript and React.
- Web uses Next.js.
- Desktop uses Electron.
- Mobile uses Expo and React Native.
- PostgreSQL stores long-term data.
- Redis stores short-term state.
- Kafka moves events.
- MinIO stores files.
- LiveKit handles calls.
