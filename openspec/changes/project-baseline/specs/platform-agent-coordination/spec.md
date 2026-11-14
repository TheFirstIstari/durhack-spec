# Spec Delta

## Purpose

The agent-coordination baseline makes every agent session recoverable by any teammate: Entire captures session checkpoints into Git, links them to commits, and mirrors the repo, using only free-tier features. It also pins down the privacy, safety and cost boundaries that the free tier does not enforce on its own.

## ADDED Requirements

### Requirement: Entire enabled at each repo root with hooks per agent

Each repo (`durhack-spec` and `durhack`) SHALL have Entire enabled at its root, with hooks installed for every agent contributors use (`entire enable --agent <agent>` on first setup, `entire agent add <agent>` afterwards). Hook files MUST live at the repo root, not in subdirectories such as a change folder.

#### Scenario: Setup run from a subdirectory

- **WHEN** a contributor runs enable or agent setup from inside a subdirectory
- **THEN** the stray hook files are removed and setup is re-run at the repo root, and `git ls-files` shows no agent hook directories below the root

### Requirement: Capture is verified, not assumed

Because agent hooks exit silently when the `entire` binary is missing from `PATH`, each contributor SHALL verify capture after setup with `entire status --detailed` and a test commit that carries an `Entire-Checkpoint` trailer. Contributors using GUI Git clients MUST set the absolute hook path.

#### Scenario: Missing binary

- **WHEN** a contributor's agent session runs on a machine where `entire` is not on `PATH`
- **THEN** the verification step fails visibly (no trailer on the test commit) instead of work silently going uncaptured

### Requirement: Team settings are committed, personal settings stay local

Team-wide behaviour SHALL live in the committed `.entire/settings.json`: link commits automatically (`commit_linking` set to always), PII redaction enabled, summary generation disabled, and the checkpoint-push decision. Personal preferences (telemetry, log level, absolute hook path, private redaction rules) MUST live in the gitignored `.entire/settings.local.json`.

#### Scenario: Effective settings check

- **WHEN** a contributor runs `entire status --detailed`
- **THEN** the effective values match the committed team settings except for explicitly personal keys

### Requirement: Checkpoint visibility is a recorded decision

Checkpoints pushed with code are readable by anyone who can read the repository, and by anyone if it is public. Before checkpoints are pushed from a repo, the team SHALL record one choice: the repo is private, checkpoints go to a separate private checkpoint remote, or pushing sessions is disabled. Redaction is best-effort, so secrets and personal data MUST NOT be pasted into prompts.

#### Scenario: Public repo

- **WHEN** the repo's GitHub visibility is public
- **THEN** checkpoint pushing is disabled or routed to a private checkpoint remote before the first push

### Requirement: Agent commits are linked and pushed with code

Commits produced with agent assistance SHALL be linked to their checkpoint (link prompt answered always, or the committed setting), and checkpoints SHALL be pushed with the code so teammates can trace any change.

#### Scenario: Teammate traces agent-produced code

- **WHEN** a contributor runs `entire checkpoint explain` on a checkpoint or `entire why FILE[:LINE]` on code they did not write
- **THEN** they see the originating commit, prompt, transcript and session behind it without asking its author

### Requirement: Cross-agent handoff via sessions

Handoffs between contributors or agents SHALL go through Entire sessions (`entire session resume`, optionally with a branch name), not by pasting transcripts. A forced resume MUST NOT be used unless the team accepts that newer commits without checkpoints may be reset.

#### Scenario: Agent-to-agent handoff

- **WHEN** one contributor's session ends and another picks up the branch
- **THEN** the second contributor resumes by branch, Entire restores the session log, and no commits are lost

### Requirement: GitHub mirror for shared checkpoint access

The `TheFirstIstari/durhack` GitHub repo SHALL have an Entire mirror (GitHub stays upstream, the Entire GitHub App has access, region chosen deliberately), and contributors SHALL add or clone through the `entire` remote where faster reads help. A mirror MUST NOT be relied on to bypass GitHub authentication or branch rules.

#### Scenario: New machine joins mid-hackathon

- **WHEN** a teammate clones through the mirror or adds the `entire` remote
- **THEN** they can fetch code and checkpoints and use search and explain without any local session history

### Requirement: Search for past decisions

Contributors and their agents SHALL use `entire search "<query>"` (and `--code` for literal code text) to find prior decisions before re-deciding them, with the understanding that results reflect only indexed work.

#### Scenario: Find an earlier decision

- **WHEN** a contributor searches for why a decision was made
- **THEN** results include the related commits and sessions, or the contributor falls back to `entire checkpoint list` for work not yet indexed

### Requirement: Free tier only, with an explicit not-allowed list

The project SHALL depend only on free features: the CLI, checkpoints, session storage, search, mirrors, human-only trails and local continuous memory. It MUST NOT use paid agent features: `entire review`, review agents on trails or gates, custom agents, `entire recap` (until confirmed free), `entire checkpoint explain --generate`, summary generation, or memory extraction from past sessions. No team workflow, hook or CI job may invoke them.

#### Scenario: Credits exhausted or no card added

- **WHEN** any individual has no agent credit
- **THEN** checkpoint capture, search, resume, mirrors and all team workflows work unchanged

#### Scenario: CI inspection

- **WHEN** CI configuration and git hooks are audited
- **THEN** none invoke a paid Entire command

### Requirement: Trails are optional and human-only

If trails are used, they SHALL carry intent and human approvals only, and GitHub pull requests remain the merge path for mirrored repos. Gates that require agent review findings MUST NOT be enabled.

#### Scenario: Merging a change

- **WHEN** a change is ready to merge
- **THEN** merge needs only the GitHub pull request checks and human approval, and no Entire agent review

### Requirement: Local continuous memory is opt-in and contained

Entire's continuous memory (brain) MAY be used locally after the repo has at least one commit. Setup MUST decline memory extraction from past sessions, MUST NOT connect to a hosted repository without team agreement, and its generated guide (`.entire/agent-guide.md` and managed blocks in agent instruction files) SHALL be reviewed and committed so the team shares it.

#### Scenario: Enabling memory

- **WHEN** a contributor enables continuous memory
- **THEN** extraction from past sessions is declined, no hosted sync starts, and the generated guide is reviewed before commit
