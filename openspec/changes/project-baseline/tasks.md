# Tasks — development plan (this repo is specs-only; app phases execute in the application repo)

## 1. Agent coordination (both repos; free tier only)

- [x] 1.1 `entire enable -y --agent opencode` in `durhack-spec` repo root — hooks installed (`.opencode/plugin/entire.ts`, `.entire/settings.json` committed)
- [ ] 1.2 Re-verify the mirror: it was recorded as created (`entire repo mirror add "/gh/TheFirstIstari/durhack"`, aws-us-east-2) but no `entire` remote exists in the current checkouts; confirm on the machine that created it, add the `entire` remote where missing (`entire repo remote add`), and verify a fetch through it
- [ ] 1.3 Remove the stray agent hook directories committed under `openspec/changes/project-baseline/` (`.entire/`, `.claude/`, `.pi/`), then install hooks at the `durhack-spec` repo root for every agent in use, including Claude Code (`entire agent add claude-code`); verify with `git ls-files | grep 'project-baseline/\.'` returning nothing and `entire status --detailed` listing each agent
- [ ] 1.4 Commit the team settings in `.entire/settings.json` (`commit_linking: always`, PII redaction on, summaries off, checkpoint-push decision) and document personal-only keys; verify `entire status --detailed` shows the effective values
- [ ] 1.5 Record the checkpoint-visibility decision: check repo visibility (`gh repo view --json visibility`) and choose private repo, separate private checkpoint remote, or no session push; verify the choice is written in `docs/agent-coordination.md` before any checkpoint push
- [ ] 1.6 Each additional contributor: `entire login`, `entire agent add <their-agents>` (GUI Git clients: absolute hook path); verify with `entire status --detailed` and a test commit carrying an `Entire-Checkpoint` trailer
- [ ] 1.7 Verify a checkpoint round-trips: agent change → commit (trailer present) → `entire checkpoint list` → second machine fetches and runs `entire checkpoint explain` and `entire why <file>`
- [ ] 1.8 Verify `entire search "<query>"` and `entire session resume <branch>` from a clean checkout; the team never uses `--force` blindly and NEVER runs `entire review`, trail review agents, `entire checkpoint explain --generate`, summaries or memory extraction (paid or metered agents); verify by grepping CI and hooks for those commands
- [ ] 1.9 Optional: enable local continuous memory after the first commit (Git 2.36 or later, extraction declined, no hosted sync); verify the generated `.entire/agent-guide.md` is reviewed and committed
- [ ] 1.10 App repo `durhack` (currently no commits, no `.entire/`, no hooks): at its first commit run `entire enable --agent <agents>`, commit the team settings, create or confirm the mirror, and verify the first commit carries an `Entire-Checkpoint` trailer

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
