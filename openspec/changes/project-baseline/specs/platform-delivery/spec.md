# Spec Delta

## Purpose

The delivery path defines where the project builds, where it runs, and how the public reaches it, so CI, deploy, and demo networking are decided once.

## ADDED Requirements

### Requirement: Self-hosted CI runners only

All continuous-integration runs for the application repo (`TheFirstIstari/durhack`) SHALL execute on its dedicated `durhack-ci` runners on cachyos-x8664. No cloud-hosted, shared, or other-repo runners SHALL be used.

#### Scenario: Push triggers CI

- **WHEN** a contributor pushes a commit to the application repo
- **THEN** build, test, and release artifact steps run on a `durhack-ci` runner and report pass/fail on the commit

### Requirement: Dedicated deploy runner

Deploys SHALL run only on the `durhack-deploy` runner. CI jobs MUST NOT be able to land on it (workflows target exact labels; the deploy label is never on a CI runner and vice versa).

#### Scenario: CI and deploy queued together

- **WHEN** a CI run and a deploy dispatch overlap
- **THEN** they execute on their respective runners without queueing behind each other

### Requirement: CI dependency caching

The CI pipeline SHALL cache dependency artifacts (Hex packages, npm modules, mise-installed toolchains, and build outputs where safe) across runs, keyed so a cache hit restores them and a lockfile change invalidates them.

#### Scenario: Repeat run with warm cache

- **WHEN** CI runs on a commit whose lockfiles match a previous run
- **THEN** dependency-fetch steps restore from cache instead of re-downloading, and the run completes measurably faster than a cold run

#### Scenario: Lockfile change invalidates cache

- **WHEN** a lockfile or `mise.toml` changes
- **THEN** the affected cache is rebuilt from scratch and subsequent runs use the fresh cache

### Requirement: Deploy target CachyOS x86_64

Releases SHALL target CachyOS x86_64 as the deployment host.

#### Scenario: Release deploys to target

- **WHEN** a release is cut
- **THEN** the artifact runs on CachyOS x86_64 without architecture-specific modification

### Requirement: Public access via Cloudflare Tunnel

Public traffic SHALL reach the deployed app through a Cloudflare Tunnel fronted by a public domain.

#### Scenario: Public demo access

- **WHEN** a judge or visitor opens the public domain during the event
- **THEN** the tunnel routes them to the running CachyOS x86_64 deployment over HTTPS
