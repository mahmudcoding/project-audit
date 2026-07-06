# Realtime Audit

## Realtime Summary

Aloqa realtime behavior is split between:

- WebSocket gateway for product events.
- LiveKit for audio/video media.
- Kafka/outbox for durable backend event propagation.
- Redis for ephemeral room, presence, typing, notification, and coordination state.

Source paths:

- `aloqa-backend/ws-gateway/`
- `aloqa-backend/realtime-service/`
- `aloqa-backend/messaging-service/`
- `aloqa-backend/platform/migrations/`
- `aloqa-frontend/packages/core/src/realtime/client.ts`
- `aloqa-frontend/packages/core/src/realtime/events.ts`
- `aloqa-frontend/docs/adr/0022-livekit-client-sdk.md`

## WebSocket Gateway

`ws-gateway` owns persistent WebSocket connections and event fanout.

Responsibilities:

- authenticate socket clients
- handle subscribe/unsubscribe messages
- manage channel and meeting subscriptions
- process typing events
- process device-state events
- process reactions
- publish room sync behavior
- consume Kafka events
- fan out chat and meeting updates
- deliver user-specific notifications through Redis pub/sub
- coordinate presence/gateway leases

Source paths:

- `aloqa-backend/ws-gateway/internal/ws/messages.go`
- `aloqa-backend/ws-gateway/internal/ws/client.go`
- `aloqa-backend/ws-gateway/`

## Client WebSocket Protocol

The backend WebSocket message definitions include channel and meeting subscription messages, typing, device state, reactions, ping/pong-style behavior, and room sync behavior.

Source paths:

- `aloqa-backend/ws-gateway/internal/ws/messages.go`
- `aloqa-backend/ws-gateway/internal/ws/client.go`

The frontend client maps logical rooms such as chat channels and meetings into backend subscription payloads. Source path: `aloqa-frontend/packages/core/src/realtime/events.ts`.

## Frontend Realtime Client

The frontend realtime client supports:

- idempotent connect
- token injection/refresh hooks
- reconnect with exponential backoff and jitter
- heartbeat every configured interval
- missed pong handling
- resume key and resume sequence support
- both envelope-style and flat backend frames

Source path: `aloqa-frontend/packages/core/src/realtime/client.ts`.

This is a good baseline for production realtime clients. The next step is validating it against backend behavior under network failure, token expiry, and backend restart scenarios.

## Event Families

Frontend event names cover:

- message created, edited, deleted, restored
- reaction updates
- typing
- notifications
- meeting updates
- breakout room updates
- waiting room updates
- room settings
- pin updates
- admin updates
- device requests and device state

Source path: `aloqa-frontend/packages/core/src/realtime/events.ts`.

Backend producers live across:

- `aloqa-backend/messaging-service/`
- `aloqa-backend/realtime-service/`
- `aloqa-backend/ws-gateway/`

The event map should become a versioned contract.

## Chat Realtime Flow

Expected flow:

1. Client sends a message through HTTP API.
2. API gateway calls `messaging-service`.
3. Messaging service writes message data to Postgres.
4. Messaging service writes an outbox event.
5. Outbox relay publishes to Kafka.
6. WebSocket gateway consumes the Kafka event.
7. WebSocket gateway sends event to subscribed channel clients.
8. Frontend realtime client normalizes and dispatches event to app bridges.

Source paths:

- `aloqa-backend/messaging-service/`
- `aloqa-backend/platform/migrations/20260610000001_messaging_outbox.*`
- `aloqa-backend/ws-gateway/`
- `aloqa-frontend/packages/core/src/realtime/bridges.ts`

## Meeting Realtime Flow

Expected flow:

1. Client creates, joins, or updates meeting state through HTTP API.
2. API gateway calls meeting/realtime gRPC service.
3. Realtime service persists meeting state in Postgres.
4. Realtime service writes meeting outbox rows and/or Redis state.
5. Kafka carries durable meeting events.
6. WebSocket gateway fans out meeting state updates.
7. LiveKit handles media-plane audio/video.
8. Frontend call UI responds to WebSocket state and LiveKit state.

Source paths:

- `aloqa-backend/realtime-service/`
- `aloqa-backend/platform/migrations/20260622085254_meeting_outbox.*`
- `aloqa-backend/ws-gateway/`
- `aloqa-frontend/packages/features/calls/`
- `aloqa-frontend/docs/adr/0022-livekit-client-sdk.md`

## LiveKit

The frontend architecture explicitly chooses LiveKit Client SDK for WebRTC. Source path: `aloqa-frontend/docs/adr/0022-livekit-client-sdk.md`.

The backend production compose includes LiveKit. Source path: `aloqa-backend/deploy/prod/docker-compose.yml`.

The realtime service uses LiveKit server-side dependencies and issues meeting/media behavior. Source path: `aloqa-backend/realtime-service/`.

This is the correct decision. Custom WebRTC signaling/media infrastructure would be much riskier.

## Redis Realtime State

Redis stores ephemeral meeting and presence state such as:

- room state
- pin state
- online users
- participant state
- typing state
- presence
- reaction limit state

Source path: `aloqa-backend/realtime-service/internal/infrastructure/redis/room_state.go`.

Redis is also used by the WebSocket gateway for user-specific notification pub/sub such as `notif:<userID>` style channels.

Redis should not be the only source of truth for durable meeting history.

## Outbox and Kafka

Realtime fanout relies on durable outbox tables:

- `messaging_outbox`
- `meeting_outbox`
- `channels_outbox`

Source paths:

- `aloqa-backend/platform/migrations/20260610000001_messaging_outbox.*`
- `aloqa-backend/platform/migrations/20260612000001_channels_outbox.*`
- `aloqa-backend/platform/migrations/20260622085254_meeting_outbox.*`

Kafka carries events from services to consumers such as WebSocket gateway and search.

The outbox pattern is strong, but operationally it needs:

- retry visibility
- duplicate handling
- dead-letter handling
- event schema versioning
- consumer lag alerts

## Realtime Risks

### Event Contract Drift

Backend event producers and frontend event names can drift. Source paths:

- `aloqa-backend/ws-gateway/`
- `aloqa-frontend/packages/core/src/realtime/events.ts`

This should be caught with contract tests.

### Duplicate Events

Outbox/Kafka systems can deliver duplicate events. Clients and consumers should be idempotent where possible.

The frontend realtime client supports resume sequence concepts, but I cannot determine from static inspection whether every backend event has stable sequencing semantics.

### Reconnect and Auth Expiry

Realtime clients must survive token expiry, network loss, server restarts, and duplicate subscriptions. The frontend client has reconnect support, but end-to-end tests are needed against `ws-gateway`.

### Meeting State Complexity

Meetings combine:

- Postgres durable state
- Redis room state
- WebSocket events
- LiveKit state
- role/admin permissions
- waiting rooms
- breakout rooms

That is a lot of state to keep consistent.

## Recommended Realtime Tests

Add tests for:

- socket connect with valid token
- socket reject with invalid token
- subscribe/unsubscribe channel
- message event delivered once or idempotently handled
- reconnect and resume after network drop
- token refresh during reconnect
- meeting join and room sync
- waiting room admit/reject
- meeting admin permission update
- device request accept/decline
- breakout room create/join/close
- duplicate Kafka event handling
- Redis restart behavior where acceptable

## Realtime Assessment

The realtime architecture is appropriate for the product, but it is operationally sensitive. The next maturity step is event schema versioning, integration tests, and observability for outbox, Kafka, WebSocket fanout, Redis room state, and LiveKit joins.
