# 05. Database and Stored Data

## What is the database?

The database is where Aloqa stores long-term truth.

Real-life analogy:

The database is the company's warehouse and filing cabinet.

It stores records such as:

- users
- sessions
- companies
- workspaces
- channels
- messages
- files
- roles
- permissions
- meetings
- notifications

## Why does it exist?

If a server restarts, a user still expects:

- their account to exist
- messages to remain
- uploaded files to still be listed
- workspace membership to remain correct
- meeting history and settings to be remembered

That is why Aloqa needs a durable database.

## The main storage systems

Aloqa does not use only one storage tool.

```text
PostgreSQL  -> long-term business records
Redis       -> short-term memory
MinIO       -> actual uploaded file contents
OpenSearch  -> search index
Kafka       -> event delivery log
LiveKit     -> live media rooms
```

## What is PostgreSQL?

PostgreSQL is the main relational database.

Plain English:

It stores structured records in tables, like a very powerful spreadsheet system with rules.

Why Aloqa needs it:

Most business data must be stored carefully and queried later.

Where it is defined:

```text
aloqa-backend/platform/migrations/
```

## What is a migration?

A migration is a step-by-step database change.

Analogy:

If the database is a warehouse, a migration is a signed instruction like:

```text
Add a new shelf for file shares.
Add a new label to meeting participants.
Create a new cabinet for breakout rooms.
```

Why Aloqa needs migrations:

As the product grows, the database needs new tables and columns. Migrations keep those changes ordered.

Where they are stored:

```text
aloqa-backend/platform/migrations/
```

## User journey: login and sessions

```text
User logs in
  -> Auth Service checks users table
  -> Auth Service creates session
  -> session is stored
  -> future requests prove the user is logged in
```

Data involved:

- users
- sessions

Why PMs should care:

If these tables change, login can break for every user.

## User journey: workspace and channels

```text
Admin creates company
  -> company record is stored

Admin creates workspace
  -> workspace record is stored

Admin creates channel
  -> channel record is stored
  -> channel membership records are stored
```

Data involved:

- companies
- company_members
- workspaces
- workspace_members
- channels
- channel_members

Business reason:

Aloqa needs to know who belongs where and who can see what.

## User journey: message

```text
User sends message
  -> message row is stored
  -> optional reactions, replies, pins, mentions are stored
  -> read cursor tracks what users have read
```

Data involved:

- messages
- reactions
- channel_members read state
- message mentions
- threads/replies
- pins

Why PMs should care:

Changing message behavior can affect search, notifications, unread counts, and realtime updates.

## User journey: file upload

```text
User uploads file
  -> MinIO stores the file bytes
  -> PostgreSQL stores file metadata
  -> database stores who can access it
  -> quotas may be updated
```

Data involved:

- files
- file_shares
- workspace_storage_settings
- user quotas
- message file IDs
- meeting chat file IDs

Important distinction:

The database usually does not store the full file contents. It stores the record that describes the file. The actual file lives in MinIO.

## User journey: meeting

```text
User creates meeting
  -> meeting record is stored

User joins
  -> participant record is stored

Host changes settings
  -> room settings are stored

Host creates breakout room
  -> breakout room records are stored
```

Data involved:

- meetings
- meeting_participants
- meeting_settings
- meeting_admins
- meeting_room_settings
- meeting_permission_requests
- meeting_device_requests
- meeting_chat_messages
- breakout_rooms
- breakout_room_participants
- breakout_room_chat

Why PMs should care:

Meeting data is one of the most complex parts of the product. Small meeting changes can touch many tables.

## What is Redis?

Redis is fast short-term memory.

Analogy:

PostgreSQL is the permanent filing cabinet. Redis is the sticky note on someone's desk.

Why Aloqa needs it:

Some data changes too quickly to treat like permanent records.

Examples:

- who is currently online
- who is typing
- current meeting room state
- temporary notification delivery

Where it is used:

```text
aloqa-backend/realtime-service/internal/infrastructure/redis/
aloqa-backend/ws-gateway/
```

## What is MinIO?

MinIO stores uploaded files.

Analogy:

MinIO is the storage room. PostgreSQL stores the index card that says which box is which.

Where it is used:

```text
aloqa-backend/file-service/
aloqa-backend/deploy/prod/docker-compose.yml
```

## What is OpenSearch?

OpenSearch is the search engine.

Analogy:

The database is the warehouse. OpenSearch is the searchable index at the front desk.

Why Aloqa needs it:

Users expect fast search across messages, files, and people. Searching directly through every database record can be slow and limited.

Where it is used:

```text
aloqa-backend/search-service/
```

## What is an outbox table?

An outbox table stores events that need to be delivered later.

Analogy:

It is the "outgoing mail" tray on a desk.

Example:

```text
Message saved
  -> outbox row created
  -> background worker sends event to Kafka
  -> WebSocket Gateway notifies users
```

Why Aloqa needs it:

It prevents losing events when the database succeeds but the delivery system is temporarily unavailable.

Where it is used:

```text
aloqa-backend/platform/migrations/20260610000001_messaging_outbox.*
aloqa-backend/platform/migrations/20260612000001_channels_outbox.*
aloqa-backend/platform/migrations/20260622085254_meeting_outbox.*
```

## Important database files

```text
aloqa-backend/platform/migrations/
aloqa-backend/deploy/compose/core/docker-compose.yml
aloqa-backend/deploy/prod/docker-compose.yml
```

## What can break if database changes?

Changing database tables can break:

- login
- permissions
- message history
- file access
- unread counts
- meeting joins
- search indexing
- notifications
- migrations during deployment

Change cost:

```text
Add simple optional field       -> medium
Change required field           -> high
Change permission-related table -> high
Change meeting table            -> high
Delete or rename table/column   -> very high
```

## Unknowns from code alone

I cannot determine from the codebase alone:

- whether every production migration has run successfully
- production backup policy
- data retention policy for all data types
- whether dashboards exist for stuck outbox rows
- exact production storage hardening for uploaded files

## What you should remember

- PostgreSQL is the long-term source of truth.
- Migrations are ordered database changes.
- Redis is short-term memory.
- MinIO stores uploaded file contents.
- OpenSearch makes search fast.
- Kafka and outbox tables help deliver events.
- Meeting data is one of the most complex areas.
- Database changes are usually higher risk than UI-only changes.
- Permissions and file access changes need extra care.
