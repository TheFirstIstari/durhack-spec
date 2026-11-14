# Tasks — Lynx (this repo is specs-only; every task below executes in the application repo `TheFirstIstari/durhack`)

Phases are ordered so each ends demoable. Groups 1 to 8 are the observer product (live, regional, scrubbable, realtime). Groups 9 to 15 are the what-if stack and have their own minimum viable slice: ledger-tier edits, undo and redo, return to live, two-way compare, and a saved visitor workspace. Cut lines, in order of expendability: group 19 (stretch); the flow tier and redispatch inside group 11; sharing, archive, notes and three- to four-way compare inside groups 12, 14 and 15; group 16 (depth); the canary in group 18.

Working method for every group: land the work as small commits linked to Entire checkpoints (see the baseline `platform-agent-coordination` spec), using free features only. Never run `entire review`, trail review agents, `entire checkpoint explain --generate`, summaries or memory extraction.

## 1. Preconditions and spikes

- [ ] 1.1 Confirm baseline phases 2 and 3 (scaffold, drag island) and baseline tasks 1.3 to 1.10 (Entire hooks at both repo roots, team settings, checkpoint visibility decision) are done; verify `mix assets.build` passes via mise, the toggle demo drags with no per-frame requests, and the first app commit carries an `Entire-Checkpoint` trailer
- [ ] 1.2 Resolve every **verify** item in design.md (rate limits, auth, licence and attribution wording per source, FUELINST incremental fetch parameter, B1610 shape, Open-Meteo multi-location, region id list, DNO boundary source, carbon factor table, interconnector capacities, `check_origin` configuration) and record results in `docs/sources.md`; verify by re-reading design.md and finding no unresolved **verify** marker
- [ ] 1.3 Spike Dragology: background pan with semantic zoom, and the overlay-SVG approach for edit tools over the map (hit-testing by point in polygon), measuring speculative-render cost with the full region shapes and a 200-node graph on screen; verify by a recorded decision and a measured frame-time table added to design.md
- [ ] 1.4 Spike belt animation: 60 dash-animated SVG paths over the real region outline; verify by a measured frame-time table on the slowest team laptop, which sets the animated-element cap and the Canvas 2D fallback decision
- [ ] 1.5 Network data spike: load the ETYS boundary geometry and the 32 constraint groups from the NESO flows-and-limits dataset, settle what `Flow_MW` means relative to `Limit_MW` (explain the SCOTEX 6,460 versus 2,800 sample), map groups to cuts, sketch the zone graph, and test the calibration fit on a week of data; verify by a written decision in design.md (ship the flow tier, or boundary tier only) with the fit residuals
- [ ] 1.6 Realtime spike: run a Phoenix Channel through the Cloudflare Tunnel with WebSocket and with long-polling, measuring idle disconnects and reconnect behaviour; verify by a recorded result and the chosen fallback settings in `docs/realtime.md`
- [ ] 1.7 Confirm the cookie notice wording and the checkpoint visibility decision with the team; verify by the agreed notice text recorded in `docs/privacy.md`

## 2. Time handling foundation

- [ ] 2.1 Implement UTC to UK-local conversion and settlement period derivation; verify by tests asserting 46 periods on 2026-03-29, 50 on 2026-10-25 and 48 on an ordinary day
- [ ] 2.2 Add fixtures from the real 2026 clock-change days (if upstream retention allows, otherwise synthetic); verify tests pass and the repeated local hour maps to two distinct UTC periods
- [ ] 2.3 Document the time conventions in `docs/time.md`; verify the examples in the doc run as written

## 3. Ingestion core (`mix-ingest`)

- [ ] 3.1 Add migrations for observations (unique on series, spatial id, period start) and compressed, content-hash-deduplicated raw payloads; verify migrate-up runs once and an upsert test shows no duplicates
- [ ] 3.2 Record fixtures for every in-scope source and write a normaliser per source (regional mix and intensity, national mix, fuel and interconnector outturn, demand, system price, weather); verify fixture tests assert units, signs, absent-versus-zero and provenance class
- [ ] 3.3 Implement pollers as unique jobs with cadence per source and incremental fetch; verify a two-node test shows one upstream fetch per series per period
- [ ] 3.4 Implement backoff and staleness (last good retained, flagged with age, never carried as fresh); verify tests injecting 429, 5xx and timeouts leave other sources updating
- [ ] 3.5 Implement gap backfill over at least 48 hours; verify a restart-after-downtime test ends with no gaps and no duplicates
- [ ] 3.6 Send a generic client identifier and honour documented limits; verify by inspecting an outbound request and finding no personal or contact details
- [ ] 3.7 Ingest the published boundary flows, limits and interconnector capacities as day-ahead published values with history for calibration; verify a fixture stores them with the upstream-modelled class and the day-ahead label
- [ ] 3.8 Write the ingestion runbook (sources, cadences, failure modes) in `docs/ingest.md`; verify a teammate can follow it to start and observe a poll on a clean machine

