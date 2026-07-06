# 09. Realtime

## What does realtime mean?

Realtime means users see updates quickly without refreshing the page.

Examples:

- a new message appears
- someone is typing
- a notification arrives
- a meeting participant joins
- a host mutes someone
- a breakout room changes

Real-life analogy:

Normal API requests are like sending letters.

Realtime is like staying on a phone call.

```text
Letter:
User asks -> server replies -> connection ends

Phone call:
User connects -> connection stays open -> updates arrive anytime
```

## Why does Aloqa need realtime?

Aloqa is a collaboration product.

Users expect live behavior:

- chat should update immediately
- meetings should show participant changes
- notifications should appear without reload
- typing indicators should feel instant

Without realtime, Aloqa would feel slow and outdated.

## The main realtime parts

```text
Frontend realtime client
  -> WebSocket Gateway
  -> Kafka events
  -> backend services
  -> Redis short-term state
  -> LiveKit for audio/video
```

## What is WebSocket?

WebSocket is a connection that stays open between client and server.

Analogy:

It is a phone call that stays connected.

Why Aloqa needs it:

The backend can push updates to the user instead of waiting for the user to refresh.

Where it is used:

```text
aloqa-backend/ws-gateway/
aloqa-frontend/packages/core/src/realtime/client.ts
```

## What is the WebSocket Gateway?

The WebSocket Gateway is the backend service that manages live connections.

What it does:

- accepts user socket connections
- checks who the user is
- subscribes users to channels or meetings
- receives Kafka events
- sends events to the right users

Real-life analogy:

It is a phone switchboard operator. It keeps track of who is connected and sends the right call updates to the right people.

Where it is used:

```text
aloqa-backend/ws-gateway/
```

## What is Kafka in realtime?

Kafka is the internal delivery service.

Analogy:

Kafka is the company mailroom.

Example:

```text
Messaging Service saves a message
  -> drops an event into Kafka
  -> WebSocket Gateway picks it up
  -> users receive live update
```

Why Aloqa needs it:

Backend services should not all directly call each other for every live event. Kafka lets events move through the system in a more reliable way.

## What is Redis in realtime?

Redis is short-term memory.

Analogy:

It is a whiteboard in a meeting room.

It can store:

- who is online right now
- who is typing right now
- current room state
- temporary reaction limits
- notification delivery state

Where it is used:

```text
aloqa-backend/realtime-service/internal/infrastructure/redis/
aloqa-backend/ws-gateway/
```

## What is LiveKit?

LiveKit handles audio and video.

Analogy:

WebSocket is the meeting status phone line. LiveKit is the actual conference room with microphones and cameras.

Why Aloqa needs it:

Video/audio is hard to build correctly. Aloqa uses LiveKit for media, while Aloqa backend handles product rules like permissions, waiting rooms, and admins.

Where it is used:

```text
aloqa-backend/realtime-service/
aloqa-frontend/docs/adr/0022-livekit-client-sdk.md
```

## User journey: live chat message

```text
1. Alice sends a message.
2. Messaging Service saves it.
3. Messaging Service creates an event.
4. Kafka carries the event.
5. WebSocket Gateway receives it.
6. Bob's browser already has a WebSocket connection open.
7. Bob sees the message appear.
```

Diagram:

```text
Alice
  -> API Gateway
  -> Messaging Service
  -> Database
  -> Kafka
  -> WebSocket Gateway
  -> Bob
```

## User journey: typing indicator

```text
Alice starts typing
  -> frontend sends typing event
  -> WebSocket Gateway receives it
  -> Redis may store short-term typing state
  -> Bob sees "Alice is typing"
```

This state is short-lived. It does not need to be stored forever.

## User journey: meeting update

```text
Host mutes a participant
  -> frontend sends request
  -> Realtime Service checks permission
  -> database stores meeting state
  -> Redis stores room state
  -> Kafka/WebSocket sends update
  -> meeting screens update
```

Affected systems:

```text
realtime-service
ws-gateway
Redis
Kafka
LiveKit
frontend calls package
```

## Frontend realtime client

The frontend has a shared realtime client.

What it does:

- connects to WebSocket
- reconnects after network problems
- sends heartbeat checks
- gets a new token when needed
- receives backend events
- sends events to app screens

Where it is used:

```text
aloqa-frontend/packages/core/src/realtime/client.ts
aloqa-frontend/packages/core/src/realtime/events.ts
aloqa-frontend/packages/core/src/realtime/bridges.ts
```

## What can break in realtime?

Realtime can fail in partial ways.

Examples:

- message saves but does not appear live
- user receives duplicate event
- meeting state updates late
- WebSocket reconnect fails
- token expires during a socket connection
- Kafka event is delayed
- Redis loses temporary state
- mobile reconnect behavior differs from web

## Change cost guide

| Change | Cost | Why |
|---|---:|---|
| Add simple event display | Medium | frontend and backend event must match |
| Change event name | High | can break all clients |
| Change reconnect behavior | High | affects reliability |
| Change meeting realtime | High | many systems involved |
| Change LiveKit behavior | High | media, backend, clients, permissions |

## Important files

```text
aloqa-backend/ws-gateway/
aloqa-backend/ws-gateway/internal/ws/messages.go
aloqa-backend/ws-gateway/internal/ws/client.go
aloqa-backend/messaging-service/
aloqa-backend/realtime-service/
aloqa-backend/platform/migrations/20260610000001_messaging_outbox.*
aloqa-backend/platform/migrations/20260622085254_meeting_outbox.*
aloqa-frontend/packages/core/src/realtime/client.ts
aloqa-frontend/packages/core/src/realtime/events.ts
```

## Unknowns from code alone

The frontend has resume-related logic, but I cannot prove from static inspection that every backend event has complete sequencing and resume behavior.

## What you should remember

- Realtime means users see updates without refreshing.
- WebSocket is a phone call that stays open.
- WebSocket Gateway manages live client connections.
- Kafka is the internal mailroom for events.
- Redis is short-term memory for live state.
- LiveKit handles audio and video.
- Chat and meetings depend heavily on realtime.
- Realtime failures can be partial and hard to notice.
- Event changes should be treated as high risk.
