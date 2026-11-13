# Tasks — development plan (this repo is specs-only; app phases execute in the application repo)

## 1. Agent coordination (this repo — DONE)

- [x] 1.1 `entire enable -y --agent opencode` in repo root — hooks installed (`.opencode/plugin/entire.ts`, `.entire/settings.json` committed)
- [x] 1.2 Mirror created: `entire repo mirror add "/gh/TheFirstIstari/durhack"` (aws-us-east-2); remote `entire` added, fetch verified
- [ ] 1.3 Each additional contributor: `entire login`, `entire agent add <their-agents>`; verify with `entire agent list` + `entire status --detailed`
- [ ] 1.4 Verify a checkpoint round-trips: agent change → commit (answer `always` at link prompt) → `entire checkpoint list` → second machine fetches and `entire checkpoint explain`s it
- [ ] 1.5 Verify `entire search --compact` and `entire session resume` from a clean checkout; NEVER run `entire review` (paid agents) in team workflows

## 2. Scaffold application (app repo — `TheFirstIstari/durhack`, stays EMPTY until event start)

- [ ] 2.1 On cachyos-x8664: `GH_RUNNER_TOKEN_CI=… GH_RUNNER_TOKEN_DEPLOY=… sudo -E ~/setup-durhack-runners.sh` (tokens from repo Settings → Actions → Runners); registers 4× `durhack-ci` + 1× `durhack-deploy`, installs `durhack-runner-ci@` / `durhack-runner-deploy@` system units; verify `systemctl list-units 'durhack-runner-*'` all active and GitHub shows 5 idle runners
- [ ] 2.2 Run `mix phx.new` with Ecto/Postgres (at event start); record all toolchain pins (Elixir/OTP/Node) in `mise.toml` — no ad-hoc installs
- [ ] 2.3 Add JSX loader to esbuild config; install `dragology`, `react`, `react-dom` (React ^19) via npm in `assets/`; verify `mix assets.build` passes via mise toolchain
- [ ] 2.4 Record exact pinned versions of all toolchain deps

## 3. Prove drag island (app repo)

- [ ] 3.1 Mount minimal `DraggableRenderer` (toggle demo) inside `phx-update="ignore"` region, with initial state seeded from the server (embedded in page or fetched on mount)
- [ ] 3.2 Wire `onDropState` to push final state to server once per drop; verify no per-frame requests
- [ ] 3.3 Verify LiveView re-render leaves mounted canvas and in-progress drag undisturbed

## 4. Delivery path (app repo + infra)

- [ ] 4.1 Add CI workflow pinned to self-hosted CachyOS x86_64 runners (build + test + artifact); fail closed if a cloud/shared runner picks it up
- [ ] 4.2 Add dependency caching to CI (Hex, npm, mise data, build outputs), keyed on lockfiles / `mise.toml`; verify a warm-cache rerun skips re-downloads
- [ ] 4.3 Verify release artifact runs on CachyOS x86_64 deploy host
- [ ] 4.4 Configure Cloudflare Tunnel + public domain; verify HTTPS reaches the deployment

## 5. Prove runtime baseline (app repo)

- [ ] 5.1 Verify a `mix release` build with digested/compressed assets; canvas bundle absent from non-canvas pages
- [ ] 5.2 Verify restart/redeploy preserves Postgres-backed state; migrations run exactly once per deploy
- [ ] 5.3 Verify a second identical node can start behind the tunnel with no node-local state required

## 6. Project identity (new change, once decided)

- [ ] 6.1 Define concept, users, and interactions in a new OpenSpec change; behavior specs constrain the app build