## 4. Geography pipeline (`mix-geodata`)

- [ ] 4.1 Build region shapes from the best available boundary source, with the Voronoi fallback behind a flag; verify the join check fails on a deliberately removed region and passes on the full set
- [ ] 4.2 Map every interconnector code in the feed to a landing point and far-end country, with an unmapped-source list; verify a synthetic unknown code appears in the list and still counts in national totals
- [ ] 4.3 Project all layers with the documented projection and precompute screen coordinates; verify the alignment check (landings on coast, assets inside regions) passes
- [ ] 4.4 Extract the OSM-derived asset layer from a pinned extract with recorded date, selection rules and simplification tolerance; verify the output records them and respects the lazy-load size budget
- [ ] 4.5 Implement the ranked unit-to-site join with confidence labels, a hand-verified override table for the largest units and an unplaced count; verify a report lists counts per confidence level and total placed capacity
- [ ] 4.6 Build boundary geometry and the zone partition, assign every region and placed unit to a zone, and record region-to-zone shares with their method; verify the zone check passes and lists any region or unit without exactly one zone
- [ ] 4.7 Draw schematic corridor routes between zone anchors at 0, 45 and 90 degrees; verify the route check passes
- [ ] 4.8 Make the geodata build deterministic; verify two runs from pinned inputs yield byte-identical outputs and first-paint geometry is within budget when compressed
- [ ] 4.9 Document the pipeline and attribution in `docs/geodata.md`; verify the documented commands reproduce the shipped assets

## 5. Grid frame (`mix-grid-state`)

- [ ] 5.1 Define and document the frame schema (sections, modalities, units, sign convention, provenance classes including scenario, absent representation, network section); verify with a schema validation test over golden frames
- [ ] 5.2 Implement the frame builder with as-of lookup, native resolution and age carry-forward; verify a five-minute tick between half-hourly publications shows the regional values with their period and age
- [ ] 5.3 Implement reconciliation (outturn versus aggregated regional mix, embedded solar rule, tolerance, unexplained-discrepancy flag); verify with fixtures for agreement, the solar gap and a forced disagreement
- [ ] 5.4 Implement integrity checks (shares sum, units, periods, identifiers); verify a corrupted fixture is published flagged, not clean
- [ ] 5.5 Implement derived metrics with method ids (relative carbon intensity, weather-implied versus reported wind and solar, net interconnector share) and their written method definitions; verify each metric resolves to a definition and matches hand-computed values on a fixture
- [ ] 5.6 Expose latest and windowed retrieval with versioning, conditional requests and columnar window encoding; verify an unchanged request returns not-modified and the measured compressed 48-hour window size is recorded against the design estimate
- [ ] 5.7 Enforce that observed frames never contain scenario-class values and that scenario frames name their base; verify with a test that builds a scenario frame and audits the latest observed frame
- [ ] 5.8 Write the frame contract in `docs/frames.md` and commit the golden fixtures used by both server and client tests; verify the doc's example response matches a real response

## 6. Realtime and page shell (`mix-realtime`)

- [ ] 6.1 Render the page shell from a controller with the latest frame and manifest embedded as JSON; verify the page source contains the state and the map paints with sockets blocked
- [ ] 6.2 Mount the React island from a plain client entry at DOM ready and ensure no server patching touches its subtree; verify a tick arriving mid-drag leaves the drag undisturbed
- [ ] 6.3 Add the Channel endpoint with signed short-lived tokens, origin checks and per-workspace join authorisation; verify forged, expired and cross-workspace joins are refused
- [ ] 6.4 Fan out new ticks across nodes with Postgres `NOTIFY` and a local `PubSub` re-broadcast; verify a two-node test delivers an ingest on node A to a client on node B
- [ ] 6.5 Implement catch-up on join (last seen tick in, missed ticks or latest frame out); verify a reconnect-after-gap test ends in the same state as a client that never dropped
- [ ] 6.6 Implement acknowledged, idempotent commits with a client outbox; verify a drop between send and ack results in exactly one node after reconnect
- [ ] 6.7 Implement the conditional-polling fallback and the connection indicator (live, reconnecting, polling, offline, changes queued); verify with the channel blocked that frames still advance and the indicator says so
- [ ] 6.8 Add jittered reconnect backoff, connection caps, message size and rate limits, and input validation on every message; verify a restart-all-nodes test spreads reconnects and an oversized or foreign payload is rejected
- [ ] 6.9 Document the realtime contract in `docs/realtime.md`; verify the documented message shapes match the implementation tests

