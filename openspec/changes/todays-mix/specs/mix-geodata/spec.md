# Spec Delta

## Purpose

Defines the static geography that gives every number a place on the map: region shapes, interconnector landings and power assets, built offline and reproducibly, with honest confidence about how precisely each item is located.

## ADDED Requirements

### Requirement: Geography is built offline, not fetched live

All map geometry SHALL be produced by a build-time pipeline and shipped as static assets. The runtime path MUST NOT depend on live map-tile servers or live OpenStreetMap query endpoints.

#### Scenario: Provider outage at demo time

- **WHEN** public map and OSM query services are unreachable
- **THEN** the map renders completely from shipped assets

### Requirement: Region shapes match the feed's region identifiers

Every distribution region identifier returned by the regional generation feed SHALL have exactly one shape, and every shape SHALL map to a feed identifier. Aggregate identifiers (nation-level rollups) MUST NOT be drawn as additional regions.

#### Scenario: Join integrity check

- **WHEN** the geodata build compares shape identifiers with the feed's region list
- **THEN** the sets match exactly, and the build fails if any region lacks a shape or any shape lacks a region

### Requirement: Interconnector landings are mapped, unknowns are surfaced

Every interconnector code present in the fuel outturn feed SHALL map to a GB landing point and a far-end country. A code that is not in the mapping MUST appear in a visible "unmapped source" list, not be dropped silently.

#### Scenario: New interconnector appears

- **WHEN** the feed contains an interconnector code absent from the mapping
- **THEN** its flow is still counted in national totals and listed as unmapped, and a diagnostic names the code

### Requirement: OSM-derived asset layer

The geography SHALL include a curated layer of power generation sites, substations and high-voltage lines derived from OpenStreetMap, simplified to a documented size budget, with the input extract date recorded.

#### Scenario: Asset layer provenance

- **WHEN** a maintainer inspects the shipped asset layer
- **THEN** it records the OSM extract date, the selection rules used, and the simplification tolerance

### Requirement: Unit location with explicit confidence

Each dispatchable unit SHALL carry a location and a confidence label: verified, matched (automatic join by name or reference), region-only, or unplaced. Units that cannot be placed MUST remain counted in region and national totals, and the map MUST show how many are unplaced.

#### Scenario: Unit cannot be matched

- **WHEN** a unit's name or reference matches no OSM site
- **THEN** it is placed at region level or listed as unplaced, its output still counts in totals, and the UI exposes the unplaced count

### Requirement: Single documented projection

All layers SHALL share one documented projection and precomputed screen coordinates, so shapes, assets and belts always align.

#### Scenario: Layer alignment check

- **WHEN** region shapes, interconnector landings and assets render together
- **THEN** every landing point lies on the coast of its region and every asset lies inside its region's outline

### Requirement: First-paint geometry budget

The geometry needed for first paint (regions, landings, national outline) SHALL stay within a documented compressed-size budget; detailed asset layers load lazily after first paint.

#### Scenario: Cold load size

- **WHEN** a production build is served over compression
- **THEN** first-paint geometry is within budget, and asset detail loads after the map is interactive

### Requirement: Reproducible geography build

The geodata build SHALL be deterministic from pinned inputs and documented commands, so a clean checkout reproduces identical outputs.

#### Scenario: Rebuild comparison

- **WHEN** the build runs twice from the same pinned inputs
- **THEN** the outputs are byte-identical

### Requirement: Map data attribution

OpenStreetMap-derived data SHALL display the attribution its licence requires wherever it is shown.

#### Scenario: Attribution on the map

- **WHEN** the asset layer is visible
- **THEN** an OpenStreetMap contributors attribution is visible on the map surface

### Requirement: Transmission boundary geometry and zones

The geography SHALL include published transmission boundary geometry and a partition of Great Britain into network zones derived from it, so every point, unit and region maps to exactly one zone.

#### Scenario: Zone coverage

- **WHEN** the geodata build runs its zone check
- **THEN** every region and every placed unit maps to exactly one zone and the check lists any that do not

### Requirement: Zone assignment for injections

Each placed unit SHALL carry its zone, and each region SHALL carry the zone shares used to allocate its embedded generation and demand, with the method documented.

#### Scenario: Region spanning zones

- **WHEN** a region overlaps two zones
- **THEN** its allocation shares across both zones are recorded, sum to one, and cite their method

### Requirement: Schematic corridor geometry

Equivalent corridors SHALL have schematic routes between zone anchors, constrained to horizontal, vertical and 45-degree segments, for the belts and overlays that draw them.

#### Scenario: Corridor route check

- **WHEN** corridor routes are validated
- **THEN** every segment is at 0, 45 or 90 degrees and every corridor joins two existing zones
