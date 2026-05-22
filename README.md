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
bin/
  codex-with-claude-sync
scripts/
  sync-claude-to-codex
  sync-and-commit
  init-local-config
  install-codex-wrapper
  install
  doctor
```

## Fresh Install

Clone the repo wherever you keep personal tooling:

```bash
git clone git@github.com:molson253/ceso-ai-workflow-kit.git /path/to/ceso-ai-workflow-kit
cd /path/to/ceso-ai-workflow-kit
```

Create and edit the local sync config:

```bash
scripts/init-local-config
```

Edit `config.local.json` so each source path exists on your machine and each
destination path is where you want that source stored in the kit.

Install the generated Codex files:

```bash
scripts/sync-claude-to-codex
scripts/install
scripts/doctor
```

Install the Codex startup wrapper:

```bash
scripts/install-codex-wrapper
```

Add the stable wrapper alias to `~/.zshrc`:

```bash
alias codex="$HOME/.local/bin/codex-with-claude-sync"
```

Reload the shell:

```bash
source ~/.zshrc
```

## Workflow

Choose which Claude files and skills to sync in `config.local.json`. This file
is gitignored because every developer can have different local paths.

Update the kit from the configured Claude source files, generate Codex files,
install them, commit the changes, and push:

```bash
cd /path/to/ceso-ai-workflow-kit
scripts/sync-and-commit
```

Use a custom commit message:

```bash
cd /path/to/ceso-ai-workflow-kit
scripts/sync-and-commit "Update CESO workflow notes"
```

Run the steps manually when debugging:

```bash
cd /path/to/ceso-ai-workflow-kit
scripts/sync-claude-to-codex
scripts/install
scripts/doctor
```

## Conversation Commands

Tell Claude to update the CESO AI workflow kit. Claude should copy the files
listed in `config.local.json` into this repo, then run `scripts/sync-and-commit`.

Tell Codex to refresh the workflow kit and reload the ceso-labs guidance for the
current conversation. Codex should pull this repo if needed, run
`scripts/sync-claude-to-codex`, run `scripts/install`, then read the relevant
updated files for the active conversation.

When Codex refreshes the kit mid-conversation, it should report:

- The git result: pulled new commits or already up to date.
- The sync/install result: both scripts completed successfully.
- The verification result: current commit hash and the relevant changed content
  found in the installed Codex files under `~/.codex`.

## Startup Wrapper

Point your Codex shell alias at a wrapper that runs sync and install before
launching Codex. That keeps Codex updated whenever you start a new session.

```bash
alias codex="$HOME/.local/bin/codex-with-claude-sync"
```

On this machine, the alias lives in `~/.zshrc`.
