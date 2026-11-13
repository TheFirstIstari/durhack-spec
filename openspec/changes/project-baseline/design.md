# Design

## Context

Greenfield repo: no code, no specs. Toolchain observed on dev machine: Elixir 1.20.4 / OTP 29 via mise, Node 20. Delivery hosts and self-hosted runners are CachyOS x86_64. See proposal.md for motivation.

## Goals / Non-Goals

**Goals:**

- Lock stack, delivery, drag-integration, and runtime boundaries before scaffolding.
- Keep per-frame drag work client-side; Phoenix handles persistence/broadcast only.

**Non-Goals:**

- Data model, auth, multiplayer protocol (unknown until identity lands).
- Concrete scaling targets / load numbers (no traffic profile yet; baseline only requires the architecture not block scaling).

## Decisions

- **Phoenix serves shell + API, React owns canvas.** Dragology speculatively re-renders every pointer frame; a server round-trip per frame is infeasible. Alternative (LiveView-rendered SVG) rejected: latency kills drag feel.
- **Dead view or `phx-update="ignore"` island preferred over full LiveView for canvas pages.** Avoids morphdom clobbering React's `id`-reconciled tree.
- **Repo-scoped runners, exact labels.** Repo-level runners can't serve multiple repos (that's org-level, needs an org). So `TheFirstIstari/durhack` gets 4× `durhack-ci` + 1× `durhack-deploy` on cachyos-x8664; workflows target `[self-hosted, durhack-ci]` / `[self-hosted, durhack-deploy]`. Sharing one pool across repos would need a shared GitHub org — rejected: overkill for a hackathon, and blast radius if one repo's workflow goes rogue. Deploy runner label never overlaps CI. Dripstone/irc-serve runners untouched (own dirs, units, labels).
- **Runner mise via shims, repo pins via mise.toml.** Runners don't source shell rc files, so each runner dir ships a `.env` putting `~/.local/share/mise/shims` on PATH. System `mise` (2026.9.9 on the box) resolves tools; workflows run `mise install`. Never `pacman`-install Elixir/Node in CI.
- **Stateless nodes + Postgres is the scaling story.** `mix release` nodes hold no durable state; Ecto/Postgres owns it; PubSub adapter choice deferred until multiplayer needs land. Vertical scaling comes free with the BEAM; horizontal needs a second node behind the tunnel and shared storage — nothing else.
- **Efficiency = release + digested assets + canvas code-split + drop-only pushes.** Deliberately scoped to standard Phoenix levers; no exotic optimisation (clustering, CDN, read replicas) until a measured bottleneck exists.

- Cloudflare Tunnel adds an external dependency at demo time; mitigate by testing tunnel + domain before judging day.
- CachyOS x86_64 everywhere removes arch skew but narrows runner options; confirm runner availability early.
- Two repos to keep straight: `durhack-spec` (this planning repo, commits freely) vs `durhack` (code repo, stays empty until the hackathon starts). Entire mirror for the code repo is a start-of-event task.
