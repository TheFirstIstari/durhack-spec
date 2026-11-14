# Design

## Context

See proposal.md for motivation. This change builds on the `project-baseline` contracts: Elixir/Phoenix server, React 19 + Dragology client island, Postgres via Ecto, stateless nodes, drop-only pushes, a code-split canvas bundle, self-hosted CachyOS runners, and a Cloudflare Tunnel. It adds a product on top of them and changes none.

Everything below marked **verify** is an observation or inference that a task must confirm before it is relied on. Data and docs were probed on 2026-11-14.

### Data sources (observed)

| Source | Endpoint | What it gives | Notes |
|---|---|---|---|
| NESO Carbon Intensity | `api.carbonintensity.org.uk/regional`, `/generation` | Per-region mix (%), intensity forecast and index; national mix | Half-hour periods. Open CORS, no auth, CC BY 4.0. Rate limits not stated (**verify**). Regions 1–14 are DNO areas, 15–17 nation aggregates (**verify** list). |
| Elexon Insights | `data.elexon.co.uk/bmrs/api/v1/datasets/FUELINST` | 5-minute national outturn (MW) by fuel and by interconnector: BIOMASS, CCGT, COAL, OCGT, OIL, NUCLEAR, NPSHYD, WIND, PS, OTHER, INTELEC, INTEW, INTFR, INTGRNL, INTIFA2, INTIRL, INTNED, INTNEM, INTNSL, INTVKL | Negative values observed (PS −712, INTGRNL −341). No solar category (transmission-metered only). `no-store`, so caching is ours. About 12 hours per default response (3,020 rows). |
| Elexon Insights | `/demand/outturn/summary`, `/balancing/settlement/system-prices/{date}`, `/generation/actual/per-type` | National demand (5-minute), system prices, half-hourly generation by type | Observed. Per-type coverage **verify**. |
| Elexon Insights | `/balancing/physical?bmUnit=`, `/reference/bmunits/all` | Per-unit declared levels (PN, MELS, MILS); unit registry (names, lead party, capacity) | Registry has **no coordinates**. B1610 returned empty for settlement day 2026-11-13 period 20 (**verify**). |
| NESO open data (CKAN datastore) | `api.neso.energy/api/3/action/datastore_search[_sql]`, resource `38a18ec1-…` | Half-hourly `Flow_MW` and `Limit_MW` for 32 constraint groups (for example SCOTEX, ESTEX, SEIMP, FLOWSTH, SSHARN), 2023-01-01 to 2026-11-14 23:30, about 765,000 rows | **Day-ahead snapshot, not measured actuals.** On 2026-11-14 SCOTEX showed flow 6,460 MW against limit 2,800 MW; whether flow is pre- or post-constraint-management is unconfirmed (**verify**). |
| NESO open data | `etys-gb-transmission-system-boundaries` (GIS ZIP), DNO licence areas, grid supply point regions (GeoJSON) | Boundary geometry and area polygons | NESO Open Data Licence. Boundary-to-cut mapping **verify**. Public line impedances were not found. |
| NESO open data | thermal constraint costs, constraint breakdown, network congestion forecast and actual, interconnector capacity values | Constraint economics and capacity inputs | Stretch and calibration inputs. |
| Open-Meteo | `api.open-meteo.com/v1/forecast` | `wind_speed_100m` (km/h), `shortwave_radiation`, `cloud_cover` | 900-second interval. Multi-location **verify**. Free tier is non-commercial. |
| Overpass (OSM) | `overpass-api.de` | OSM queries | `406` without a User-Agent, `502 at capacity` with one. Not usable at runtime. |

A consistency check on the live sample: summing the FUELINST categories gave about 31.9 GW, with wind at 11,021 MW (about 34.6%); the regional feed's national wind share was 35.0%. The regional feed also reported solar at 13.9% where FUELINST has no solar key. That gap is why `mix-grid-state` requires explicit reconciliation.

