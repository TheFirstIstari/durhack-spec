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
- **mise is the single source of toolchain truth.** `mise.toml` pins Elixir/OTP/Node and any extra CLIs; local dev, CI, and deploy all activate it. Nothing is installed ad-hoc outside mise.
- **Self-hosted runners keep arch identical to deploy.** Same CachyOS x86_64 image family for CI and target removes cross-arch surprises; tradeoff is the team owns runner uptime.
- **Stateless nodes + Postgres is the scaling story.** `mix release` nodes hold no durable state; Ecto/Postgres owns it; PubSub adapter choice deferred until multiplayer needs land. Vertical scaling comes free with the BEAM; horizontal needs a second node behind the tunnel and shared storage — nothing else.
- **Efficiency = release + digested assets + canvas code-split + drop-only pushes.** Deliberately scoped to standard Phoenix levers; no exotic optimisation (clustering, CDN, read replicas) until a measured bottleneck exists.

## Risks / Trade-offs

- Dragology is a research prototype (~467★, LLM-written docs, unstable API): budget learning time; start with one simple `d.between`/`d.vary` interaction.
- Cloudflare Tunnel adds an external dependency at demo time; mitigate by testing tunnel + domain before judging day.
- CachyOS x86_64 everywhere removes arch skew but narrows runner options; confirm runner availability early.
