# Spec Delta

## Purpose

Defines who owns what: each anonymous visitor gets a private workspace holding their own scenario graphs and edit history for a short period, and can publish shareable links that keep their changes and full history alive for others to open, with no accounts and minimal data.

## ADDED Requirements

### Requirement: Anonymous visitor identity created on first save

A visitor SHALL be identified by an opaque, unguessable identifier (at least 128 bits) held in a first-party cookie that is set only when they first save work, never on first page view. It MUST contain no personal data and MUST NOT be used for cross-site tracking.

#### Scenario: Browsing without saving

- **WHEN** a visitor views the map and scrubs time without committing an edit
- **THEN** no visitor identifier cookie is set

#### Scenario: First save

- **WHEN** the visitor commits their first edit
- **THEN** the identifier is created, and a short notice states what is stored and for how long

### Requirement: A private workspace per visitor

Each visitor SHALL have their own workspace containing their own scenario trees, nodes, labels and current positions, and SHALL be able to hold several trees and switch between them.

#### Scenario: Two visitors, two histories

- **WHEN** two visitors each make different edits at the same time
- **THEN** each sees only their own graph and history

### Requirement: Isolation between visitors

Every read and write on a workspace SHALL be authorised against the requesting visitor's identity. No visitor MAY read or change another visitor's workspace, and identifiers MUST NOT be enumerable.

#### Scenario: Guessing another tree

- **WHEN** a visitor requests a tree identifier belonging to another visitor
- **THEN** the response is indistinguishable from not found

### Requirement: Short idle retention

A workspace SHALL be deleted after a configured idle period (default 7 days since last activity). Any activity extends it, and the workspace SHALL show when it will expire.

#### Scenario: Return within retention

- **WHEN** a visitor returns on the same browser after three days
- **THEN** their trees, nodes, labels and current position are restored and the expiry extends

#### Scenario: Expiry

- **WHEN** a workspace has been idle for the retention period
- **THEN** it and all its unshared data are deleted

### Requirement: Shareable links publish a snapshot with history

A visitor SHALL be able to publish a link to a chosen branch or to a whole tree. The link carries the selected nodes and their complete edit history as an immutable snapshot, with labels included and personal notes only if the visitor opts in.

#### Scenario: Share a branch

- **WHEN** a visitor shares the path to a node
- **THEN** the link opens a read-only view of that node with its ancestors, edits and results, and later changes to the visitor's tree do not alter it

#### Scenario: Notes are private by default

- **WHEN** a visitor shares a tree containing notes without opting in to notes
- **THEN** the shared view omits the notes

### Requirement: Shared links outlive the workspace

A shared snapshot SHALL persist independently of the publisher's workspace, for a configured period since it was last opened (default 30 days), and opening it MUST extend that period.

#### Scenario: Publisher workspace expires

- **WHEN** the publisher's workspace is deleted but their shared link was opened recently
- **THEN** the link still opens the snapshot

#### Scenario: Link lapses

- **WHEN** a shared link has not been opened for the retention period
- **THEN** it shows a clear expired message and its data is deleted

### Requirement: Viewers fork on first edit

Opening a shared link SHALL be read-only. A viewer's first edit MUST copy the shared ancestry into a new tree in the viewer's own workspace, creating their identity if needed, and leave the snapshot untouched.

#### Scenario: Continue someone else's scenario

- **WHEN** a viewer edits from a shared node
- **THEN** a new tree appears in their workspace with the shared history plus their edit, and the original link still shows the original

### Requirement: Revoke and clear

A visitor SHALL be able to revoke any link they published, and to clear their workspace on demand, deleting their trees and, if chosen, their links, with a confirmation.

#### Scenario: Clear my data

- **WHEN** a visitor confirms clearing their workspace
- **THEN** their trees are deleted immediately and they are told what remains (links they chose to keep)

### Requirement: Quotas and abuse limits

Workspaces SHALL have documented caps on trees, shares and total stored size, and publishing and creation SHALL be rate limited. Hitting a cap MUST explain the cap and offer clearing or archiving.

#### Scenario: Share cap reached

- **WHEN** a visitor reaches their share cap
- **THEN** publishing is refused with an explanation and an offer to revoke older links

### Requirement: Data minimisation

Stored workspace data SHALL consist of scenario content, identifiers and timestamps only. IP addresses and device details MUST NOT be stored with it, and logs MUST NOT contain workspace content.

#### Scenario: Data audit

- **WHEN** a stored workspace record is inspected
- **THEN** it holds no IP address, device fingerprint or personal field

### Requirement: Losing the cookie is explained

Because there are no accounts, a visitor who loses their identifier cannot recover their private workspace. The product SHALL say so at first save and point to sharing as the way to keep or move work.

#### Scenario: New device

- **WHEN** a visitor wants their work on another device
- **THEN** they publish a link, open it there, and fork it into that device's workspace
