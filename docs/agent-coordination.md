# Agent coordination: Entire settings

Team settings live in the committed `.entire/settings.json`. Personal settings live in `.entire/settings.local.json`, which Git ignores and which overrides project values for the same path.

## Team settings (committed)

| Key | Value | Why |
|---|---|---|
| `commit_linking` | `"always"` | Link commits to checkpoints automatically |
| `redaction.pii.enabled` | `true` | Redaction is best-effort; never paste secrets into prompts |
| `strategy_options.summarize.enabled` | `false` | Summaries are a paid feature and not allowed |
| `strategy_options.push_sessions` | `false` | This repo is public, so checkpoints must not be pushed with code |

`push_sessions: false` is the checkpoint-visibility decision for this public repo. The team may instead route checkpoints to a private checkpoint remote (`strategy_options.checkpoint_remote`).

## Personal-only keys (`.entire/settings.local.json`)

Do not commit these:

- `telemetry`
- `log_level`
- `absolute_git_hook_path` (set to `true` for GUI Git clients)
- `redaction.custom_redactions` (private redaction rules)
- `external_agents` (honoured only in the local file)

Example:

```json
{
  "log_level": "debug",
  "redaction": {
    "custom_redactions": {
      "private_codename": "PROJECT_[A-Z]{4}"
    }
  }
}
```

Source: https://docs.entire.com/guides/configuration/settings-layers
