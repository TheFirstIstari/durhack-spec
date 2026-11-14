# Spec Delta

## Purpose

Defines how users compare different choices: selecting two to four states, reading deltas with their causes, and seeing the differences on the map, without ever implying a like-for-like comparison where the bases differ.

## ADDED Requirements

### Requirement: Compare two to four states

Users SHALL be able to compare two to four states chosen from the graph, any of which may be an observed moment or a scenario node.

#### Scenario: Compare against live

- **WHEN** a user selects a scenario node and the live head
- **THEN** both appear in the compare panel, labelled by class and time

### Requirement: Like-for-like guard

When compared states differ in base moment, fidelity tier or model version, the panel SHALL say so prominently and label deltas as not like-for-like. Differing edits and parameters are the point of a comparison and are listed, not flagged. The comparison MUST still be viewable.

#### Scenario: Different bases

- **WHEN** a user compares nodes from two trees based on different moments
- **THEN** a persistent warning names both base times and deltas carry the not-like-for-like label

#### Scenario: Different tiers

- **WHEN** a user compares a ledger-tier node with a flow-tier node
- **THEN** the panel warns that the tiers differ and shows network metrics only for the state that has them

### Requirement: Metric table with deltas

The panel SHALL tabulate, for national and each region: composition shares, carbon intensity, import dependence, total generation, surplus, deficit and unmet demand and, where the tier provides them, boundary utilisation, corridor loading, overloads and redispatched volume, with absolute values and deltas against a chosen reference, sortable by largest change.

#### Scenario: Sort by impact

- **WHEN** a user sorts by absolute delta in carbon intensity
- **THEN** regions order by the size of that change, with each value showing its class

### Requirement: Reference selection and swap

Users SHALL choose which state is the reference and swap it, with all deltas inverting accordingly.

#### Scenario: Swap reference

- **WHEN** the reference is swapped from A to B
- **THEN** every delta changes sign and no absolute value changes

### Requirement: Map diff views

The map SHALL offer a delta overlay using a colour-vision-safe diverging encoding with a non-colour cue for magnitude, and a side-by-side view of up to four synchronised maps.

#### Scenario: Synchronised maps

- **WHEN** a user pans or zooms one map in side-by-side view
- **THEN** all maps pan and zoom together

### Requirement: Composition difference chart

The panel SHALL include a chart aligning each state's fuel composition using the product's fuel colours, so shifts in composition are readable at a glance.

#### Scenario: Gas removed in one branch

- **WHEN** one compared branch removes most gas
- **THEN** the chart shows that branch's gas band shrunk and the bands that replaced it grown, using the same colours as elsewhere

### Requirement: Attribute differences to edits

For compared scenario nodes the panel SHALL show their lowest common ancestor and the edits on each side since it, so every difference traces to a choice.

#### Scenario: Diverging branches

- **WHEN** nodes A and B diverged three edits ago
- **THEN** the panel lists A's edits and B's edits since the common ancestor

### Requirement: Parameter differences and class breakdown

The panel SHALL list the parameters that differ between compared states (against each other and against the defaults) and group the edits since the common ancestor by class, so a difference can be traced to a physical change, a network change or a policy choice.

#### Scenario: Same grid, different policy

- **WHEN** two branches differ only in curtailment order
- **THEN** the panel shows no grid or network edits, lists the differing priority list, and shows the resulting output deltas

### Requirement: Unmet demand is prominent

If any compared state has unmet demand, the panel SHALL flag it prominently on that state, in the table and on the map.

#### Scenario: Branch with shortfall

- **WHEN** one compared state leaves demand unmet in a region
- **THEN** the state, table row and region are flagged with the unmet megawatts

### Requirement: Shareable and accessible comparisons

A comparison SHALL be encoded in a link (tree and compared nodes). Deltas MUST NOT rely on colour alone, and screen-reader users get a summary of the largest changes and the table.

#### Scenario: Open a shared comparison

- **WHEN** someone opens a comparison link
- **THEN** the same states, reference and view appear, and a text summary of the largest changes is available
