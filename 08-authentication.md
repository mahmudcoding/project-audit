# 08. Authentication and Authorization

## What is authentication?

Authentication means proving who the user is.

Plain English:

It answers:

```text
Are you really this user?
```

Real-life analogy:

Authentication is showing your ID at the building entrance.

## What is authorization?

Authorization means deciding what the user is allowed to do.

Plain English:

It answers:

```text
Now that we know who you are, what doors can you open?
```

Real-life analogy:

Authorization is the access badge that lets you enter some rooms but not others.

## Why Aloqa needs both

Example:

```text
Alice logs in
  -> authentication proves she is Alice

Alice tries to delete a channel
  -> authorization checks whether Alice may delete that channel
```

Both are required.

If authentication fails, the user should not enter.

If authorization fails, the user may enter Aloqa but cannot perform that action.

## Login journey

```text
1. User opens Aloqa.
2. User clicks Login.
3. User enters email and password.
4. Frontend sends login request.
5. Backend checks the user record.
6. Backend checks password.
7. Backend creates session.
8. Frontend stores login state safely.
9. User sees workspace.
```

Diagram:

```text
User
  -> Frontend
  -> BFF for web users
  -> API Gateway
  -> Auth Service
  -> Database
  -> response returns
```

## What is a session?

A session means "this user is currently logged in."

Analogy:

A session is a visitor pass.

When you enter a building, reception gives you a pass. You use it until you leave or it expires.

Why Aloqa needs it:

Users should not type their password on every click.

Where sessions are used:

```text
aloqa-backend/auth-service/
aloqa-backend/platform/migrations/20260504074928_init.*
```

## What is a token?

A token is a signed proof that a user is logged in.

Plain English:

It is a digital pass that backend services can check.

Why Aloqa needs it:

Each request must prove who is making it.

Important file:

```text
aloqa-backend/platform/pkg/utils/auth.go
```

## Web login and the BFF

The web app uses a BFF.

BFF means "Backend for Frontend."

Analogy:

The BFF is a receptionist sitting between the browser and backend.

```text
Browser
  -> BFF receptionist
  -> backend auth service
```

Why Aloqa needs it:

Sensitive login information should not be handled directly by browser code when it can be kept safer on the server side.

What happens:

```text
Backend returns login tokens
  -> BFF seals them into a safe cookie
  -> browser keeps the cookie
  -> future requests go through BFF
```

Where it is used:

```text
aloqa-frontend/apps/web/src/lib/auth/sessionCookie.ts
aloqa-frontend/apps/web/src/lib/auth/sessionRefresh.ts
aloqa-frontend/apps/web/app/api/[...path]/route.ts
```

## What is a sealed cookie?

A cookie is a small piece of data stored by the browser.

A sealed cookie is protected so the browser cannot casually read or change the sensitive contents.

Analogy:

It is like a locked envelope. The browser carries it, but the server is the one that can safely open it.

Why Aloqa needs it:

The web app needs a safer way to remember backend login information.

## What is CSRF protection?

CSRF means a bad website tries to trick a logged-in browser into making a request.

Plain English:

Imagine someone tricks your browser into clicking a dangerous button without you seeing it.

Why Aloqa needs protection:

The web app uses cookies, and cookies are sent automatically by browsers. Mutating requests need extra checks.

Where it is used:

```text
aloqa-frontend/apps/web/src/lib/auth/withCsrf.ts
```

## Password reset journey

```text
User clicks Forgot password
  -> enters email
  -> backend creates reset flow
  -> Notification Service sends email
  -> user clicks reset link
  -> backend verifies link
  -> user sets new password
```

Affected services:

```text
auth-service
notification-service
api-gateway
```

## 2FA journey

2FA means two-factor authentication.

Plain English:

The user needs password plus a second proof, usually a code.

Why Aloqa needs it:

It protects accounts even if a password is stolen.

Journey:

```text
User enables 2FA
  -> backend sends code
  -> user confirms code
  -> 2FA becomes active

Later login
  -> user enters password
  -> backend asks for 2FA code
  -> user enters code
  -> login completes
```

Affected files:

```text
aloqa-backend/shared/api/api-gateway/v1/paths/security/
aloqa-backend/auth-service/
```

## Google login and magic links

Google login:

The user proves identity through Google.

Magic link:

The user receives a special login link by email.

Why they exist:

They reduce password friction and support modern login flows.

Affected services:

```text
auth-service
notification-service
api-gateway
```

## Authorization and permissions

After login, Aloqa checks what the user can do.

Example:

```text
User tries to invite member
  -> backend checks company/workspace role
  -> if allowed, invite is created
  -> if not allowed, request is rejected
```

Important term: ABAC.

ABAC means attribute-based access control.

Plain English:

The backend decides permission by looking at facts:

- who is the user?
- what company are they in?
- what workspace is this?
- what role do they have?
- what action are they trying to perform?

Where it is used:

```text
aloqa-backend/org-service/internal/core/abac/
aloqa-backend/platform/pkg/permissions/
```

## Features affected by auth and permissions

| Feature | Why auth matters |
|---|---|
| Messaging | Users should only see channels they can access |
| Files | Users should only open files they are allowed to see |
| Meetings | Users need join, admin, mute, kick, ban rules |
| Admin | Only allowed users should manage members and roles |
| Search | Search should not reveal private content |

## Important files

```text
aloqa-backend/auth-service/
aloqa-backend/shared/proto/auth/
aloqa-backend/platform/pkg/utils/auth.go
aloqa-backend/org-service/internal/core/abac/
aloqa-backend/platform/pkg/permissions/
aloqa-frontend/apps/web/src/lib/auth/sessionCookie.ts
aloqa-frontend/apps/web/src/lib/auth/sessionRefresh.ts
aloqa-frontend/apps/web/src/lib/auth/withCsrf.ts
```

## What can break if auth changes?

- users cannot log in
- users stay logged in too long
- users are logged out too often
- 2FA blocks valid users
- password reset links fail
- frontend and backend disagree about session state
- unauthorized users gain access
- authorized users are denied

## Unknowns from code alone

I cannot determine live production cookie behavior from the repo alone because it depends on domains, TLS, proxy headers, and the active edge routing.

## What you should remember

- Authentication proves who the user is.
- Authorization decides what the user can do.
- A session is like a visitor pass.
- A token is digital proof used by requests.
- The web BFF protects browser login handling.
- CSRF protection helps stop browser trick attacks.
- ABAC means permission decisions use facts about user, action, and resource.
- Auth changes are high risk because every platform depends on them.
