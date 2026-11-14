# Spec Delta

## Purpose

Defines how the browser and the Phoenix server stay in step: a page that paints without any live connection, one authenticated realtime channel for server pushes and acknowledged commits, honest catch-up after drops, and delivery that works whichever app node ingested the data.

## ADDED Requirements

### Requirement: First paint does not depend on the realtime connection

The page SHALL be delivered as a server-rendered shell with the latest frame and application manifest embedded, so the map island mounts and renders without any WebSocket or long-poll connection being established.

#### Scenario: Realtime blocked

- **WHEN** a visitor loads the page on a network that blocks WebSockets and long-polling
- **THEN** the map renders from the embedded state, and the connection indicator reports polling or offline rather than a blank page

### Requirement: Server pushes data, never markup, to the island

The island SHALL be the sole owner of its DOM. The server MUST send it data only (ticks, acknowledgements, frames), and no server-driven DOM patching may touch the island's subtree.

#### Scenario: Update while dragging

- **WHEN** a new tick arrives while a visitor is mid-drag
- **THEN** the drag continues undisturbed and the new data is applied after the drop

### Requirement: Polling fallback keeps data fresh

If the realtime channel is unavailable, the client SHALL fall back to conditional polling at the data cadence, so the map still advances without a channel.

#### Scenario: Channel unavailable for an hour

- **WHEN** the channel cannot connect for an hour
- **THEN** the client keeps fetching new frames by conditional requests and the connection indicator says so

### Requirement: Authenticated, origin-checked connection

The realtime connection SHALL be authenticated by a short-lived signed token issued with the page, accepted only from the public origin, and MUST NOT grant access to any visitor's workspace except the token holder's own.

#### Scenario: Forged or expired token

- **WHEN** a connection presents an expired or tampered token
- **THEN** it is refused, no workspace data is returned, and the page continues in polling mode

### Requirement: At-most-once pushes with catch-up

Because server pushes can be lost on drops, the client SHALL present its last seen frame on every (re)join and the server SHALL reply with what it missed (or the latest frame if the gap is large).

#### Scenario: Reconnect after a gap

- **WHEN** a client reconnects after missing several ticks
- **THEN** it receives the missed ticks or the latest frame and ends in the same state as a client that never dropped

### Requirement: Acknowledged, idempotent commits

Each committed edit SHALL be sent over the channel, acknowledged by the server, and kept in a client outbox until acknowledged. Re-sending a commit MUST have no additional effect.

#### Scenario: Drop mid-commit

- **WHEN** the connection drops after a commit was sent but before its acknowledgement arrived
- **THEN** the client re-sends on reconnect and exactly one node results

### Requirement: Delivery is independent of which node ingests

A newly ingested frame SHALL reach clients connected to any app node, without sticky sessions and without requiring the nodes to form a clustered runtime.

#### Scenario: Ingest on node A, clients on node B

- **WHEN** node A completes an ingest and clients are connected to node B
- **THEN** node B's clients are notified of the new frame within the documented delay

### Requirement: Reconnect storms are bounded

Clients SHALL reconnect with jittered backoff, and the server SHALL cap connections and message rates, so a restart or deploy does not trigger a stampede.

#### Scenario: Deploy restarts all nodes

- **WHEN** every node restarts and all clients reconnect
- **THEN** reconnects spread over the backoff window and the server remains within its connection and rate limits

### Requirement: Untrusted input and bounded messages

Every message from the client SHALL be validated and authorised against the sender's own workspace, with size and rate limits per connection. Violations MUST be rejected without affecting other visitors.

#### Scenario: Oversized or foreign payload

- **WHEN** a client sends an oversized message or one naming another visitor's tree
- **THEN** it is rejected with an error and no state changes

### Requirement: Connection and sync state is visible

The workspace SHALL show whether it is live, reconnecting, polling or offline, and how many changes are queued, so visitors never wonder whether work is saved.

#### Scenario: Offline with queued work

- **WHEN** the connection is lost after an edit
- **THEN** the indicator shows offline with one change queued, and shows saved once the acknowledgement arrives