### Client and server model (researched)

- **LiveView:** a LiveView renders statically over HTTP, then mounts again over a socket, and its hooks' `mounted` runs only after the server-side mount completes. State lives in a per-connection process and is rebuilt by `mount/3` on every reconnect. Events and params are client-controlled input and must be authorised again in each handler. `phx-update="ignore"` stops patching of a subtree but only reconsiders it when its `data-*` attributes change.
- **Channels:** the page signs a token (`Phoenix.Token`), the socket's `connect/3` verifies it, and `join/3` authorises a topic. One lightweight process per client per topic. Pushes get replies (acks). Delivery from server to client is at-most-once; the client buffers outgoing pushes only in memory for a short default timeout; the docs recommend a `last_seen_id`-style catch-up on join. The JS client reconnects with exponential backoff and rejoins. Default PubSub fans out through distributed Erlang between nodes. WebSocket and long-polling transports are both available.
- **Dragology v0.2.0 (MIT; peers react ^18 || ^19):** `DraggableRenderer` takes `draggable`, `state` or `initialState`, `onDropState` (once, at drop), `onDragState` (continuously, with the preview state), `width` and `height`. The render function is called speculatively on many hypothetical states, so it must be pure and cheap. State must be an object. Positioning uses SVG `transform`; identity uses `id` (unique, no slashes), never React `key`; fragments are unsupported; components inside a draggable are opaque and do not interpolate; new entity ids should be opaque and random. Nothing in the docs covers server-side rendering, so it is treated as client-only.
- **Entire (free tier):** CLI, session storage, search, mirrors, human-only trails, code hosting and local memory are free; agent usage (review agents, custom agents, summaries, memory extraction) is pay-as-you-go. Checkpoints are Git refs pushed with the code and readable by anyone who can read the repo.

## Goals / Non-Goals

**Goals:**

- A single fused, provenance-labelled grid frame that is the product's source of truth, assembled identically on server and client.
- Region-level granularity wherever the data allows, and honesty (labels, confidence, residuals) wherever it doesn't.
- What-if editing composed from accounting, physics and policy, cheap to explore and impossible to lose: immutable branching history with a livestream-style return to live.
- A private, short-lived workspace per visitor and shareable links that carry the full edit history.
- A page that paints without any socket, and a demo that survives upstream outages, rate limits and venue Wi-Fi.
- A visual system with a point of view, held to numeric budgets.

**Non-Goals:**

