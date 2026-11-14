# Spec Delta

## Purpose

Defines how editing the grid works as a composition of three layers: an accounting layer (who generates and who covers a shortfall), a physics layer (whether it can be moved across the network), and a policy layer (the rules and parameters in force), evaluated at a chosen fidelity tier with strict honesty about what each tier shows.

## ADDED Requirements

### Requirement: Scenarios start from a real frame

Every scenario SHALL be based on exactly one observed frame (the live head or a historical moment), captured as a self-contained snapshot so the scenario remains valid after that moment leaves the history window.

#### Scenario: Base outlives retention

- **WHEN** a scenario based on a moment older than the history window is reopened
- **THEN** it evaluates against its stored base snapshot and results are unchanged

### Requirement: Layered composition at a chosen tier

Evaluation SHALL compose layers in a documented order: the accounting layer under the configured policy, then, if the tier requires, the physics layer (see `mix-network-model`), then redispatch if enabled, repeating until the loop ends. The tiers are ledger (accounting only), boundary and flow.

#### Scenario: Ledger tier

- **WHEN** a scenario is evaluated at the ledger tier
- **THEN** no network outputs are produced, and results are labelled as ledger-tier accounting

### Requirement: Three classes of edit

Edits SHALL belong to one of three classes, shown with every edit: grid edits (scale or switch off a fuel's output in a region or a located unit's output; add synthetic capacity or storage in a region; scale national demand; override an interconnector within its capacity), network edits (defined in `mix-network-model`) and policy edits (parameters defined in `mix-model-parameters`).

#### Scenario: Edit class shown

- **WHEN** a user views a node's change log
- **THEN** each edit is labelled grid, network or policy

### Requirement: Invalid edits are rejected

Each edit has typed parameters with valid ranges, and an invalid edit MUST be rejected with a stated reason, leaving state unchanged.

#### Scenario: Out-of-range interconnector override

- **WHEN** a user sets an interconnector override above its documented capacity
- **THEN** the edit is rejected and the reason names the limit

### Requirement: Deterministic, tolerance-bound evaluation

For the same base snapshot, ordered edits, parameters and model version, evaluation SHALL give the same results wherever it runs, with the server authoritative and the client preview agreeing within a documented numerical tolerance and rounding.

#### Scenario: Cross-implementation agreement

- **WHEN** the shared golden scenarios are evaluated by the server and the client
- **THEN** every output agrees within the documented tolerance, and committed results use the server's rounded values

### Requirement: Transparent, configurable balancing

The accounting layer SHALL resolve supply against demand by the configured priority orders and report what covered any shortfall and where any surplus went. Shortfall that nothing covers MUST be reported as unmet demand, never clipped or hidden.

#### Scenario: Removing a large gas block

- **WHEN** a user switches off more gas than imports and remaining dispatchable plant can replace
- **THEN** the result lists the replacement by source and shows the remainder as unmet demand in megawatts

### Requirement: Defined outputs by tier

Every evaluation SHALL output composition by fuel (national and per region), carbon intensity (national and per region) from the active factor table, import dependence, and surplus, deficit and unmet demand. At the boundary and flow tiers it SHALL also output boundary transfers and utilisation, overloads and redispatch volumes, and at the flow tier corridor flows. Each output names the tier that produced it.

#### Scenario: Output inventory

- **WHEN** a flow-tier result is inspected
- **THEN** every listed output is present with units, and each states its tier and the identifier of the factor table used

### Requirement: Scenario results are labelled and segregated

Every scenario value SHALL carry the scenario provenance class, the method identifier, the fidelity tier and the parameter set in force, and MUST NOT appear in any frame presented as observed.

#### Scenario: Mixed-up views

- **WHEN** a scenario node and the live frame are open in the same session
- **THEN** each value is classed observed-type or scenario, and no view mixes the two classes without labels

### Requirement: Honest limits stated per tier

Wherever results are shown, the product SHALL state what the tier does and does not cover: the ledger tier ignores the network entirely; the boundary and flow tiers check thermal transfer on a reduced network and ignore losses, voltage, reactive power, stability, inertia and full contingency analysis. Results MUST NOT claim the edited grid is operable.

#### Scenario: Caveat visibility

- **WHEN** a scenario result is shown anywhere in the product
- **THEN** the caveat for its tier is visible or one interaction away, with a link to the full method

### Requirement: Declared regional attribution

Region-level edits SHALL act on a derived regional allocation of national output, with its method and uncertainty stated. Edits on a located unit act on that unit's declared output and carry the unit's location confidence.

#### Scenario: Region versus unit edit

- **WHEN** a user switches off gas in a region and separately switches off one located gas unit
- **THEN** the first result is labelled with the allocation method and uncertainty, and the second with the unit's declared output and location confidence

### Requirement: Model version recorded

Each evaluation SHALL record the model version, which covers the evaluator, factor tables and parameter registry, and any change to them MUST increment it.

#### Scenario: Model change

- **WHEN** the factor table changes
- **THEN** the model version increments and previously stored scenarios still identify the version they were evaluated with

### Requirement: Human-readable edit descriptions

Each edit and result SHALL have a generated plain-language description derived from its parameters, for use in change logs and screen-reader announcements.

#### Scenario: Describe an edit

- **WHEN** an edit scales region gas output to 40 percent
- **THEN** its description states the fuel, region and resulting scale in plain language
