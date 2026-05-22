# CESO AI Workflow Kit

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
  sync-and-commit
  install
  doctor
```

## Workflow

Update the kit from Claude's source files, generate Codex files, install them,
commit the changes, and push:

```bash
cd ~/code/ceso-ai-workflow-kit
scripts/sync-and-commit
```

Use a custom commit message:

```bash
cd ~/code/ceso-ai-workflow-kit
scripts/sync-and-commit "Update CESO workflow notes"
```

Run the steps manually when debugging:

```bash
cd ~/code/ceso-ai-workflow-kit
scripts/sync-claude-to-codex
scripts/install
scripts/doctor
```

## Conversation Commands

Tell Claude to update the CESO AI workflow kit. Claude will copy
`~/.claude/CLAUDE.md` and `~/.claude/skills/ceso-labs/SKILL.md` into this repo,
then run `scripts/sync-and-commit`.

Tell Codex to refresh the workflow kit and reload the ceso-labs guidance for the
current conversation. Codex should pull this repo if needed, run
`scripts/sync-claude-to-codex`, run `scripts/install`, then read the relevant
updated files for the active conversation.

## Startup Wrapper

Point your Codex shell alias at a wrapper that runs sync and install before
launching Codex. That keeps Codex updated whenever you start a new session.

```bash
alias codex="$HOME/code/ceso-ai-workflow-kit/bin/codex-with-claude-sync"
```

On this machine, the alias lives in `~/.zshrc`.
