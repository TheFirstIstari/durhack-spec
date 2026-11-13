# Tasks

## 1. Scaffold project

- [ ] 1.1 Run `mix phx.new` with Ecto/Postgres in repo root; record all toolchain pins (Elixir/OTP/Node) in `mise.toml` — no ad-hoc installs

## 2. Prove drag island

- [ ] 2.1 Mount minimal `DraggableRenderer` (toggle demo) inside `phx-update="ignore"` region, with initial state seeded from the server (embedded in page or fetched on mount)
- [ ] 2.2 Wire `onDropState` to push final state to server once per drop; verify no per-frame requests
- [ ] 2.3 Verify LiveView re-render leaves mounted canvas and in-progress drag undisturbed

## 3. Delivery path

- [ ] 3.1 Add CI workflow pinned to self-hosted CachyOS x86_64 runners (build + test + artifact); fail closed if a cloud/shared runner picks it up
- [ ] 3.2 Add dependency caching to CI (Hex, npm, mise data, build outputs), keyed on lockfiles / `mise.toml`; verify a warm-cache rerun skips re-downloads
- [ ] 3.3 Verify release artifact runs on CachyOS x86_64 deploy host
- [ ] 3.4 Configure Cloudflare Tunnel + public domain; verify HTTPS reaches the deployment

## 4. Prove runtime baseline

- [ ] 4.1 Verify a `mix release` build with digested/compressed assets; canvas bundle absent from non-canvas pages
- [ ] 4.2 Verify restart/redeploy preserves Postgres-backed state; migrations run exactly once per deploy
- [ ] 4.3 Verify a second identical node can start behind the tunnel with no node-local state required

## 5. Follow-ups (out of this change)
- [ ] 5.2 Record exact pinned versions of all toolchain deps once scaffolded

## 6. Entire agent coordination ( researched from https://docs.entire.com — all free tier )

- [ ] 6.1 Each contributor: `entire login` (device flow on headless: `entire login --device`), then `entire enable -y --agent opencode` in the repo root; answer `always` at the checkpoint-link prompt
- [ ] 6.2 Each contributor: `entire agent add <each-agent-they-use>` (hook map: OpenCode → `.opencode/plugins/entire.ts`, Pi → `.pi/extensions/entire/index.ts`); verify with `entire agent list` + `entire status --detailed`
- [ ] 6.3 Create the mirror: `entire repo mirror add "/gh/TheFirstIstari/durhack" --cluster <nearest-region>`, then `entire repo remote add entire`; verify `git fetch entire && git push entire` round-trips to GitHub
- [ ] 6.4 Verify a checkpoint round-trips: agent change → commit → `entire checkpoint list` shows it → second machine fetches and `entire checkpoint explain`s it
- [ ] 6.5 Verify `entire search --compact` and `entire session resume` work from a clean checkout; do NOT run `entire review` (paid agents) in any team workflow
