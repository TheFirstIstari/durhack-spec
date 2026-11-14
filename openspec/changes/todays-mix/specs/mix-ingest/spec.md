# Spec Delta

## Purpose

Defines how live upstream grid data reaches the product: collected server-side, normalised, persisted with provenance, resilient to upstream failure, and replayable offline so a demo never depends on third-party uptime.

## ADDED Requirements

### Requirement: Server-side ingestion only

The server SHALL be the only component that calls upstream data providers. Browsers MUST NOT call them directly; they consume frames from the app (see `mix-grid-state`).

#### Scenario: Browser traffic audit

- **WHEN** a page is loaded with the browser network panel open
- **THEN** no request goes to an upstream provider host, and all grid data arrives from the app's own origin

### Requirement: Modality coverage and cadence

Ingestion SHALL cover these series at or faster than the stated cadence: regional generation mix and carbon intensity for all distribution regions (30 minutes), national mix (30 minutes), national fuel and per-interconnector outturn (5 minutes), national demand (5 minutes), system price (per settlement period), and weather at each region centroid (15 minutes).

#### Scenario: Fresh data after one cadence

- **WHEN** an upstream source publishes a new period
- **THEN** the new observation is persisted and visible in the latest frame within one cadence interval plus a documented grace period

### Requirement: Single logical poller across nodes

Each upstream series SHALL be fetched by exactly one worker per period regardless of how many app nodes run, so scaling out never multiplies upstream requests.

#### Scenario: Second node joins

- **WHEN** a second identical node starts behind the tunnel
- **THEN** upstream request volume per period is unchanged and both nodes serve the same frames

### Requirement: Idempotent, provenance-carrying persistence

Every persisted observation SHALL record its source, series, spatial key (region, link or national), period start, resolution, value, unit, provenance class, upstream publish time and fetch time. Re-fetching the same period MUST update, never duplicate, the observation.

#### Scenario: Repeated fetch of one period

- **WHEN** the same period is fetched twice (poll overlap or backfill)
- **THEN** exactly one observation exists for that key, holding the latest values and fetch time

### Requirement: Raw payload retention for replay and audit

The system SHALL retain the raw upstream payload behind each observation for a configured retention window, so any displayed value can be traced to the bytes it came from.

#### Scenario: Trace a number to its source

- **WHEN** a maintainer inspects an observation inside the retention window
- **THEN** the raw upstream payload that produced it is retrievable

### Requirement: Gap backfill

On startup and after any outage, ingestion SHALL backfill missing periods across the history window (at least 48 hours) without duplicating existing observations.

#### Scenario: Restart after downtime

- **WHEN** the app restarts after being down for several hours
- **THEN** the missing periods are fetched and the history window has no gaps for periods the upstream still serves

### Requirement: Isolated, honest failure handling

A failing source SHALL NOT block other sources. After upstream errors or rate-limiting, ingestion MUST back off, keep the last good value flagged stale with its age, and never fabricate, interpolate or carry forward a value as fresh.

#### Scenario: One provider down

- **WHEN** one upstream provider returns errors for a sustained period
- **THEN** modalities from other providers keep updating, and the affected modality is served as stale with its age

### Requirement: Polite, anonymous client identity

Ingestion SHALL respect documented rate limits and send a generic application identifier to providers, with no personal or contact details from contributors.

#### Scenario: Provider asks for client identity

- **WHEN** a provider requires or recommends a client identifier
- **THEN** a generic project-level identifier is used and no individual's name or email is transmitted

### Requirement: Offline replay dataset

The project SHALL ship a recorded capture of all modalities covering at least 48 hours, and a replay mode that serves it through the same frame contract with zero upstream traffic.

#### Scenario: Demo with the network cut

- **WHEN** replay mode is enabled and outbound access to providers is blocked
- **THEN** the full product works, scrubbable across the captured window, and is visibly marked as replay

### Requirement: Network reference data

Ingestion SHALL also collect published half-hourly boundary flows and limits and interconnector capacity values, treating the boundary flows as day-ahead published values (not measured actuals), with enough history for calibration and validation and the same staleness and provenance handling as other series.

#### Scenario: Day-ahead values stored as such

- **WHEN** a boundary flow and limit are ingested
- **THEN** they are stored with the upstream-modelled class, their period and the label day-ahead published

### Requirement: Source attribution preserved

Each source's attribution and licence text SHALL be stored with its data and surfaced wherever its values are shown.

#### Scenario: Attribution visible

- **WHEN** a user opens the data sources panel
- **THEN** every active source is listed with its licence and attribution as the provider requires
