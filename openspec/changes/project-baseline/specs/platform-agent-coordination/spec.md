# Spec Delta

## Purpose

The agent-coordination baseline makes every agent session recoverable by any teammate: Entire captures session checkpoints into Git, links them to commits, and mirrors the repo — all on the free tier, with paid review agents explicitly out of scope.

## ADDED Requirements

### Requirement: Entire CLI enabled with agent hooks

Every contributor SHALL run `entire enable` in the repo checkout and install hooks for each agent they use (`entire agent add opencode`, plus other agents as needed). Agent-assisted commits SHALL link to their session checkpoint (answer `always` at the link prompt).

#### Scenario: Teammate traces agent-produced code

- **WHEN** a contributor runs `entire checkpoint explain` (or `entire why <file>`) on a commit they did not author
- **THEN** they see the originating prompt, transcript, and attribution behind that commit without asking its author

### Requirement: Cross-agent handoff via sessions

Handoffs between contributors or between agent types SHALL go through Entire sessions (`entire session resume`), not by pasting transcripts.

#### Scenario: Agent-to-agent handoff

- **WHEN** one contributor's agent session ends and another picks up the work
- **THEN** the second contributor resumes the recorded session (`opencode -s SESSION_ID` or equivalent printed by `entire session resume`) with full context restored

### Requirement: GitHub mirror for shared checkpoint access

The `TheFirstIstari/durhack` GitHub repo SHALL have an Entire mirror (`entire repo mirror add /gh/TheFirstIstari/durhack`, `entire repo remote add entire`) so pushed checkpoints are retrievable by all teammates.

#### Scenario: New machine joins mid-hackathon

- **WHEN** a teammate clones and fetches through the Entire remote
- **THEN** they can search and inspect prior checkpoints (`entire search --compact`, `entire checkpoint list`) without any local session history

### Requirement: No paid review agents

The project SHALL NOT depend on Entire paid review agents (`entire review`, custom agents, recap). All coordination uses the free tier: CLI, checkpoints, search, session resume, trails. Paid-agent usage is opt-in per individual and MUST NOT gate any team workflow.

#### Scenario: Credits exhausted

- **WHEN** any individual's Entire agent credit runs out (or they never added a card)
- **THEN** checkpoint capture, search, resume, and all team workflows continue working unchanged