## 7. First demo: live regional map (`mix-map`, vertical slice)

- [ ] 7.1 Mount the map with the server-seeded latest frame, region shapes and fuel-coloured composition tiles; verify the page paints from shipped assets with the network to providers blocked
- [ ] 7.2 Render interconnector belts from signed measured flow with static direction markers; verify opposite-direction links render oppositely and the layout check shows only 0, 45 and 90 degree segments
- [ ] 7.3 Add the provenance encodings, legend and stale treatment; verify by screenshot review against a fixture with one stale modality
- [ ] 7.4 Add drill-down to source for regions and links; verify selecting a region's wind share shows value, class, provider, period and age
- [ ] 7.5 Advance the map when a tick arrives, fetching conditionally; verify a new frame updates the view with no full reload and no per-frame requests

## 8. Time travel (`mix-time-scrub`)

- [ ] 8.1 Load the 48-hour columnar window after first paint and assemble frames client-side; verify the client output equals the server output on every golden fixture
- [ ] 8.2 Build the Dragology timeline handle with the composition stream as its track; verify no network requests occur during a 24-hour drag and the URL updates once on drop
- [ ] 8.3 Add playback with speeds and stepping; verify playback stops at the live head
- [ ] 8.4 Add live-follow rules and the return-to-live control; verify a frame arriving while scrubbed back leaves the view unchanged
- [ ] 8.5 Mark gaps and handle clock-change windows in the UI; verify screenshots of the 46- and 50-period days show no duplicated or skipped hour
- [ ] 8.6 Implement shareable time links with out-of-window fallback; verify a link restores the same frame and an old link opens at the earliest frame with a notice
- [ ] 8.7 Document the time UI behaviour in `docs/time.md`; verify the documented URL parameters round-trip

## 9. Model parameters (`mix-model-parameters`)

- [ ] 9.1 Build the parameter registry (id, type, range, default, unit, tier, rationale) with validation and rejection reasons; verify tests for in-range, out-of-range and wrong-type values
- [ ] 9.2 Implement priority lists (dispatch, curtailment, redispatch) with completeness and duplicate checks; verify a list missing a fuel is rejected and a reorder changes a golden result
- [ ] 9.3 Add editable factor tables with default sources and non-default flags; verify a changed carbon factor labels results as non-default
- [ ] 9.4 Add tier, security margin, redispatch mode, iteration cap and allocation method parameters; verify each is registered and affects only its documented tier
- [ ] 9.5 Ship named presets and saved custom presets as single edits with difference listings; verify applying a preset creates one node and undo restores the previous values
- [ ] 9.6 Version the registry with the model, and migrate or flag saved nodes that reference changed parameters; verify a renamed-parameter fixture migrates with a notice
- [ ] 9.7 Document the registry in `docs/parameters.md` generated from the code; verify the doc lists every registered parameter

## 10. Scenario model: accounting layer (`mix-scenario-model`)

- [ ] 10.1 Build the base snapshot from a frame (national megawatts by fuel, embedded solar, link flows, demand, declared unit levels), self-contained and compressed; verify a golden snapshot evaluates with the history window empty
- [ ] 10.2 Implement the grid-edit vocabulary with typed parameters and rejection reasons, plus the interconnector capacity table with its sources; verify tests for each edit including an out-of-range rejection that leaves state unchanged
- [ ] 10.3 Implement the derived regional allocation with the capacity-weighted versus equal-weight band; verify regional sums equal national totals within tolerance and the band brackets both allocations
- [ ] 10.4 Implement balancing by the configured priorities and the ledger outputs (composition, carbon intensity, import dependence, surplus, deficit, unmet demand); verify golden scenarios for a large gas removal that leaves unmet demand and for a surplus case
- [ ] 10.5 Add model versioning and plain-language edit and result descriptions; verify a factor-table change forces a version bump and descriptions match golden strings
- [ ] 10.6 Implement the evaluator in JavaScript with the agreed tolerance and rounding; verify every golden scenario agrees with the Elixir implementation within tolerance and the server's rounded values are the stored ones
- [ ] 10.7 Hold a domain sanity review of the default priorities, factors and per-tier caveat wording; record assumptions in `docs/scenario-model.md`; verify the doc names a reviewer and every default in design.md is confirmed or amended

