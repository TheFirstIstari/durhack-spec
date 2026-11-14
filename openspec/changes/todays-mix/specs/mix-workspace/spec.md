# Spec Delta

## Purpose

Defines the workspace around the map: the always-visible mode indicator, the panel set and its arrangement, how edits are composed and committed, keyboard and responsive behaviour, and honest status for saving, so exploring what-ifs stays clear and safe.

## ADDED Requirements

### Requirement: Mode is always visible

A persistent indicator SHALL show the current mode: observed live, observed at a time, or scenario (with node label, base time, how far behind live and the fidelity tier). Scenario mode MUST also apply a distinct frame treatment to the map so it cannot be mistaken for observed data.

#### Scenario: Entering a scenario

- **WHEN** the user makes a first edit from the live view
- **THEN** the indicator changes to scenario with the base time, and the map's frame treatment changes

### Requirement: Defined panel set

The workspace SHALL provide panels for the map, observed timeline, history graph, edit palette, inspector, node details, model parameters, network results, compare, and method and sources. Panels open, close and dock independently, and layout is restored on return as a per-viewer preference.

#### Scenario: Toggle panels

- **WHEN** a user closes the graph and opens compare
- **THEN** the map remains interactive, and the arrangement is restored on the next visit

### Requirement: Compose edits by dragging

The edit palette SHALL offer draggable edit tools that the user drops on regions, fuels, corridors or located units. During a drag, the map and a result strip update live from local evaluation. Only the drop commits, creating exactly one node, and Escape cancels with no node created.

#### Scenario: Drag and cancel

- **WHEN** a user drags a capacity tool over a region, then presses Escape
- **THEN** the preview disappears, no node is created and no request is sent

#### Scenario: Drag and drop

- **WHEN** the user drops the tool on the region
- **THEN** one node is created and saved once

### Requirement: Inspector with precise parameters

Selecting an entity SHALL open the inspector showing its values with provenance and the edits available, with numeric inputs and sliders for exact parameters. Every edit achievable by dragging MUST be achievable without dragging.

#### Scenario: Keyboard edit

- **WHEN** a keyboard-only user selects a region and enters a value for a fuel scale
- **THEN** the same node is created as by dragging, and the change is announced

### Requirement: Preview before commit

A pending edit SHALL display its effect (map ghosting and a result strip with deltas) before it is committed.

#### Scenario: Preview strip

- **WHEN** an edit is pending
- **THEN** the strip shows composition, carbon intensity and unmet demand deltas against the current node

### Requirement: Discoverable shortcuts

Undo, redo, jump to live, back, resume scenario and toggle compare SHALL have keyboard shortcuts listed in a cheat sheet, without conflicting with browser or operating-system shortcuts. State changes MUST be announced to assistive technology.

#### Scenario: Shortcut cheat sheet

- **WHEN** the user opens the cheat sheet
- **THEN** every shortcut is listed and each works as described

### Requirement: Responsive panels

On wide screens panels dock side by side. On narrow screens the map remains primary and the graph, compare and inspector appear as bottom sheets. Edit, undo, return to live and compare MUST remain reachable at 390 pixels width.

#### Scenario: Phone width

- **WHEN** the viewport is 390 pixels wide
- **THEN** a user can make an edit, undo it, open compare and return to live without horizontal page scrolling

### Requirement: First-run guidance

First use SHALL explain scenario mode on the first edit and offer the method panel. The empty graph MUST explain how to start.

#### Scenario: First edit

- **WHEN** a first-time user commits an edit
- **THEN** a short explanation of scenario mode and the caveat for the active fidelity tier appears once

### Requirement: Method, caveats and sources panel

A panel SHALL document the scenario model's assumptions, priority order, factor tables, version and caveats, with source attribution, and be reachable from the mode indicator.

#### Scenario: From the banner

- **WHEN** the user activates the mode indicator
- **THEN** the method panel opens at the current model version

### Requirement: Drag cost is independent of model and map size

The cost of rendering a drag SHALL not grow with the size of the map, the history graph or the network model. Model evaluation during a drag MUST run outside the drag's rendering path.

#### Scenario: Heavy workspace

- **WHEN** a drag starts with the full map, a 200-node graph and the flow tier active
- **THEN** drag frames stay within the interaction budget and the evaluation runs at its own documented rate

### Requirement: Model panel with reorderable priority

The model panel SHALL show all parameters (see `mix-model-parameters`) grouped by tier, mark non-default values, and let the user reorder each priority list by dragging, with the same precise numeric and keyboard alternatives as other edits.

#### Scenario: Reorder by keyboard

- **WHEN** a keyboard-only user moves a fuel up in a priority list
- **THEN** the same edit results as by dragging and the new order is announced

### Requirement: Results panel with fidelity and confidence

The results panel SHALL show the active tier's outputs with its fidelity badge, the redispatch status (converged, iteration-limited or infeasible), overloads, and each boundary's confidence class and residual against the day-ahead published value.

#### Scenario: Low-confidence boundary

- **WHEN** a boundary has low confidence
- **THEN** the results panel marks it, shows its residual, and links to the calibration report

### Requirement: Workspace and sharing controls

The workspace SHALL show when the visitor's workspace expires, and offer share (with scope and notes opt-in), revoke, and clear-my-data actions with confirmations.

#### Scenario: Share a branch

- **WHEN** a visitor chooses share and picks a branch
- **THEN** they are shown what will be included, a link is created, and its retention period is stated

### Requirement: Panels load on demand

Graph, compare and method panels SHALL load on demand without blocking the map, and their opening MUST NOT push interaction below the frame budget.

#### Scenario: Open compare under animation

- **WHEN** compare opens while belts animate
- **THEN** the map stays within the frame budget
