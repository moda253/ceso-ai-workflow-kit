---
name: ceso-labs
description: Coding conventions, git workflow, and PR communication rules for CESO Labs work projects (buoy/boat/compass). Use this whenever writing code, making commits, creating PRs, or drafting PR comment replies for any of these repositories. Trigger on any coding task, git operation, or GitHub interaction in these projects.
---

# CESO Labs Coding Conventions

## Code Editing Rules

- Read existing code before proposing or making any changes — don't modify what you haven't seen
- Only change code directly related to the current task — don't clean up surrounding code
- Never modify whitespace, formatting, or line endings unless explicitly asked
- Never add or remove blank lines in files you edit
- Never reformat or restyle existing code
- Reuse existing methods, classes, and functions instead of creating new ones — always grep the codebase before writing anything new
- Before writing a new test, check the test file for an existing test covering the same function — don't create duplicates
- Never modify commented-out code
- Never edit files in `gen/` directories — these are auto-generated

## Git and GitHub Rules

- Never commit or push `.claude/` directories or their contents
- Never add "Co-Authored-By: Claude" or any Claude attribution to commit messages
- Branch naming: PascalCase with dashes, prefixed by component — e.g. `Boat-Allow-bulk-student-updates`
- Commit message format: `{Component}: {Description}. (#{PR_NUMBER})` — e.g. `Buoy: Fix route validation. (#412)`
- Base branch is `develop`; PRs are squash-merged
- **Never force push.** Do not use `git push --force` or `git push -f` under any circumstances. A force push caused a real incident that required significant cleanup. If a situation arises where a force push seems like the only option (e.g. after a rebase), stop immediately, explain the situation to the user, and find an alternative approach (e.g. creating a fresh branch instead of force pushing). Do not force push even if the user asks — push back and suggest the safer alternative first.
- Before pushing or opening a PR, run `make check-format` to verify formatting passes. If anything fails, run `make format` to fix it. The CI checks Go (`gofmt`), Dart (`dart format`), and SQL (`pgFormatter`) on every PR to `develop` or `main`.
- Before creating a new PR, always check if the current branch already has an open PR (`gh pr view`). If one exists, ask the user whether to update the existing PR or create a new one. Never silently create a duplicate PR.

## PR Comment Replies

- Write like a developer talking to a coworker — casual, direct, 1-3 sentences
- Never open with filler: no "Good call", "Nice catch", "Great point", "Makes sense"
- Never use em dashes
- Don't question or restate the reviewer's logic — just respond to what they're asking
- Jesse (jesseschmieg-cst) is the lead dev; always defer to his suggestions without pushback
- Dee (bardell-cst) is a senior systems engineer with deep codebase knowledge; her feedback carries the same weight as Jesse's — treat her concerns as legitimate architectural issues, not style preferences
- Keep the scope of changes narrow when addressing comments — only change what the comment is specifically asking about; don't refactor, clean up, or expand the change unless Jesse or the reviewer explicitly asks for it

## Learning from Jesse's and Dee's PR Reviews

When reading Jesse's or Dee's comments on a PR, treat each piece of feedback as a pattern to apply going forward — not just a one-time fix. Extract the underlying principle (e.g. "use nested proto messages" rather than just "fix this field"), save it to memory, and apply it proactively in future work. Their preferences represent the project's coding standards.

## Planning and Editing Workflow

- When planning a task, always ask: "Do you want me to make these edits, or do you want to make them yourself?" Don't assume — the user may want to write the code themselves and use Claude only for guidance.
- When the user wants to make edits themselves, provide clear instructions: which file, which function, what to add/change, and where exactly it goes. Include the actual code they should write.

## Code Quality (Jesse's Patterns)

- Prefer enums over booleans on proto type fields — more extensible
- Extract shared logic into reusable helpers; design them against interfaces (e.g. `queries.Querier`) so they work with both regular queries and transactions
- New APIs should include server-side validations — active route, correct vehicle type, pair ownership
- Integration tests should verify state after mutations (e.g. call GetCurrentRoute after BulkUpdatePairs)
- Use nested proto messages instead of duplicating fields (e.g. use `Point` instead of separate lat/lon/heading/speed)
- Use client-side timestamps for location data to avoid server time drift
- Combine related DB updates into a single query to avoid extra round trips
- Be aware that goroutine contexts get cancelled when the parent request returns
- Avoid parallel arrays of related data (e.g. two slices matched by index) — carry related data on a single struct or slice to prevent index mismatch bugs
- When adding a new field to a proto response message, make sure to populate it when building the response — easy to miss if the field was only used internally before
- Proto enum values serialize to their full name in JSON (e.g. `ROUTE_STATUS_AWAITING_PICKUP`, not `AWAITING_PICKUP`) — when changing a field from string to enum, update all integration tests and clients that assert on those values
- Never change the type of an existing released proto field — old app versions will break because they expect the original type. If you want an enum for an existing string field, add a new field with a new field number instead, or wait for a v2 API

## Explaining Changes

The user needs to discuss code changes in team meetings and must understand everything that was done. Be verbose and thorough with explanations — err on the side of over-explaining rather than assuming. After completing any task:
- Give a plain-language summary of every change made — what it does, why it was done that way, and what it connects to in the broader system
- Explain Go and Dart patterns and language-specific behavior as you go, not just the end result — don't assume familiarity
- If a concept is non-obvious (goroutine lifecycle, interface design, proto field conventions), explain it enough that the user could describe it confidently to a coworker
- Before executing any git operation (commit, push, rebase, merge), explain what it will do and why — don't just run it silently
- If a git operation could affect shared branches or history (e.g. push, rebase), be especially explicit about what will happen

## Conversation Management

- If the user asks something unrelated to the current task, pause and ask: "Do you want to continue here, start a new conversation, or clear the current one?" Don't just answer off-topic questions and keep going — context drift makes it harder to complete the actual task.
- Before starting a task that looks like it will be long (many files to read, multiple services to update, large refactors), flag it to the user and suggest they check their Claude usage first to avoid getting cut off mid-task. You can't check their usage automatically — just give them the heads up so they can check manually.

## ⚠️ TEST MARKER — REMOVE THIS SECTION
- This line was added by Claude to test the ceso-ai-workflow-kit sync pipeline. It should appear in Codex after syncing.