## 11. Network model: physics layer (`mix-network-model`)

- [ ] 11.1 Build the zone graph, the boundary-to-cut mapping with an unmapped list, and zone injections (declared unit levels, allocated embedded generation and demand, interconnectors); verify the mapping check lists unmapped boundaries and injections sum to the ledger totals
- [ ] 11.2 Implement the boundary tier (net transfer across each cut against its limit); verify the north-of-boundary example in the spec computes the expected utilisation
- [ ] 11.3 Build the calibration and validation report (per-boundary residuals over the hindcast window, confidence classes, low-confidence labelling); verify the report is regenerated by one command and every boundary has a class
- [ ] 11.4 Implement the flow tier on calibrated equivalent corridors with documented slack handling (subject to the decision from 1.5); verify the baseline reproduces published flows within tolerance for high-confidence boundaries
- [ ] 11.5 Implement overload detection with the security margin and the greedy sensitivity-based redispatch loop with converged, iteration-limited and infeasible statuses; verify a wind-behind-constraint fixture and an unrelievable fixture behave as specified
- [ ] 11.6 Implement network edits (outage, derate, upgrade, reactance) and isolation reporting; verify an outage that islands a zone reports isolation, unmet demand and still evaluates the rest
- [ ] 11.7 Implement the physics evaluator in JavaScript and add shared golden cases; verify server and client agree within tolerance and a deliberate perturbation is logged as a mismatch
- [ ] 11.8 Show derived corridor flows and boundary loading in observed mode on the map with confidence and published residuals; verify the corridor belts use the derived encoding, show low-confidence labels, and an overloaded fixture shows the non-colour cue
- [ ] 11.9 Measure evaluation time per tier at the largest network on the slowest team laptop; verify against the documented budget and record it in `docs/network-model.md` with the method and honest-limits text

## 12. Visitor workspaces and version tree (`mix-visitor-state`, `mix-history-graph` server)

- [ ] 12.1 Issue the signed visitor cookie only on first save, with the notice from 1.7; verify browsing and scrubbing set no cookie and the first commit sets one with the documented attributes
- [ ] 12.2 Add migrations for workspaces, trees, content-addressed nodes, node metadata and shared snapshots; verify migrate-up runs once and conflict-ignoring inserts keep one row per node id under a concurrency test
- [ ] 12.3 Implement canonical edit encoding and node hashing; verify the same edit on the same parent yields one id and a label change leaves ids unchanged
- [ ] 12.4 Implement the tree API (create from a base, append with validation, evaluation and stored summary with mismatch logging, fetch, metadata, archive and restore) with per-visitor authorisation; verify two concurrent appends produce two siblings and a request for another visitor's tree is indistinguishable from not found
- [ ] 12.5 Enforce quotas (trees, nodes, shares, bytes) and creation rate limits; verify refusal tests at each cap with the explanatory message
- [ ] 12.6 Implement idle workspace retention (default 7 days) with a scheduled deletion job and an expiry indicator value; verify a time-travelled test deletes only idle workspaces
- [ ] 12.7 Implement publish: immutable snapshot of a path or whole tree with notes opt-in, its own unguessable id, 30-day since-last-opened retention extended on open, and revocation; verify later edits do not alter a snapshot and an expired link shows the expired message
- [ ] 12.8 Implement read-only shared views and fork-on-first-edit into the viewer's workspace; verify the snapshot is byte-identical after a viewer edits
- [ ] 12.9 Implement clear-my-data and the data-minimisation audit (no IP or device data stored, logs free of workspace content); verify a stored record and a log sample contain neither
- [ ] 12.10 Handle model-version mismatch with explicit re-evaluation; verify a saved tree opened after a version bump keeps old results until re-evaluated
- [ ] 12.11 Document the data model, limits, retention and link semantics in `docs/workspaces.md`; verify the documented API examples return the documented responses

## 13. Workspace and edit composition (`mix-workspace`)

