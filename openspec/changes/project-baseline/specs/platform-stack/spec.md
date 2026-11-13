# Spec Delta

## Purpose

The platform stack pins the languages, frameworks, and versions every future change builds against, so implementation choices stay consistent across the hackathon.

## ADDED Requirements

### Requirement: mise-standardised toolchain

All project tooling (Elixir, OTP, Node, and any other runtimes or CLIs) SHALL be installed, pinned, and invoked via mise, with versions recorded in `mise.toml` at the repo root.

#### Scenario: New contributor setup

- **WHEN** a contributor clones the repo and runs the documented setup command
- **THEN** mise installs the exact pinned toolchain (no manual version management) and project commands run against it

#### Scenario: CI uses the same toolchain

- **WHEN** CI runs any build, test, or release step
- **THEN** it activates the same `mise.toml` pins as local development, so versions never drift between machines

#### Scenario: Runner resolves tools via mise shims

- **WHEN** a job runs on a `durhack-ci` or `durhack-deploy` runner
- **THEN** `mise` (system install) with its shims on `PATH` resolves `elixir`/`mix`/`node` from the repo's `mise.toml`; workflows call `mise install` (or `jdx/mise-action`), never `apt`/`pacman` for toolchain languages

### Requirement: Elixir server stack

The project SHALL use Elixir with the Phoenix framework for all server-side code.

#### Scenario: New server code follows the stack

- **WHEN** a contributor adds server-side behavior
- **THEN** it is implemented in Elixir on Phoenix, using the project's mise-pinned toolchain

### Requirement: React + Dragology drag canvas

All bespoke draggable SVG interactions SHALL be built with Dragology on React 19, running client-side in the browser.

#### Scenario: New draggable element

- **WHEN** a draggable UI element is added
- **THEN** it is a Dragology `Draggable` rendered by `DraggableRenderer` inside a React island, following Dragology's positioning rules (SVG `transform`, `id` identity, no React `key`)

### Requirement: Package versions pinned

The project SHALL record exact pinned versions of Elixir, OTP, Node, React, and Dragology in a follow-up change once the project is scaffolded.

#### Scenario: Version audit before judging

- **WHEN** the team verifies the build before demo
- **THEN** `mise.toml` / lockfiles reproduce the same toolchain and JS dependencies on a clean machine
