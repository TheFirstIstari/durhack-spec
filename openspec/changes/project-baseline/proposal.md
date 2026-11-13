# Proposal

## Why

DurHack project has no recorded baseline: stack, platform, and delivery path live only in conversation. Writing them down now gives every future change a shared contract to build against before any code exists.

## What Changes

- Records the decided technical baseline: Elixir + Phoenix (server), React 19 + Dragology (client drag canvas), all tooling standardised and pinned via mise.
- Records the runtime baseline: stateless nodes scalable horizontally and vertically, persistent data storage via Postgres/Ecto, production releases with standard Phoenix efficiency levers (digested assets, code splitting where cheap, no per-frame server traffic).
- Records the delivery baseline: CI on self-hosted CachyOS x86_64 runners with dependency caching, deploy target CachyOS x86_64, public access via Cloudflare Tunnel from a public domain.
- Records the Dragology integration contract: Phoenix builds and serves the Dragology client bundle (JSX loader, npm deps, React island with `phx-update="ignore"`); client owns per-frame drag state, server seeds initial state and persists drops only.

## Capabilities

### New Capabilities

- `platform/stack` — language, framework, and client library decisions with versions and constraints.
- `platform/runtime` — scalability (horizontal + vertical), persistent storage, and efficiency constraints.
