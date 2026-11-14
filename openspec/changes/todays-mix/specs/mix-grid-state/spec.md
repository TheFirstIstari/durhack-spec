# Spec Delta

## Purpose

Defines the fused, multi-modal grid frame that the map consumes: what each frame contains, how every value declares where it came from, how sources are reconciled instead of blended, and the contract for retrieving frames.

## ADDED Requirements

### Requirement: Unified time-indexed grid frame

The system SHALL expose the grid as frames, each representing one instant and containing a national section, a section per distribution region, and a section per interconnector link, populated from every available modality.

#### Scenario: Frame shape

- **WHEN** the latest frame is requested
- **THEN** it contains national, per-region and per-link sections, each value with a unit, and an index of which modalities are present, stale or missing

### Requirement: Modalities carried in each frame

Frames SHALL carry: supply composition by fuel (national and per region), carbon intensity (per region), interconnector flow (per link), demand (national), system price, weather (per region), and declared unit dispatch. Balancing-action and constraint-cost signals are an optional additional modality.

#### Scenario: Modality inventory

- **WHEN** the frame schema is inspected
- **THEN** each listed modality has a defined location in the frame, a unit, and a native resolution

### Requirement: Provenance class on every value

Every value SHALL be tagged measured, declared, upstream-modelled, derived or scenario. A derived or scenario value MUST name the method that produced it. No value is presented without a class.

#### Scenario: Inspecting a derived number

- **WHEN** a consumer reads a derived value
- **THEN** it carries the class derived and a method identifier that resolves to a written definition

### Requirement: Scenario frames are segregated from observed frames

Frames produced by the scenario model (see `mix-scenario-model`) SHALL be marked as scenario frames with a reference to their base frame, and observed frames MUST NOT contain scenario values.

#### Scenario: Observed frame audit

- **WHEN** the latest observed frame is inspected after scenarios have been created
- **THEN** it contains no value of class scenario, and every scenario frame names its base frame

### Requirement: Missing is not zero

Absent data SHALL be represented as absent, never as zero. A fuel genuinely reported as zero MUST be distinguishable from a fuel not reported.

#### Scenario: Source omits a fuel

- **WHEN** a source returns no entry for a fuel in a period
- **THEN** that fuel is marked absent for the period, not rendered as 0

### Requirement: Native resolution preserved

Where modalities have different resolutions, a coarser value SHALL be carried across finer ticks flagged with its native period and age. The frame MUST NOT present half-hourly data as five-minute data.

#### Scenario: Five-minute frame with half-hourly regional data

- **WHEN** a frame is built at a five-minute tick between two half-hourly regional publications
- **THEN** the regional values carry their half-hour period and age, and consumers can tell they have not changed since publication

### Requirement: Reconcile sources, never silently blend

The system SHALL compare overlapping sources (for example, national fuel outturn against the aggregate of regional mixes) and report discrepancies beyond a documented tolerance. Known structural differences, such as embedded solar missing from transmission-metered outturn, MUST be documented and surfaced rather than averaged away.

#### Scenario: Solar missing from outturn

- **WHEN** the national outturn feed has no solar category but the regional mix reports solar
- **THEN** the frame attributes solar to the regional source, marks it upstream-modelled, and records that outturn totals exclude embedded generation

#### Scenario: Unexplained disagreement

- **WHEN** two sources disagree beyond tolerance with no documented explanation
- **THEN** the frame flags the discrepancy with both values and publishes neither as the reconciled figure

### Requirement: Regional shares are not megawatts

Regional generation mix SHALL be exposed as shares. A megawatt figure for a region MUST be a derived value with a stated method and uncertainty, and never inherit the measured class.

#### Scenario: Region drill-down

- **WHEN** a user views a region's composition
- **THEN** shares are shown as upstream-modelled, and any megawatt figure is labelled derived with its method

### Requirement: Derived cross-modal metrics

The frame SHALL provide at least: each region's carbon intensity relative to the national figure; weather-implied versus reported wind and solar share per region as a plausibility signal; and net interconnector import or export as a share of supply. Each carries its method identifier.

#### Scenario: Wind plausibility signal

- **WHEN** a region reports a high wind share while weather shows low wind speed
- **THEN** the frame exposes the mismatch as a derived signal with its method, without altering the reported share

### Requirement: Signed flows with defined direction

Interconnector flow and storage charging SHALL keep their sign, with one documented convention for import, export, charging and discharging, applied identically across the frame and the UI.

#### Scenario: Exporting interconnector

- **WHEN** a link is exporting from Great Britain
- **THEN** its value is negative per the documented convention and renders as flow away from the coast

### Requirement: Frame integrity checks

Each frame SHALL pass automated checks before it is published as clean: regional shares sum to 100 within a documented tolerance, units are consistent, periods align, and required identifiers are present. A failing frame MUST be published flagged, never as clean.

#### Scenario: Shares do not sum

- **WHEN** a region's shares sum outside tolerance
- **THEN** the frame is flagged with the failing check and region, and the UI shows the region as degraded

### Requirement: Frame retrieval contract

The application SHALL provide the latest frame and frames over a time window through a versioned, documented contract with conditional-request support, and SHALL notify connected clients once per newly ingested frame rather than per render.

#### Scenario: Unchanged frame request

- **WHEN** a client re-requests the latest frame and nothing new has been ingested
- **THEN** the server answers that the content is unchanged without resending the frame

#### Scenario: New frame ingested

- **WHEN** ingestion lands a new frame
- **THEN** each connected client is notified exactly once and can fetch the new frame

### Requirement: Network section

Frames SHALL include a network section with, per mapped boundary, the published day-ahead flow and limit (class upstream-modelled, labelled day-ahead) and the model's derived transfer, utilisation, residual and confidence class, plus modelled corridor flows. Missing published values MUST be absent, not zero.

#### Scenario: Boundary without published data

- **WHEN** no published value exists for a boundary and period
- **THEN** the frame carries the modelled transfer with its confidence and marks the published value absent
