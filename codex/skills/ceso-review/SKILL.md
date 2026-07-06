<!-- GENERATED FROM claude/skills/ceso-review. Do not edit directly. -->

---
name: ceso-review
description: Read-only CESO branch and pull request code review workflow. Use when the user asks to review a branch, PR, diff, or code changes for logic, correctness, implementation quality, unnecessary complexity, project rules, formatting, tests, database safety, transaction boundaries, generated code, observability, client impact, or deploy safety; especially phrases like "run ceso-review", "review this branch", "review this PR", "check my changes", or "findings only".
---

# CESO Review

## Review Contract

Run a code-review pass by default. Modify no files unless the user explicitly asks for fixes. Prefer read-only commands and report findings only.

Only call out issues that are introduced by, modified by, or materially exposed by the branch/PR diff. Read surrounding code freely for context, but do not report pre-existing unrelated problems as findings. If a pre-existing issue is important context, mention it only under assumptions or residual risk and clearly label it as pre-existing.

Lead with findings ordered by severity. Use precise file and line references. Keep summaries brief and secondary. If no issues are found, say that clearly and mention residual risk or test gaps.

Respect dirty worktrees. Treat existing uncommitted or untracked files as user work. Do not revert, format, regenerate, or clean up files during review.

## Workflow

1. Establish the review target.
   - For a branch review, identify the current branch and compare against the best available base, usually `develop`, upstream tracking branch, `origin/develop`, or the PR base if available.
   - For a PR review, use `gh pr view`, `gh pr diff`, and `gh pr checks` when available and approved by existing permissions.
   - If the base is ambiguous and materially affects the review, state the assumption in the final response.

2. Inspect the changed surface.
   - Run `git status --short`, `git diff --stat`, and the relevant `git diff` or PR diff.
   - Include tracked and relevant untracked files when they appear connected to the change.
   - Read surrounding code, not only the diff, before judging behavior.
   - Anchor every finding to changed lines or to unchanged lines whose behavior is directly affected by the changed lines.

3. Review with these lenses.
   - Logic and correctness: state transitions, edge cases, nil/empty values, error paths, old app/version behavior, and idempotency.
   - Race conditions and transaction boundaries: check whether multi-step decisions are atomic, especially duplicate prevention, route ownership, assignment, completion, and status changes.
   - Database enforcement vs app-only checks: flag behavior that should be backed by constraints, indexes, locks, or transactional error handling.
   - Query performance: look for missing indexes, growing-table scans, N+1 behavior, expensive joins, and changed query selectivity.
   - Implementation quality: prefer existing project patterns; flag bloated code, unnecessary abstractions, repeated logic, unclear names, and avoidable branching.
   - Project rules and formatting: check language formatters, lint conventions, SQL formatting, generated-code expectations, and naming consistency.
   - Generated code: verify whether sqlc, proto, mocks, or localization outputs need regeneration and whether generated files are tracked or intentionally ignored.
   - Error contracts: check Connect/HTTP codes, message stability, client expectations, localization needs, and whether new errors are actionable.
   - Observability: review logs, metrics, New Relic events, context propagation, noise level, and sensitive data exposure.
   - Tests: judge whether tests prove behavior and important failure modes, not only whether mocks are satisfied. Look for integration-test needs when service/database behavior changes.
   - Client impact: consider Boat, Compass, Postman collections, proto contracts, and old mobile versions when backend behavior changes.
   - Migration and deploy safety: check migration ordering, rollback behavior, partial deploys, indexes/constraints on existing data, and production data compatibility.
   - Security and privacy: check authorization, tenant/vendor boundaries, identifiers, secrets, and PII in logs or telemetry.

4. Validate when feasible without modifying files.
   - Use targeted tests first, then broader tests if the blast radius warrants it.
   - Use dry checks such as `git diff --check`, formatter list/diff modes, `go test`, `flutter analyze`, `npm test`, or repo-native commands when appropriate.
   - Do not run commands that rewrite files, regenerate code, install dependencies, or start destructive processes unless the user explicitly asks.
   - If a formatter would change files, report that and include the relevant formatter diff or file list.

5. Report clearly.
   - Findings first, ordered by severity: Critical, High, Medium, Low.
   - Each finding must be branch/PR-caused or branch/PR-exposed; skip unrelated legacy issues.
   - Each finding must include a clickable local file link with a single line number when possible.
   - Explain the impact and why it matters. Keep suggested fixes concise.
   - Include open questions or assumptions after findings.
   - Include verification commands run and their outcomes.
   - Mention files were not modified.

## Output Shape

Use this structure unless the review is tiny:

```markdown
Findings:

1. **High: concise title**
   [file.ext](/abs/path/file.ext:123) explains the problem, impact, and suggested direction.

Open questions / assumptions:
- Base branch assumed to be `origin/develop`.

Verification:
- `command` passed/failed, with key detail.

No files were modified.
```

If there are no findings:

```markdown
I did not find any blocking issues.

Residual risk:
- Mention untested areas or assumptions.

Verification:
- `command` passed.

No files were modified.
```
