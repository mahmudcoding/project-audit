# 15. Project Manager Guide

## What is this guide?

This is the practical guide for managing Aloqa work.

It is not a code manual.

It helps you ask better questions, understand estimates, and see why some requests are small while others are expensive.

## The one-sentence product summary

Aloqa is a multi-platform workplace communication product with chat, files, meetings, search, notifications, roles, and permissions.

Quick terms used in this guide:

- API Gateway means the backend reception desk.
- BFF means Backend for Frontend, the web app's helper server layer.
- Outbox means a database tray of events to send later.
- Kafka means internal event delivery.
- WebSocket means a live connection that stays open.
- LiveKit means the audio/video meeting system.

## The one-minute architecture summary

```text
Users
  -> web, desktop, or mobile app
  -> API Gateway or BFF
  -> backend service department
  -> database or infrastructure
  -> realtime update if needed
```

## How to think about feature work

Every feature should be discussed in layers.

```text
User action
  -> screen
  -> API
  -> backend service
  -> database
  -> realtime event
  -> other platforms
```

If a request only changes one screen, it may be small.

If it changes backend rules, permissions, database, and realtime, it is not small.

## Feature readiness checklist

Use this table before calling a feature done.

| Question | Why it matters |
|---|---|
| Does the API exist? | frontend cannot finish without backend door |
| Does backend logic exist? | API path alone is not enough |
| Does web UI exist? | browser users need it |
| Does desktop UI exist? | desktop parity may be required |
| Does mobile UI exist? | mobile users may expect it |
| Are permissions correct? | prevents data leaks |
| Does realtime update? | chat and meetings depend on it |
| Is it searchable? | users may need to find it later |
| Are notifications needed? | users may need alerts |
| Are tests present? | reduces release risk |
| Is there a smoke test? | proves production path works |

## Questions to ask engineers

For any feature:

```text
Which user action starts this?
Which screens change?
Which API endpoint is used?
Which backend service owns it?
Which tables change?
Does it need realtime?
Does it need notifications?
Does it need search indexing?
Does it affect permissions?
Does it affect all platforms?
What test proves it works?
```

## How to estimate feature cost

Low cost:

- text change
- small UI-only change
- display existing data differently

Medium cost:

- new API field
- new frontend form
- small backend rule
- one-platform feature

High cost:

- auth/session change
- permission change
- database migration
- realtime event change
- file upload/access change
- meeting behavior change
- multi-platform feature

## Product areas and likely owners

| Product area | Likely backend owner | Likely frontend owner |
|---|---|---|
| Login/security | auth-service | auth screens and web BFF |
| Workspaces/channels | org-service | workspace and settings UI |
| Chat/DMs | messaging-service | chat screens |
| Files | file-service | files UI and upload routes |
| Meetings | realtime-service | calls UI |
| Notifications | notification-service | notification UI and realtime bridges |
| Search | search-service | search UI |
| Live updates | ws-gateway | realtime client |

## How to handle bug reports

Start with the user action.

Example: "Message sent but teammate did not see it."

Ask:

```text
Did the message save?
Did sender get success?
Did database store it?
Did outbox event create?
Did Kafka deliver?
Did WebSocket Gateway receive?
Did teammate socket stay connected?
Did frontend event handler update UI?
```

This prevents blaming the wrong team too early.

## How to handle requests for "simple" changes

Some requests sound simple but are not.

Example:

"Add a setting so only admins can speak in a meeting."

This may require:

```text
new setting in database
new backend permission rule
new meeting API field
new WebSocket event
new web UI
new desktop UI
new mobile UI
LiveKit behavior check
tests
```

So the PM question is not "is the setting simple?"

The PM question is:

```text
Which layers does it touch?
```

## Release risk areas

Highest risk:

- login/session changes
- permissions
- file access
- meeting behavior
- realtime event changes
- production routing
- database migrations

Medium risk:

- new API fields
- new notification behavior
- search behavior
- admin screens

Lower risk:

- copy changes
- purely visual changes
- UI layout changes that do not affect data

## Suggested roadmap hygiene

For each epic, keep a small matrix:

```text
Feature:
User journey:
Backend service:
API endpoint:
Database tables:
Frontend apps:
Realtime needed:
Search needed:
Notifications needed:
Tests:
Smoke test:
Known risks:
```

## What to ask before release

- Did we test the production entry path?
- Did web BFF routing work?
- Did login and refresh work?
- Did WebSocket connect?
- Did file upload work?
- Did meeting join work?
- Did migrations run?
- Did smoke tests pass?
- Are rollback steps known?

## What you should remember

- Always start from the user action.
- Then trace screen, API, backend, database, realtime.
- "Simple" features can be expensive if they touch permissions, meetings, files, or realtime.
- Web, desktop, and mobile may all need work.
- Backend route existence does not prove frontend readiness.
- Frontend screen existence does not prove backend readiness.
- Good PM questions reduce surprise.