- [ ] 13.1 Implement the mode state (observed live, observed at a time, scenario with tier) with the persistent indicator and the scenario frame treatment; verify a test that no panel renders a scenario value outside scenario mode and a screenshot of each mode
- [ ] 13.2 Build the panel framework (open, close, dock, per-viewer layout restore, lazy chunks for graph, compare, model and method); verify layout restores on reload and the release build lists the panels as separate chunks
- [ ] 13.3 Build the Dragology edit palette on an overlay SVG with live preview from the client evaluator running outside the drag's render path; verify a drag makes no requests, Escape creates no node, a drop creates exactly one node and one save, and frame times hold with the full map and a 200-node graph open
- [ ] 13.4 Build the inspector with provenance, numeric inputs and sliders for every edit; verify a keyboard-only run creates the same node as the drag path and announces the change
- [ ] 13.5 Build the model panel with the draggable priority lists, non-default flags, presets and reset; verify a keyboard reorder creates the same edit as dragging and previews its effect before commit
- [ ] 13.6 Build the results panel (fidelity badge, redispatch status, overloads, per-boundary confidence and residual with a link to the calibration report); verify a low-confidence boundary fixture is marked
- [ ] 13.7 Add the preview strip, shortcuts with cheat sheet and screen-reader announcements; verify deltas match the evaluator and each shortcut works without shadowing a browser default
- [ ] 13.8 Show saving and sync state, the workspace expiry, and the share, revoke and clear controls with confirmations; verify a forced save failure shows failed, keeps the node in the outbox and succeeds on retry
- [ ] 13.9 Implement responsive layouts with bottom sheets; verify a 390-pixel walkthrough (edit, undo, compare, live) with no horizontal page scroll
- [ ] 13.10 Add first-run guidance and the method, caveats and sources panel reachable from the mode indicator; verify the first edit shows the explanation once and the method panel opens at the current model version
- [ ] 13.11 Document the workspace modes, panels and shortcuts in `docs/workspace.md`; verify the documented shortcuts match the cheat sheet

## 14. History graph panel (`mix-history-graph` client)

- [ ] 14.1 Implement the append-only stable tree layout under the trunk, aligned to the scrubber's time axis; verify adding a node never moves existing nodes except sibling stacking
- [ ] 14.2 Render nodes with composition glyphs, edges, edit class markers, the live marker and base staleness; verify screenshot fixtures for a small tree and a tree three hours behind live
- [ ] 14.3 Implement undo and redo semantics (parent, most recent child, choice when several); verify tests across a fork
- [ ] 14.4 Implement jump, the back stack, return to live and resume as separate pointers; verify a walkthrough: jump A to B and back, then live and resume restores node and selection
- [ ] 14.5 Build node details with the ordered change log (with edit classes) and actions (jump, branch, compare, label, note, pin, archive, restore, share); verify renaming keeps identity and archive then restore returns the branch exactly
- [ ] 14.6 Add pan, zoom, minimap, viewport virtualisation and keyboard traversal; verify a 200-node tree stays within the frame budget and every node is keyboard reachable
- [ ] 14.7 Guarantee observed frames never alter nodes while the staleness label updates; verify a test injects three hours of frames and asserts nodes unchanged and the label updated

## 15. Compare (`mix-compare`)

- [ ] 15.1 Implement selection of two to four states (observed or scenario), reference selection and swap; verify a swap inverts every delta and changes no absolute value
- [ ] 15.2 Build the metric table with deltas, sorting, class labels and network metrics where the tier has them, plus the like-for-like guard for base, tier and model version; verify fixtures with different bases and different tiers show the persistent warnings
- [ ] 15.3 Build the composition difference chart in the fuel palette; verify a gas-removal fixture shows the shrunk and grown bands
- [ ] 15.4 Compute the lowest common ancestor, each side's edits since it grouped by class, and the parameter differences; verify a diverging-branch fixture and a policy-only fixture list the expected edits and parameters
- [ ] 15.5 Add the map delta overlay with a colour-vision-safe diverging encoding and non-colour cue, and synchronised side-by-side maps from one shared viewport; verify the sync test and the frame budget with four maps
- [ ] 15.6 Flag unmet demand in the state, table and map, add shareable comparison links and the screen-reader summary; verify an unmet-demand fixture is flagged in all three places and a shared link restores the same comparison
- [ ] 15.7 Document comparison semantics in `docs/compare.md`; verify the documented example reproduces on a fixture

