# Spec Delta

## Purpose

The drag-integration boundary keeps Dragology's per-frame speculative rendering client-side while Phoenix owns persistence and broadcast, preventing LiveView from fighting React over the SVG tree.

## ADDED Requirements

### Requirement: Phoenix builds the Dragology client bundle

Phoenix's asset pipeline SHALL compile and serve the Dragology React code: JSX transform enabled, `dragology` + `react`/`react-dom` installed as npm dependencies, and the canvas entry included in the served bundle.

#### Scenario: Fresh checkout builds the canvas

- **WHEN** a contributor runs the standard asset build on a clean machine
- **THEN** the build succeeds and the served pages include the Dragology canvas code with no manual bundler steps

### Requirement: Phoenix seeds canvas initial state

Pages hosting the canvas SHALL deliver its initial state from the server (rendered into the page or fetched on mount), so the client never hardcodes domain state.

#### Scenario: Canvas first paint

- **WHEN** the canvas page loads
- **THEN** the Dragology island mounts with the server-provided initial state and becomes draggable without further requests

### Requirement: Client owns drag frames

Intermediate drag states (pointer-move previews, interpolations) SHALL stay in the browser and MUST NOT round-trip to the server per frame.

#### Scenario: Mid-drag rendering

- **WHEN** the user drags an element
- **THEN** all preview frames render locally at interactive frame rates with no server request per frame

### Requirement: Server persists drop states only

Only settled states (drop results, explicit saves) SHALL be sent to Phoenix for persistence or broadcast.

#### Scenario: Drop persists

- **WHEN** the user drops a dragged element
- **THEN** the final state is pushed to the server (LiveView event / fetch POST) exactly once per drop and persisted

### Requirement: React island isolated from LiveView morphs

The Dragology canvas SHALL mount inside a DOM region LiveView never patches (`phx-update="ignore"` or equivalent).

#### Scenario: LiveView update during drag session

- **WHEN** a LiveView re-render occurs while the canvas is mounted
- **THEN** the React tree and in-progress drag are undisturbed
