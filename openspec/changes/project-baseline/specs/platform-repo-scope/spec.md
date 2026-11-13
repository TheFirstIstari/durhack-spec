# Spec Delta

## Purpose

This repository is the hackathon's planning and coordination surface: capability contracts, phased development plan, and task runbooks. Application code is built elsewhere and judged against these specs.

## ADDED Requirements

### Requirement: Specs-only repository

This repo SHALL contain no application code — only OpenSpec changes, capability specs, task runbooks, and supporting docs. Build scaffolding, CI config for the app, and deploy manifests live in the application repo(s).

#### Scenario: New file added to this repo

- **WHEN** a contributor adds a file
- **THEN** it is a spec, plan, runbook, or supporting doc — never application source, lockfiles, or build artifacts

### Requirement: Development plan lives in the baseline change

The phased development plan SHALL live in this change's `tasks.md`, ordered so each phase is demoable. Phase order: scaffold → prove drag island → delivery path → prove runtime → Entire coordination → project identity (separate change once decided).

#### Scenario: Planning the next work block

- **WHEN** the team picks up the next block of work
- **THEN** `tasks.md` identifies the current phase, its acceptance checks, and what unblocks the phase after it — no separate planning doc needed

### Requirement: Capability specs are the contract

Every durable system behavior SHALL be recorded as a capability spec under `specs/` (new) or in `openspec/specs/` once archived. Implementation in the application repo MUST satisfy the archived specs; behavior changes go through a new OpenSpec change, not silent edits.

#### Scenario: Dispute over intended behavior

- **WHEN** two contributors disagree on what the system should do
- **THEN** the archived capability spec settles it; if the spec is wrong, a new change amends it
