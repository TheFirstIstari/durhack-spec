# Proposal

## Why

DurHack project has no recorded baseline: stack, platform, and delivery path live only in conversation. Writing them down now gives every future change a shared contract to build against before any code exists.

## What Changes

- Records the decided technical baseline: Elixir + Phoenix (server), React 19 + Dragology (client drag canvas), all tooling standardised and pinned via mise.
- Records the runtime baseline: stateless nodes scalable horizontally and vertically, persistent data storage via Postgres/Ecto, production releases with standard Phoenix efficiency levers (digested assets, code splitting where cheap, no per-frame server traffic).
- Records the delivery baseline: CI on self-hosted CachyOS x86_64 runners with dependency caching, deploy target CachyOS x86_64, public access via Cloudflare Tunnel from a public domain.
- Records the agent-coordination baseline: Entire CLI (free tier, no paid review agents) for session checkpoints, cross-agent handoff, and a GitHub mirror; every contributor installs hooks and pushes checkpoint context with commits.
- Declares this repo specs-only: no application code lives here — plans, capability specs, and task runbooks only. The Phoenix project scaffolds separately and references this repo as its contract.

## Capabilities

### New Capabilities

- `platform/runtime` — scalability (horizontal + vertical), persistent storage, and efficiency constraints.
- `platform/agent-coordination` — Entire CLI setup, checkpoint capture, mirror, and the paid-agent opt-out.
