# Global Guidelines

- Do not modify whitespace, formatting, or line endings in files unless explicitly asked
- Preserve trailing newlines — every file should end with exactly one newline; never strip it when editing
- Only change code directly related to the current task
- Do not add or remove blank lines in files you edit
- Do not reformat or restyle existing code
- Reuse existing methods, classes, and functions whenever possible instead of creating new ones
- Never commit or push `.claude/` directories or their contents to any remote repository
- Never include API keys, credentials, tokens, or personally identifiable information (PII) in prompts, file reads, or shared context
- Never add "Co-Authored-By: Claude" or any Claude attribution to commit messages
- Never modify commented-out code
- When replying to PR comments, write natural, human-sounding responses — conversational and casual like a fellow developer, not formulaic or agent-like

# User Context

- The user is new to object-oriented programming and the languages Dart and Go. When planning, explaining, and discussing code modifications, use clear and accessible language, explain OOP concepts and language-specific patterns when relevant, and avoid assuming deep familiarity with these languages.

## ---- BEGIN CODEBASE CONVENTIONS (auto-generated 2026-03-06) ----

# Project Structure

- Monorepo with: `buoy/` (Go backend), `boat/` (Flutter/Dart driver app), `compass/cst-app/` (Next.js/React family app), `vue/` (Vue 2 map app), `proto/` (shared protobuf definitions)
- Generated code lives in `gen/` directories per project — never edit generated files
- Root `proto/` folder is committed; generated `gen/` folders are gitignored
- Root Makefile orchestrates protobuf generation across all projects
- Project-specific conventions are in each subdirectory's `.claude/CLAUDE.md`

# General Conventions

- Semantic versioning (`MAJOR.MINOR.PATCH`) aligned across Buoy and Boat
- Trunk-based development on `develop` branch; squash-merge PRs
- Branch naming: `{Feature-Description}` in PascalCase with dashes (e.g., `Boat-Allow-bulk-student-updates`)
- Commit message format: `{Component}: {Description}. (#{PR_NUMBER})` (e.g., `Boat: Fix Flutter bindings issue (#381)`)
- Auth0 for authentication across all apps
- New Relic for monitoring/error tracking across backend and mobile
- i18n support across all apps (9+ languages)

## ---- END CODEBASE CONVENTIONS ----
