# Spec Delta

## Purpose

Defines the version-control tree behind scenarios: immutable nodes, branching instead of overwriting, undo and redo that never destroy work, jumping between nodes, returning to live, and saving and sharing trees, so users can explore and compare different choices safely.

## ADDED Requirements

### Requirement: Tree of immutable nodes

Each scenario SHALL be a tree whose root is the base moment and whose other nodes each record a parent, one edit and the model version. Nodes are immutable and content-addressed: the same edit applied to the same parent yields the same node.

#### Scenario: Repeating an edit

- **WHEN** a user applies an identical edit to the same node twice
- **THEN** one child node exists, not two

### Requirement: Editing from the past branches

Editing from a node that already has children SHALL create a new branch. No existing node is overwritten or removed by editing.

#### Scenario: Branch from an earlier node

- **WHEN** a user returns to an earlier node and applies a different edit
- **THEN** a sibling branch is created and the original branch and its nodes are untouched

### Requirement: Undo and redo walk the tree

Undo SHALL move to the parent node and redo to the most recently visited child, offering a choice when several children exist. Neither deletes anything.

#### Scenario: Undo then redo across a fork

- **WHEN** a user undoes twice then redoes at a node with two children
- **THEN** the product selects the most recently visited child, or offers the choice if none was visited

### Requirement: Jump to any node and return

Users SHALL be able to jump to any node from the graph. The previous position MUST be remembered separately from undo, with a back action to return to where they were.

#### Scenario: Jump and go back

- **WHEN** a user jumps from node A to a distant node B and chooses back
- **THEN** the view returns to node A with its state, selection and time unchanged

### Requirement: Observed trunk with attached branches

The graph SHALL show observed time as a trunk, with each scenario tree attached at the observed moment it was based on. Several trees may attach at different moments.

#### Scenario: Two bases

- **WHEN** a user has built scenarios from yesterday evening and from this morning
- **THEN** each tree attaches to its own moment on the trunk, in time order

### Requirement: Return to live preserves the tree

One action SHALL return the view to the live observed head without discarding the tree. The last scenario position MUST remain reachable through a resume action.

#### Scenario: Live and back to scenario

- **WHEN** a user in a scenario chooses live, watches new frames arrive, then chooses resume
- **THEN** the live view shows observed values throughout, and resume restores the same node and selection

### Requirement: Observed time never alters nodes

New observed frames SHALL never change an existing node. The graph MUST show how far live has advanced beyond each tree's base.

#### Scenario: Base falls behind

- **WHEN** three hours of new frames arrive while a scenario is open
- **THEN** the scenario's values are unchanged and the graph labels its base as three hours behind live

### Requirement: Labels, notes and pins

Nodes and branches SHALL accept labels, notes and pins. These are metadata and MUST NOT change node identity.

#### Scenario: Rename a node

- **WHEN** a user renames a node
- **THEN** its identity, children and share links are unchanged

### Requirement: Archive instead of destroy

Users SHALL be able to collapse a subtree in view and archive a branch. Archived branches remain recoverable until the tree's retention period ends.

#### Scenario: Archive then restore

- **WHEN** a user archives a branch then restores it
- **THEN** all its nodes and labels return exactly as before

### Requirement: Trees are saved to the visitor's workspace when settled

Each committed node SHALL be saved server-side into the visitor's own workspace (see `mix-visitor-state`) as soon as it settles, on any app node. Saving state (saved, saving, failed) is always visible, and an unacknowledged node stays in a local outbox and is retried until acknowledged.

#### Scenario: Reload mid-exploration

- **WHEN** the page is reloaded after several edits
- **THEN** the tree, current node and labels are restored from the visitor's workspace

#### Scenario: Save failure

- **WHEN** a save fails
- **THEN** the UI shows the failed state, keeps the node in the outbox, and retries without losing it or creating a duplicate

### Requirement: Shared views are read-only and fork on edit

A shared link SHALL open a read-only view of its snapshot, including the full edit history it carries (publishing and retention are defined in `mix-visitor-state`). Editing from it MUST start a new tree in the viewer's own workspace, never mutating the snapshot.

#### Scenario: Edit a shared node

- **WHEN** a viewer opens a shared node and applies an edit
- **THEN** a new tree is created in the viewer's workspace containing the shared ancestry plus the edit, and the shared snapshot is unchanged

### Requirement: Safe concurrent editing

Two sessions editing one tree SHALL both succeed without corrupting it: simultaneous children of the same node appear as siblings.

#### Scenario: Two tabs, one parent

- **WHEN** two tabs each add a different edit to the same node
- **THEN** the tree shows two sibling nodes and neither edit is lost

### Requirement: Resource limits

Trees SHALL have documented caps on node count and size, and creation rates SHALL be limited. At a cap the UI MUST explain and offer archiving.

#### Scenario: Node cap reached

- **WHEN** a tree reaches its node cap
- **THEN** further edits are refused with an explanation and an offer to archive branches

### Requirement: Model-version mismatch is explicit

When a node was evaluated with an older model version, the UI SHALL say so and offer to re-evaluate, and MUST NOT silently show changed numbers.

#### Scenario: Reopen after a model update

- **WHEN** a saved tree is opened after the model version changed
- **THEN** a notice names both versions and results stay as originally evaluated until the user chooses to re-evaluate

### Requirement: Graph panel scales and is navigable

The graph panel SHALL show the trunk, branches, current node, live marker and base staleness; support pan, zoom and a minimap; stay interactive at 200 nodes; and be fully keyboard navigable.

#### Scenario: Large tree

- **WHEN** a tree with 200 nodes is open
- **THEN** pan and zoom stay within the interaction frame budget and every node is reachable by keyboard

### Requirement: Node details and change log

Selecting a node SHALL show the ordered edits from the base with their descriptions and classes (grid, network or policy), the node's result summary, fidelity tier, its model version, and actions to jump, branch, compare, label, archive and share.

#### Scenario: Read the path to a node

- **WHEN** a user selects a node five edits deep
- **THEN** the panel lists all five edits in order with plain-language descriptions and offers the actions
