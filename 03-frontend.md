# 03. Frontend

## What is the frontend?

The frontend is everything the user sees and clicks.

It includes:

- the web app in a browser
- the desktop app
- the mobile app
- buttons, forms, message lists, file views, meeting controls, settings pages

Real-life analogy:

The frontend is the shop floor. Customers do not see the warehouse or office departments. They see counters, signs, forms, and staff who take requests.

## Why does it exist?

The backend can store data and enforce rules, but users need a usable product.

The frontend turns backend capabilities into real user actions:

```text
Click Login
Click Send
Click Upload
Click Join Meeting
Click Search
Click Invite Member
```

## The three frontend apps

```text
aloqa-frontend/
  apps/web/       -> browser app
  apps/desktop/   -> Electron desktop app
  apps/mobile/    -> Expo mobile app
```

### Web app

What it is:

The browser version of Aloqa.

Why it exists:

Users can open Aloqa from a normal web browser without installing anything.

Where it is used:

```text
aloqa-frontend/apps/web/
```

Example user journey:

```text
User opens website
  -> sees login page
  -> logs in
  -> sees workspace
  -> opens a channel
  -> sends message
```

The web app also has a special backend-like layer called the BFF.

### Desktop app

What it is:

The installed desktop version of Aloqa.

Why it exists:

Some users prefer a dedicated app with desktop behavior, window controls, deep links, and call surfaces.

Where it is used:

```text
aloqa-frontend/apps/desktop/
```

### Mobile app

What it is:

The phone version of Aloqa.

Why it exists:

Users need chat, calls, files, and notifications away from their computer.

Where it is used:

```text
aloqa-frontend/apps/mobile/
```

Mobile has extra concerns:

- secure phone storage
- push notifications
- native sharing
- mobile call behavior
- reconnecting on unstable networks

## What is the BFF?

BFF means "Backend for Frontend."

Plain English:

The web browser does not always talk directly to the backend. It talks to a helper layer inside the web app first.

Analogy:

The BFF is a receptionist.

```text
Browser
  -> BFF receptionist
  -> backend reception desk
  -> backend department
```

Why Aloqa needs it:

- keep login tokens safer
- refresh login state on the server side
- protect browser requests
- make the browser talk to the same website origin

Where it is used:

```text
aloqa-frontend/apps/web/app/api/
aloqa-frontend/apps/web/src/lib/auth/
```

## Example: login in the web app

```text
1. User clicks Login.
2. Browser sends email and password to the web app.
3. The BFF forwards the request to the backend.
4. Backend checks the user.
5. Backend returns login tokens.
6. BFF seals those tokens inside a safe cookie.
7. Browser shows the workspace.
```

Diagram:

```text
User
  -> Web screen
  -> BFF
  -> API Gateway
  -> Auth Service
  -> Database
  -> back to BFF
  -> workspace screen
```

## Example: sending a chat message

```text
1. User types a message.
2. Frontend calls the API.
3. Backend saves the message.
4. Frontend shows the sent message.
5. WebSocket sends the new message to other users.
```

Diagram:

```text
Chat screen
  -> API client
  -> backend
  -> realtime client
  -> screen updates
```

## Shared frontend code

Some frontend code is shared by all apps.

This shared code lives mainly here:

```text
aloqa-frontend/packages/core/
```

What it does:

- builds API URLs
- sends API requests
- handles realtime WebSocket connections
- defines shared event names
- stores shared business logic

Why it exists:

Without shared code, the web, desktop, and mobile apps would each rewrite the same logic.

## Platform-first design

The frontend architecture says:

```text
Share logic.
Do not force every platform to share the same UI.
```

Plain English:

The mobile screen, desktop screen, and web screen can look different because users use them differently. But the rules behind them should be shared where possible.

Where this is explained:

```text
aloqa-frontend/docs/adr/0023-platform-first-architecture.md
```

Why a PM should care:

If a feature is added, it may require:

- shared logic in `packages/core`
- web UI
- desktop UI
- mobile UI
- tests for each app

That means "add a small feature" can be bigger than it sounds.

## Feature packages

Feature packages are organized areas such as:

```text
aloqa-frontend/packages/features/chat/
aloqa-frontend/packages/features/files/
aloqa-frontend/packages/features/calls/
aloqa-frontend/packages/features/search/
aloqa-frontend/packages/features/settings/
aloqa-frontend/packages/features/admin/
```

What they are:

Shared feature code for major product areas.

Why they exist:

They keep chat logic near chat code, file logic near file code, and so on.

Important note:

Some feature packages still include platform-specific UI exports. The architecture wants more shared logic and less shared UI over time. This is technical debt, not an emergency.

## What screens are affected by common features?

| Feature | Web | Desktop | Mobile |
|---|---|---|---|
| Login | auth pages | desktop auth flow | mobile auth routes |
| Chat | workspace channel pages | renderer chat screens | mobile chat routes |
| Calls | call pages | desktop call surface | mobile LiveKit/call screens |
| Files | files pages and upload BFF | desktop file views | mobile files routes |
| Settings | workspace/user settings | desktop settings | mobile settings |
| Search | search pages | desktop search | mobile search if implemented |

## Important files

Use these after you understand the frontend story.

```text
aloqa-frontend/package.json
aloqa-frontend/AGENTS.md
aloqa-frontend/apps/web/
aloqa-frontend/apps/web/app/api/[...path]/route.ts
aloqa-frontend/apps/web/app/api/upload/[...path]/route.ts
aloqa-frontend/apps/web/app/api/realtime/ws-ticket/route.ts
aloqa-frontend/apps/web/src/lib/auth/sessionCookie.ts
aloqa-frontend/apps/desktop/
aloqa-frontend/apps/mobile/
aloqa-frontend/packages/core/src/api/client.ts
aloqa-frontend/packages/core/src/api/routes.ts
aloqa-frontend/packages/core/src/realtime/client.ts
aloqa-frontend/packages/core/src/realtime/events.ts
aloqa-frontend/docs/adr/
```

## What can break if frontend changes?

Changing frontend code can break:

- login screens
- token refresh
- file uploads
- WebSocket reconnects
- mobile navigation
- desktop calls
- shared API route names
- platform-specific UI

Change cost depends on platform count:

```text
Web only        -> lower cost
Shared core     -> medium to high cost
Web + desktop + mobile -> high cost
Auth or realtime -> high risk
```

## What you should remember

- The frontend is what users see and click.
- Aloqa has web, desktop, and mobile apps.
- The web app has a BFF, which is a safe receptionist for browser requests.
- Shared logic lives mostly in `packages/core`.
- Feature packages group product areas like chat, files, calls, and search.
- Platform-first means each app can have its own UI.
- A feature may need work in all three apps.
- Auth, upload, and realtime frontend changes are higher risk.
