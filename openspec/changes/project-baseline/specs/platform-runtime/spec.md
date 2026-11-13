# Spec Delta

## Purpose

The runtime baseline bounds how the system grows and where state lives: stateless nodes that scale both directions, one durable store, and production builds that don't waste server or bandwidth.

## ADDED Requirements

### Requirement: Horizontally scalable nodes

The app SHALL run as stateless nodes behind the tunnel, so additional nodes increase capacity with no code change. No node-local state SHALL be required for correctness.

#### Scenario: Add a node under load

- **WHEN** traffic exceeds one node's capacity and a second identical node is started behind the tunnel
- **THEN** both nodes serve traffic correctly and no user-visible state is lost or duplicated

### Requirement: Vertically scalable nodes

The app SHALL benefit from additional CPU/RAM on a single node (BEAM schedulers, no hard-coded worker counts that cap a bigger machine).

#### Scenario: Bigger machine, more headroom

- **WHEN** the deploy host is given more CPU/RAM with the same release
- **THEN** throughput increases without reconfiguration or code changes

### Requirement: Persistent data storage

All durable domain state SHALL live in Postgres via Ecto, with migrations checked into the repo. No durable state SHALL live only in memory, on local disk, or in the tunnel.

#### Scenario: Restart preserves data

- **WHEN** the app is stopped and restarted (or redeployed)
- **THEN** all previously persisted domain state is intact and served correctly

#### Scenario: Schema change ships safely

- **WHEN** a release includes a migration
- **THEN** deploy runs it exactly once and the app boots against the migrated schema

### Requirement: Production efficiency levers

Production builds SHALL use the standard Phoenix efficiency baseline: `mix release`, digested/compressed static assets, code-split canvas bundle loaded only on canvas pages, and no server round-trips for per-frame drag previews (per `platform-drag-integration`).

#### Scenario: Production asset audit

- **WHEN** the release is built for production
- **THEN** static assets are digested and compressed, the Dragology bundle is absent from non-canvas pages, and drag previews generate zero server traffic per frame
