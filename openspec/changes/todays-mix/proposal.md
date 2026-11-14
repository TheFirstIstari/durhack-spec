# Proposal

## Why

Existing national grid dashboards stop at the country level: one 2D total, one fuel-mix chart, no geography, no way to ask "what if". The data to do much better is public and keyless (probed live 2026-11-14): regional generation mix and carbon intensity for the GB distribution regions, 5-minute national fuel and per-interconnector outturn, per-unit dispatch, system prices, weather, and NESO's half-hourly day-ahead flows and limits for 32 transmission constraint groups. No one fuses them onto a map, and no one lets you edit the grid, keep your alternatives and share them.

Today's Mix is that product: a multi-modal, region-granular, Factorio-style schematic of Britain's electricity, scrubbable through time, in which each visitor can compose changes (switch off a gas block, add storage, cut a corridor, reorder the balancing priority), branch their choices in a version-control-style tree, compare them side by side, and jump back to live as on a livestream. Edits are evaluated by a composition of models: an accounting layer (who generates, who covers a shortfall), a physics layer (whether it can cross the network) and a policy layer (the rules in force). That combination, and the fact that visitors compose alternatives, is how it answers the hackathon theme ("Composition").

## What Changes

- Adds a live GB grid-state product on top of the baseline stack: Phoenix ingests and fuses upstream data server-side; a React + Dragology island renders, scrubs and edits client-side.
- Ingests multiple modalities with provenance: regional mix and carbon intensity, national fuel and interconnector outturn, demand, system price, per-unit dispatch, weather, and published day-ahead boundary flows and limits (as day-ahead values, not measured actuals).
- Establishes a static geography layer: region shapes, interconnector landings, OSM-derived assets, transmission boundary geometry and a zone partition, built offline, with unit-to-location joins carrying a confidence level.
- Defines a unified time-indexed grid frame that labels every value as measured, declared, upstream-modelled, derived or scenario, and reconciles sources instead of hiding disagreements.
- Defines the map experience: schematic-geographic, Factorio-style animated belts, equivalent corridors coloured by loading, drill-down to source, a time scrubber and a replay mode that keeps the demo independent of upstream uptime.
- Adds grid editing through a layered model: a fidelity ladder (ledger, boundary transfer, calibrated DC flow), redispatch that relieves overloads, and three edit classes (grid, network, policy), with honest per-tier limits and published calibration residuals per boundary.
- Adds a registry of model parameters (balancing and curtailment priority lists, factor tables, tier and solver settings, allocation methods, presets), where every change is an ordinary, undoable, comparable edit.
- Adds a version-control tree for edits: immutable nodes, branching, undo and redo that never destroy work, jump to any node and back, one-action return to live with a resume path.
- Gives each visitor a private workspace for their own graph and history, stored for a short period with no accounts, and lets them publish shareable links that carry their changes and full edit history; viewers fork on first edit.
- Defines the realtime model: a server-rendered shell that paints without any socket, one authenticated Phoenix Channel with at-most-once pushes and explicit catch-up, acknowledged idempotent commits, cross-node fan-out through Postgres, and a polling fallback.
- Adds comparison of two to four states with deltas, edit-class attribution, parameter diffs and guards against comparing unlike bases or tiers.
- Defines the workspace: the always-visible mode indicator, the panel set, drag-and-keyboard-equivalent editing with preview before commit, shortcuts and responsive layouts.
- Declares performance, accessibility and art-direction constraints so "artsy" is a testable contract.
- Records, in the baseline agent-coordination spec and this change's tasks, how the team uses Entire's free tier while building (hooks at the repo root, committed team settings, checkpoint visibility decision, free-only boundary), including corrections to the setup state observed in the repos.

## Capabilities

### New Capabilities

