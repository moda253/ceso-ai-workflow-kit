# AI Workflow Kit

Personal AI assistant workflow configuration for Claude and Codex.

Claude is the source of truth for now. The sync script converts Claude-side
rules and skills into Codex-compatible files, then the install script copies the
generated output into the local assistant config directories.

## Layout

```text
claude/
  CLAUDE.md
  skills/
codex/
  generated/
  skills/
scripts/
  sync-claude-to-codex
  install
  doctor
```

## Workflow

```bash
scripts/sync-claude-to-codex
scripts/install
scripts/doctor
```

## Startup Wrapper

Point your Codex shell alias at a wrapper that runs sync and install before
launching Codex. That keeps Codex updated whenever you start a new session.

