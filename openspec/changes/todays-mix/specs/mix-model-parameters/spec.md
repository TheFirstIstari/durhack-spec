# Spec Delta

## Purpose

Defines the model's tunable parameters (balancing priority, factors, solver and fidelity settings, allocation methods) as a documented registry whose changes are ordinary, comparable, undoable edits in the version tree.

## ADDED Requirements

### Requirement: Documented parameter registry

Every tunable parameter SHALL be registered with an identifier, type, allowed range or values, default, unit, description, the tier it affects, and the source or rationale for its default.

#### Scenario: Inspect a parameter

- **WHEN** a user opens a parameter's details
- **THEN** they see its range, default, unit, the tier it affects and where the default comes from

### Requirement: Parameter changes are tree edits

Changing a parameter SHALL create a node in the history tree like any other edit, be validated against the registry, and be undoable, branchable and comparable.

#### Scenario: Branch by policy

- **WHEN** a user changes the curtailment order and then returns to the parent node to change the security margin instead
- **THEN** two sibling branches exist, each with one parameter edit

### Requirement: Out-of-range values are rejected

A parameter value outside its allowed range or set SHALL be rejected with the reason and the valid range, leaving state unchanged.

#### Scenario: Margin out of range

- **WHEN** a user sets the security margin above its maximum
- **THEN** the edit is refused and the message states the allowed range

### Requirement: Reorderable balancing priority

The dispatch order for covering shortfall, the curtailment order for absorbing surplus and the redispatch order SHALL each be an ordered list of fuels or categories the user can reorder. A list that omits or duplicates an entry MUST be rejected.

#### Scenario: Wind-last curtailment

- **WHEN** a user moves wind to the end of the curtailment order
- **THEN** the next evaluation curtails other categories first and the change log records the new order

### Requirement: Factor tables are editable and sourced

Per-fuel carbon factors and availability or derate factors SHALL be editable, display their default source, and be flagged wherever a non-default value is in use.

#### Scenario: Non-default carbon factor

- **WHEN** a user changes a fuel's carbon factor
- **THEN** results using it are labelled as using a non-default factor and the default remains one action away

### Requirement: Fidelity and solver settings

The user SHALL be able to choose the fidelity tier (ledger, boundary or flow), the security margin, the redispatch mode and its iteration limit, and the regional allocation method, each as a registered parameter.

#### Scenario: Move up the ladder

- **WHEN** a user changes the tier from ledger to boundary
- **THEN** boundary results appear and the mode indicator shows the new tier

### Requirement: Presets

The product SHALL ship named presets that set several parameters at once, show how each differs from the defaults, and apply as a single edit. Users SHALL be able to save their current parameters as a named preset in their workspace.

#### Scenario: Apply a preset

- **WHEN** a user applies a preset
- **THEN** one node is created, the changed parameters are listed, and undo restores the previous values

### Requirement: Non-default values are always visible

The workspace SHALL mark every parameter that differs from its default and offer reset per parameter and reset all.

#### Scenario: Reset one parameter

- **WHEN** a user resets a single non-default parameter
- **THEN** that parameter returns to its default as one edit and others are unchanged

### Requirement: Parameter changes preview their effect

Before a parameter change is committed, the workspace SHALL show its effect on the key outputs against the current node, where evaluation cost allows.

#### Scenario: Preview a priority change

- **WHEN** a user drags a fuel within the priority list
- **THEN** the result strip shows output deltas for the pending order and commits only on drop

### Requirement: Registry is versioned with the model

Any change to the registry (new, removed, renamed or re-ranged parameter, or changed default) SHALL increment the model version. Saved nodes that reference changed parameters MUST be migrated or flagged, never silently reinterpreted.

#### Scenario: Parameter renamed in a new version

- **WHEN** a saved tree references a parameter that was renamed
- **THEN** it is migrated with a visible notice, or flagged as needing re-evaluation if no safe migration exists