- `mix-ingest`: server-side polling, normalisation, persistence, staleness handling, network reference data and offline replay of the upstream feeds, with one logical poller regardless of node count.
- `mix-geodata`: the build-time geography layer: region shapes, interconnector landings, OSM-derived assets, boundary geometry and zones, unit-to-location joins with confidence, and attribution.
- `mix-grid-state`: the fused multi-modal grid frame: modalities, provenance classes (including scenario), reconciliation, derived metrics, network section and the frame API contract.
- `mix-map`: the Factorio-style schematic-geographic map: belts, corridors, regions, drill-down to source, art direction, performance budget and accessibility.
- `mix-time-scrub`: live head versus history window, drag-to-scrub, replay mode, and correct handling of UTC, settlement periods and clock-change days.
- `mix-scenario-model`: how editing works as a composition of accounting, physics and policy layers at a chosen tier, with outputs and honesty rules.
- `mix-network-model`: the reduced-network physics layer: zones and corridors, boundary and flow tiers, calibration and confidence, redispatch, and network edits.
- `mix-model-parameters`: the parameter registry, priority lists, factor tables, presets and versioning.
- `mix-history-graph`: the version tree of scenarios: nodes, branches, undo and redo, jump and back, return to live, and the graph panel.
- `mix-visitor-state`: anonymous per-visitor workspaces, short retention, isolation, shareable snapshot links with full history, fork-on-edit, revocation and data minimisation.
- `mix-realtime`: first paint without sockets, the authenticated channel, catch-up, acknowledged commits, cross-node delivery, fallbacks and limits.
- `mix-compare`: comparing two to four states, deltas, diff views, attribution and the like-for-like guard.
- `mix-workspace`: mode indicator, panel set and layout, edit composition and preview, model and results panels, shortcuts, responsive behaviour and first-run guidance.

### Modified Capabilities

None in `openspec/specs/`. The baseline specs (`platform-stack`, `platform-runtime`, `platform-drag-integration`, `platform-delivery`) are still in the unarchived `project-baseline` change, so this change builds on their contracts without altering them. Where it leans on them (stateless nodes, durable state in Postgres, drop-only pushes, code-split canvas bundle, runner egress), the dependency is noted in design.md. The baseline's `platform-agent-coordination` spec and tasks are revised in place within `project-baseline` to record the Entire findings.

## Impact

- **This repo:** planning artifacts only (specs-only per `platform-repo-scope`).
- **Application repo (`TheFirstIstari/durhack`), when it opens:** Phoenix contexts for ingestion, frames, network model, parameters, workspaces and shares; Postgres tables (time series, raw payloads, scenario nodes, workspaces, shared snapshots) with migrations; a job queue for polling and retention; a build-time geodata pipeline; a shared evaluator implemented on server and client with golden tests; a Channel endpoint; and the React/Dragology workspace island with lazily loaded panels.
- **User data:** workspaces are the first user-created durable state. They are anonymous (a signed first-party cookie set only on first save), expire after a short idle period, hold no personal data, and are deleted on request. Shared links outlive the workspace for a longer, configured period.
- **External dependencies (observed keyless on 2026-11-14, limits unconfirmed):** NESO Carbon Intensity API, Elexon Insights (BMRS) API, Open-Meteo, NESO open data (CKAN datastore), and OpenStreetMap data. Public Overpass returned `502 at capacity`, so live Overpass queries are excluded from the runtime path.
- **Delivery:** the deploy host needs outbound HTTPS to those hosts; WebSocket behaviour through the Cloudflare Tunnel is unverified and has a spike task with a long-polling fallback; a recorded replay dataset ships with the app.
- **Dependencies on young software:** Dragology is v0.2.0 (MIT; React 18 or 19 peers), so its exact version is pinned and a spike checks the overlay and speculative-render approach.
- **Licensing and attribution:** CC BY 4.0 (Carbon Intensity), NESO Open Data Licence, Elexon terms, ODbL (OSM) and Open-Meteo terms must be honoured in-product; confirm each before the event.
- **Working method:** Entire free tier only; checkpoint visibility must be decided before checkpoints are pushed (see the baseline agent-coordination spec).
