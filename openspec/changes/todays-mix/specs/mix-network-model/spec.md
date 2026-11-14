# Spec Delta

## Purpose

Defines the physics layer: a reduced model of the GB transmission network that checks whether a dispatch can physically move across published boundaries and corridors, calibrated against published data, with redispatch to relieve overloads and strict honesty about what a reduced network can and cannot show.

## ADDED Requirements

### Requirement: Zones and equivalent corridors

The model SHALL represent Great Britain as zones joined by equivalent corridors, built so that every transmission boundary used by the product is a cut of the zone graph. The mapping from published boundary identifiers to cuts MUST be documented, and any published boundary that cannot be mapped MUST be listed as unmapped.

#### Scenario: Boundary mapping check

- **WHEN** the model is built from the published boundary list
- **THEN** each mapped boundary corresponds to a documented cut, and unmapped boundaries appear in a visible list

### Requirement: Boundary transfer tier

At the boundary tier, the model SHALL compute net transfer across each mapped boundary as the net injection on one side of its cut, and report utilisation against the boundary's limit.

#### Scenario: Scotland to England transfer

- **WHEN** zones north of the Scotland-England boundary have 8 GW of generation and 2 GW of demand
- **THEN** the boundary transfer is 6 GW and utilisation is that transfer divided by the boundary limit

### Requirement: Flow tier with calibrated corridors

At the flow tier, the model SHALL compute a linearised power flow over the equivalent corridors, reporting each corridor's flow and loading. Corridor parameters MUST be calibrated so the baseline reproduces published boundary flows, and the slack treatment MUST be documented.

#### Scenario: Baseline reproduction

- **WHEN** the flow tier runs on an observed moment inside the calibration window
- **THEN** modelled boundary flows are reported next to published flows, with residuals within the documented tolerance for boundaries marked high confidence

### Requirement: Published validation and confidence per boundary

For each boundary the product SHALL publish its calibration residual over a documented hindcast window and assign a confidence class from documented thresholds. Low-confidence boundaries MUST be labelled as such wherever their values are shown.

#### Scenario: Poorly fitting boundary

- **WHEN** a boundary's residual exceeds the low-confidence threshold
- **THEN** every display of its modelled values carries the low-confidence label and the boundary is excluded from headline summaries

### Requirement: Published values are day-ahead, not measured

The product SHALL treat published boundary flows and limits as published day-ahead values, never as measured actuals, and MUST say so wherever they are compared with model output.

#### Scenario: Comparison label

- **WHEN** modelled and published flows are shown together
- **THEN** the published value is labelled day-ahead published, with its period

### Requirement: Zone injections from declared and allocated sources

Zone injections SHALL be built from located units' declared levels, plus derived allocations of embedded generation and demand by documented methods, with unplaced contributions carried at region level. Interconnector flows SHALL enter at their landing zones with their signs.

#### Scenario: Unplaced unit

- **WHEN** a generating unit has no placed location
- **THEN** its output is included in its region's zone injections and the injection carries a wider uncertainty

### Requirement: Overload detection with security margin

The model SHALL flag any boundary or corridor whose loading exceeds its limit reduced by a configurable security margin. Full contingency analysis is not performed; users explore outages by editing.

#### Scenario: Margin applied

- **WHEN** the security margin is 10 percent and a corridor carries 95 percent of its limit
- **THEN** the corridor is flagged overloaded

### Requirement: Redispatch relieves overloads and reports honestly

When redispatch is enabled, the model SHALL relieve overloads by reducing output behind a constraint and replacing it elsewhere per the configured priority, iterating up to a limit. It MUST report volumes by fuel and zone, and finish as converged, iteration-limited or infeasible, never hiding residual overload.

#### Scenario: Wind behind a constraint

- **WHEN** wind generation behind an overloaded boundary is curtailed and gas ahead of it replaces it
- **THEN** the result lists curtailed and replacing megawatts by fuel and zone, the new boundary loading, and a converged status

#### Scenario: Cannot be relieved

- **WHEN** no permitted redispatch can relieve an overload
- **THEN** the result is infeasible, the remaining overload is reported, and nothing is silently clipped

### Requirement: Network edits

The model SHALL accept network edits: outage or derate of a corridor or boundary, increase of corridor capacity, and (at the flow tier) change of a corridor's reactance. An edit that isolates a zone MUST be reported as isolation with the affected unmet demand rather than failing.

#### Scenario: Corridor outage isolates a zone

- **WHEN** an outage disconnects a zone from the rest of the network
- **THEN** the result reports the zone as isolated, shows its unmet demand and curtailed generation, and still evaluates the rest of the network

### Requirement: Derived corridor flows in observed mode

In observed mode the product SHALL show modelled corridor flows as derived values with their confidence class and method, never as measured, alongside published boundary values where available.

#### Scenario: Observed corridor belt

- **WHEN** the map shows a corridor for an observed moment
- **THEN** its flow is classed derived, carries its confidence, and links to its method and calibration report

### Requirement: Numerical agreement and authority

The server's evaluation SHALL be authoritative. The client's evaluation is a preview and MUST agree with the server within a documented tolerance on shared golden cases. Disagreements beyond tolerance MUST be logged and the server's values displayed after commit.

#### Scenario: Preview versus commit

- **WHEN** a committed edit's server result differs from the client preview within tolerance
- **THEN** the server's rounded values replace the preview with no visible jump beyond the documented rounding

### Requirement: Interactive evaluation budget

Evaluation at each tier SHALL complete within a documented budget on a mid-range laptop at the largest supported network, fast enough to preview during a drag at interactive frame rates.

#### Scenario: Flow tier during a drag

- **WHEN** a user drags an edit tool with the flow tier selected
- **THEN** previews update at the documented rate without dropping the map below its frame budget