## 16. Depth (`mix-map` granularity)

- [ ] 16.1 Implement semantic zoom to region and asset or unit detail using the pan decision from 1.3; verify labels reflow and tiles and belts keep correct values at each level
- [ ] 16.2 Render the lazy asset layer with confidence styling and the unplaced count; verify an unplaced fixture unit appears in the count and not on the map
- [ ] 16.3 Add the weather overlay and the derived wind plausibility signal; verify a mismatch fixture highlights the region without changing its reported share
- [ ] 16.4 Add fuel isolation by dragging chips in and out of the active set; verify live emphasis during the drag, one URL commit on drop and no per-frame requests

## 17. Art direction and accessibility

- [ ] 17.1 Define colour tokens (light and dark), typography and belt, tile, corridor, node-glyph and scenario-skin motifs in a style guide; verify the guide renders every token and motif on a reference page
- [ ] 17.2 Validate the fuel palette, the loading palette and the diverging delta palette under protanopia, deuteranopia and tritanopia and check text contrast against WCAG AA; verify with a script whose output is committed with the guide
- [ ] 17.3 Implement reduced motion with static direction and magnitude cues; verify no animation runs with the preference set and direction stays readable
- [ ] 17.4 Add full keyboard navigation and a data-table view of the current frame; verify a keyboard-only walkthrough reaches every region, link and corridor and the table matches the frame
- [ ] 17.5 Design the degraded states (missing modality, failed integrity, outage, replay, save failure, model-version mismatch, low-confidence boundary); verify each state has a screenshot in the guide and replay is unmistakable

## 18. Resilience and replay (`mix-ingest`)

- [ ] 18.1 Build the capture tool exporting retained raw payloads as a replay bundle covering at least 48 hours; verify the bundle size and record the hosting decision from design.md
- [ ] 18.2 Implement replay mode through the production normaliser and frame builder with looping; verify the full product, including scenarios and the physics tiers, runs with outbound access blocked and the replay marker visible
- [ ] 18.3 Add the data-sources panel with attribution and licence text for every source; verify every active source appears with its required wording
- [ ] 18.4 Add the scheduled live-schema canary (non-gating) and a status endpoint showing last success and lag per series; verify a deliberately broken fixture triggers the canary report without failing CI
- [ ] 18.5 Run CI entirely from fixtures on the `durhack-ci` runners; verify a CI run with provider hosts blocked passes and that no workflow invokes a paid Entire command

## 19. Stretch

- [ ] 19.1 Add the constraint-cost modality from NESO open data as a layer (decide scope against the design.md open question first); verify it carries its class, source and attribution
- [ ] 19.2 Add an optional DC optimal power flow solver tier and compare it against the greedy redispatch on golden cases; verify the comparison report lists differences in volumes and loading
- [ ] 19.3 Add an indicative redispatch cost parameter labelled indicative; verify its default is sourced and results using it carry the label

## 20. Integration checks

- [ ] 20.1 Run the performance budget on the target laptop with all belts animating, scrubbing, dragging an edit at the flow tier and opening compare; verify frame times and the animated-element cap against the numbers set in 1.4
- [ ] 20.2 Run the production audit from `platform-runtime`: digested assets, canvas code-split, map and panel bundles absent from non-map pages; verify with the release build
- [ ] 20.3 Run a browser network audit; verify no request to any provider host and no per-frame requests, including during edit drags
- [ ] 20.4 Run an outage drill (block providers mid-session, then restart with cold cache), a channel-blocked drill and a save-failure drill mid-edit; verify stale, degraded, polling, replay and save-failed states appear as designed and no committed node is lost
- [ ] 20.5 Run an isolation and retention audit with two browsers: separate histories, no cross-reads, shared link read-only, fork on edit, revoke, clear-my-data and expiry; verify each against the specs
- [ ] 20.6 Rehearse the demo end to end through the Cloudflare Tunnel on the deploy host, including branching two choices at different tiers, comparing them, sharing a link, opening it in a second browser and returning to live; verify live and replay both work from the public domain

## Workflow follow-up

- Review the artifacts with the team, then use the apply workflow to start with group 1 once baseline phases 2 and 3 are done.
- Consider amending the baseline `platform-drag-integration` wording to name a Channel push as an equivalent once-per-drop transport.
- Sync the thirteen capability specs into `openspec/specs/` and archive this change after the project's review requirements are satisfied.
- Verify the archived specs match the shipped behaviour.