- AC load flow, voltage, reactive power, stability, inertia or frequency, full N-1 contingency analysis, forecasting and market or price simulation. Physics here means thermal transfer on a reduced network, and every result says so.
- Indicative redispatch cost in v1 (it would imply market accuracy the data can't back). A later change can add it as a labelled parameter.
- Merging branches, and re-applying an edit sequence onto a different base moment.
- Accounts, real-time multiplayer, moderation tooling and cross-device recovery of private workspaces (sharing is the supported way to move work).
- Server-side rendering of React, 3D or WebGL, non-GB grids, and investment advice from the original idea.

## Decisions

### Realtime and rendering

- **Dead-rendered shell, plain client entry, Channel for realtime; no LiveView on the map page.** LiveView hooks mount only after the socket connects, so a React island inside a LiveView would not paint until the websocket is up, and a blocked socket would leave a blank page. Instead a controller-rendered page embeds the latest frame and manifest as JSON (kept small), a plain script mounts the island at DOM ready, and a Phoenix Channel carries pushes and commits. This also removes any chance of LiveView patching React's tree, which satisfies the baseline's island-isolation intent trivially. The baseline spec's wording ("LiveView event / fetch POST", `phx-update="ignore"`) can be amended in a follow-up; a Channel push is an equivalent once-per-drop transport. Alternative: LiveView with `phx-update="ignore"` and a hook (rejected for first-paint dependence on the socket and per-visitor LiveView processes holding nothing the island needs).
- **No server rendering of React.** It would need a Node runtime on the deploy hosts, Dragology's SSR behaviour is undocumented, and the page has a cheap loading skeleton plus the embedded state. Revisit only for share-link preview images.
- **Channel authentication, catch-up and acks.** The page mints a short-lived `Phoenix.Token` from the verified visitor cookie (or an anonymous read-only token when there is no cookie yet); `connect/3` verifies it, `check_origin` is limited to the public domain (**verify** config), and `join/3` authorises only the holder's workspace topic. Since pushes are at-most-once, the client sends its last seen tick on join and the server replies with missed ticks or the latest frame. Commits are pushes with acks, held in a client outbox until acknowledged, and idempotent because node ids are content-addressed.
- **Cross-node fan-out through Postgres, not Erlang clustering.** After an ingest commits, the node issues `NOTIFY`; every node holds a `LISTEN` connection (via `Postgrex.Notifications`) and re-broadcasts locally with `Phoenix.PubSub`. This needs no distributed Erlang between hosts behind a tunnel and no sticky sessions. Alternative: the default distributed PubSub (needs node clustering) or a Redis adapter (another service).
- **Polling fallback and visible state.** If the Channel cannot connect, the client polls conditionally at the data cadence and says so in the connection indicator. WebSocket behaviour through the Cloudflare Tunnel is unverified (the docs I read don't address it), so a spike task tests it and the long-polling transport.
- **Gesture state and model state are separate.** Dragology holds only gesture state (a tool's position, the timeline handle, a priority list's order). The evaluator runs from `onDragState`, throttled to animation frames, and its results feed ordinary React panels. The heavy map is never inside a `DraggableRenderer`, because Dragology renders its tree speculatively many times per frame. Instead each interaction gets a small overlay SVG sized to the map viewport and hosting only its draggable tools, and drop targets are resolved by our own hit-testing (point in polygon on the pre-projected region and zone shapes). Opaque components stay out of draggable trees. A spike measures speculative-render cost with the overlay approach before the workspace is built.
- **Belts are declarative SVG; JavaScript does not run per frame.** Dash patterns animated by CSS, driven by variables (speed from magnitude, direction from sign). SVG dash animation repaints rather than compositing, so an animated-element cap and a measured frame-time test exist to catch problems early. Fallback: one Canvas 2D overlay for belt items.
- **Pan and zoom need a spike.** Whether Dragology cleanly handles background panning with semantic zoom is unconfirmed. If it falls short, ship wheel, pinch and button zoom with keyboard panning, and record the choice here.

### Data, frames and time

- **Server fuses, client scrubs, one definition of "frame at time t".** The server builds the latest frame from persisted observations; the window endpoint returns the same observations as columnar arrays at native resolution, and the client assembles any frame by as-of lookup with carry-forward and age. The logic exists twice (Elixir and JS) under one golden-fixture suite. A back-of-envelope estimate (to be measured): about 35 thousand numbers across 48 hours at native resolution, which should gzip to well under 150 KB, and storing coarse series at native resolution is what makes "native resolution preserved" true by construction.
- **Provenance is a type, not a style.** Every value carries `measured | declared | upstream-modelled | derived | scenario`; derived and scenario values carry a method id that resolves to a written definition. The map's visual encoding is a function of that field. Published day-ahead boundary values are upstream-modelled, labelled day-ahead.
- **Poll with a Postgres-backed job queue and unique jobs.** A queue (Oban is the leading candidate) keyed by series and period gives one logical poller across any number of nodes, plus retries and backoff, using the Postgres the baseline already needs; HTTP via Req. The same queue runs retention jobs. Versions are pinned at scaffold per `platform-stack`.
- **Incremental fetch, deduplicated raw storage.** Poll from the last seen publish time (**verify** the filter exists), store raw bodies compressed and content-hash deduplicated, expire after a configured retention (default 7 days). A full 12-hour FUELINST window every 5 minutes would be about 170 MB per day uncompressed.
- **Observations table plus materialised latest frame.** One narrow table with a unique key on series, spatial id and period start gives idempotent upserts; the latest frame is materialised when a poll completes and cached per node as purely derived data.
- **Time is UTC internally and UK local at the edge.** Settlement periods derive from the UK local day (48 normally, 46 on the spring change, 50 on the autumn change). The 2026 changes (29 March, 25 October) are in the past, so real upstream data for both can serve as fixtures (**verify** retention).

### Geography

- **Offline, pre-projected static assets.** A build script turns pinned inputs into simplified region paths, landings, boundary geometry, zones and an asset layer, in British National Grid. No runtime map library and no tile servers (aesthetic control, OSM tile policy, bundle size). Alternative: MapLibre with vector tiles.
- **Region shapes from official boundaries if obtainable, schematic Voronoi fallback if not.** Both keep the join check against feed ids.
- **Unit geolocation is a ranked join with a human override**, with confidence labels and a visible unplaced count. A public renewables planning dataset may improve matches (availability and licence **verify**).
- **Zones from boundary geometry.** Zones are the regions between consecutive boundary lines, so each published boundary is a cut of the zone graph. Zone shares for regions that span zones are recorded with their method.

### Model: accounting, physics and policy composed

- **Fidelity ladder.** Ledger (accounting only), boundary (net transfer across each cut compared with its limit, which needs no impedances because flow across a cut is its net injection) and flow (linearised power flow on calibrated equivalent corridors). Redispatch is a mode on top of the boundary or flow tier. Users move up the ladder as a parameter; each tier labels what it covers.
- **Accounting layer.** A snapshot ledger holds national megawatts by fuel (outturn plus embedded solar from the regional mix), interconnector flows, demand and declared unit levels. Balancing follows configured priority lists. Defaults, to be reviewed in a task: cover shortfall with interconnector headroom up to documented capacity, then dispatchable gas up to registered capacity, then report unmet demand; absorb surplus by curtailing wind and solar, then reducing imports, then storage charging, then spill. Carbon intensity is the output-weighted mean of documented per-fuel factors (the Carbon Intensity methodology's table is the leading source; **verify** it and the imports factor).
- **Regional attribution.** Regional megawatts are derived by allocating each fuel's national output across regions in proportion to regional share times regional weight (registered capacity of located units), with a capacity-weighted versus equal-weight uncertainty band. Edits on a located unit act on its declared output and skip allocation.
- **Reduced network and calibration.** Equivalent corridors join zones; the flow tier's corridor parameters are fitted so the baseline reproduces NESO's published day-ahead boundary flows over a hindcast window, with residuals and a confidence class published per boundary. Boundaries that fit poorly are labelled low-confidence and left out of headline summaries. The published series is a day-ahead snapshot, so it validates the model against NESO's view, not against metered reality. If calibration fails for a corridor the product degrades to the boundary tier for it (decision gate in the network spike).
- **Flow numerics.** Dense linear solves on a small network with a fixed pivot order in both implementations; tolerance-bound agreement rather than bit equality; the server's rounded result is authoritative and the client's is a preview. This replaces the earlier plan for exact cross-implementation equality, which floating-point solves cannot promise.
- **Redispatch is greedy and sensitivity-based.** Relieve an overloaded boundary or corridor by curtailing units with the largest effect on it, replace per the configured priority, iterate to a cap, and report converged, iteration-limited or infeasible with volumes by fuel and zone. It is deterministic and explainable, and it is the "wind paid to switch off" mechanic. Alternative: DC optimal power flow via an LP solver (better optimum, harder to match across runtimes; a stretch comparison).
- **Network edits and isolation.** Outage, derate, upgrade and (flow tier) reactance edits; an edit that islands a zone reports isolation and unmet demand rather than failing.
- **Parameters are a registry; changes are tree edits.** Every parameter has type, range, default, unit, tier and rationale; priority lists are ordered lists validated for completeness; presets set several at once; non-default values are flagged. Registry changes bump the model version, and saved nodes are migrated or flagged.
- **Observed mode uses the physics too.** Corridor flows for the current observed moment are derived values with confidence, shown next to published boundary values, which makes the product more than a dashboard even before anyone edits.

### Persistence, visitors and sharing

- **Anonymous visitor identity, created at first save.** A signed, `HttpOnly`, `Secure`, `SameSite=Lax` first-party cookie holding an opaque id of at least 128 bits, with `max_age` matching the idle retention and refreshed on activity. No cookie is set on a plain page view.
- **Workspace and nodes.** Tables for workspaces, trees, content-addressed nodes (id is the hash of parent id, canonical edit encoding and model version), and separate mutable metadata (labels, notes, pins, archive flags, last position), so renaming never changes identity. Inserts are conflict-ignoring, so identical edits dedupe and concurrent tabs produce siblings without locks. Each tree stores its compressed base snapshot (estimated 10–20 KB).
- **Evaluate on the client, verify on the server.** During a drag the client evaluates locally, with no requests. On drop it sends only the edit; the server validates, re-evaluates, stores a compact result summary (about 1.5 KB) and logs disagreements. Per-node cost is about 150 bytes for the edit plus the summary, so a 500-node tree stays well under 1 MB.
- **Sharing publishes an immutable snapshot.** A share copies the chosen nodes (path to a node, or the whole tree) with their edit history into its own record with its own unguessable id, so it outlives the workspace. Notes are excluded unless the visitor opts in. Viewing is read-only; the viewer's first edit copies the ancestry into their own workspace (creating their identity then). Revocation and clear-my-data delete immediately.
- **Short, tiered retention.** Defaults: workspace deleted after 7 idle days; shared snapshot after 30 days since last opened; both extend on use and are configured, not hard-coded. A scheduled job deletes expired data. Quotas bound trees per workspace, nodes per tree (about 500), shares per workspace and bytes. Stored data holds no IP address or device details.
- **View, back and resume are three separate pointers.** The view is observed (live or a time) or a scenario node; the back stack holds previously viewed positions; the resume pointer records the last scenario node when the user returns to live. Undo and redo move only within the tree. A first edit from an observed view creates a tree based on that view's frame, snapshotted at that instant.

### Workspace UI

- **The history graph is a stable, composition-shaped tree under the observed trunk.** The trunk shares the scrubber's time axis, and each tree hangs below its base moment (depth is edit count, siblings stack). Append-only history makes layout incremental, so existing nodes stay put. Each node's glyph is a tiny stacked bar of its own national composition. SVG with viewport virtualisation, a minimap and keyboard traversal. Alternatives: a vertical git-log lane view, or a force-directed graph (unstable, hides time order).
- **Compare reads stored summaries and shares one viewport.** Deltas are computed client-side from per-node summaries; side-by-side maps render from one projection and one shared pan and zoom state, capped at four maps.
- **Panels are docked, lazy and mode-aware.** Desktop default: map centre, timeline with the graph beneath it, inspector, model and results panels on the right, compare replacing the right dock; narrow screens use bottom sheets. Graph, compare, model and method panels are separate lazy chunks. The current mode (observed live, observed at a time, scenario with its tier) is a single piece of render state read by every panel and by the map's frame treatment.
- **CI never calls live upstreams.** Contract and golden-fixture tests run against recorded payloads on the `durhack-ci` runners; a scheduled canary probes live schemas without gating merges.
- **Replay feeds the real pipeline.** The capture tool exports raw payloads, and replay mode pushes them through the same normaliser, reconciler and frame builder as live data.
- **Fuel palette reads as composition at a glance.** Low-carbon fuels in cool hues, fossil and combustible fuels in warm hues, imports, storage and other in neutrals, each with a secondary cue (pattern or order). Palette values live in the application repo's style guide and are validated by script.

### Working method (Entire, free tier)

- **Entire is part of how this is built, within the free tier.** Hooks at each repo root for every agent used (the setup observed in `durhack-spec` has OpenCode hooks at the root, stray Claude Code and Pi hook files committed under the baseline change folder, no Claude Code hooks at the root, and no `entire` remote; `durhack` has none of it and no commits yet). Team settings are committed (automatic commit linking, PII redaction on, summaries off); personal settings stay local. Checkpoints are pushed with the code and are readable by anyone who can read the repo, so repo visibility is decided before the first push. Search, explain, why and session resume are the daily tools; review agents, summaries and memory extraction are off the table. Details live in the baseline `platform-agent-coordination` spec; each task group here lands as commits linked to checkpoints.

## Risks / Trade-offs

- [Regional mix is an upstream model, and a polished map can make it look measured] → The provenance type drives a distinct visual encoding, and drill-down always names the provider and class.
- [A physics layer can be mistaken for operational truth] → Tier badges, per-boundary confidence and residuals, a mandatory per-tier caveat, no price or operability outputs, and published calibration reports.
- [No public impedance data, and the published flows are day-ahead snapshots] → Boundary tier needs none; flow tier is calibrated and labelled; a decision gate degrades gracefully.
- [The SCOTEX sample showed flow above limit, so the dataset's meaning is unclear] → Resolve in the network spike before any calibration is built.
- [Region polygons or zone partition unavailable or awkward to license] → Voronoi fallback specified; zone check fails the build if coverage breaks.
- [Unit-to-site matching is noisy] → Confidence labels, a hand-verified override table for the biggest units, region-level fallback, and a visible unplaced count.
- [Upstream rate limits or schema changes arrive mid-event] → Backoff, staleness handling, fixtures plus a canary, and replay mode as the backstop.
- [Dragology is v0.2.0 and renders speculatively] → Pin the exact version; keep draggable trees small; separate gesture state from model state; a spike measures overlay cost.
- [SVG belt animation may not hold frame rate] → Animated-element cap, an early frame-time test, a Canvas 2D fallback.
- [WebSockets may not pass the tunnel as expected] → Spike plus long-polling transport plus conditional-polling fallback.
- [Channel pushes are at-most-once] → Catch-up on join, idempotent acknowledged commits and a client outbox.
- [Server and client evaluators or assemblers diverge] → Shared golden fixtures, tolerance-bound agreement, server authority and mismatch logging.
- [Anonymous workspaces and links can grow or be abused] → Quotas, rate limits, short retention, 128-bit ids, no personal data, clear-my-data and revoke.
- [Cookie and privacy expectations] → Cookie only at first save, opaque and signed, no personal data or tracking, and data minimisation. Decided (durhack#9): no privacy notice, cookie banner or lost-cookie warning.
- [Entire checkpoints can leak prompt content] → Decide visibility first, enable PII redaction, never paste secrets, and treat redaction as best-effort.
- [Thirteen capabilities is a great deal for a hackathon] → Phased tasks with cut lines; the minimum viable slice is the observer map, ledger-tier edits with undo and compare, and return to live.
- [Users confuse scenario with live] → The mode indicator, the scenario frame skin, first-run explanation, and observed frames carrying no scenario values.
- [A model update changes saved results] → Version stamping on every node and an explicit re-evaluate step.
- [Cloudflare Tunnel and outbound egress are demo-day dependencies] → Replay mode removes egress dependence; the tunnel risk is carried from the baseline.

## Open Questions

- Product name, palette values and the look of tiles, belts and the scenario frame skin. These are art decisions that don't change specs or tasks.
- Whether the optional constraint-cost modality ships in this change or later. It is optional in the spec and stretch in the tasks either way.
- Where the replay dataset lives (repository or release asset), to be settled once its real size is known.
- Final numeric defaults for retention, node cap and rate limits. The design assumes 7 idle days for workspaces, 30 days since last opened for shares and about 500 nodes per tree, all configurable.
