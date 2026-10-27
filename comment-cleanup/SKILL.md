---
name: comment-cleanup
version: 1.0.0
description: Remove redundant, stale, and low-value code comments while preserving concise, non-obvious context. Use when asked to audit or clean up comments in a repository, branch, PR, staged or unstaged changes, or specific paths.
disable-model-invocation: false
user-invocable: true
argument-hint: "[natural-language scope, branch/base, PR, commit range, or path — plus optional audit]"
tags:
  - coding
  - code quality
---

# Comment Cleanup

Minimize the context needed to understand and maintain the code. Prefer self-explanatory code. Keep comments only when they preserve useful information that cannot reasonably be inferred from the code.

Ask of every comment: **What information would be lost if this disappeared?** Do not optimize for a deletion percentage.

## Scope

Resolve scope from the user's request and `$ARGUMENTS`, if supplied. Natural language is sufficient; the labels below are shortcuts, not required syntax. For example, "my uncommitted changes" means working-tree scope, "this PR" means the PR diff, and "throughout the repo" means repository scope.

- **Unstaged:** working-tree changes against the index, including relevant untracked source files.
- **Staged:** index changes against HEAD. Preserve unrelated unstaged edits.
- **Working tree:** staged, unstaged, and relevant untracked changes.
- **Branch:** committed changes from the branch's merge base with its base branch to its tip. Use the requested branch, or the current branch if none is named.
- **PR or commit range:** the requested diff. For a PR, use its actual base and head.
- **Repository:** all maintained first-party source and configuration.
- **Explicit files or directories:** the supplied paths.
- **Audit:** apply the selected scope but report proposed cleanup without editing.

Honor an explicit scope. Otherwise default to unstaged changes. If that scope is empty, report it; do not silently expand the scope. Resolve an ambiguous target or branch base before proceeding. A branch's remote-tracking copy is not necessarily its comparison base.

For diff scopes, review touched comment blocks and comments made stale by the changed code. Inspect surrounding code for context without automatically cleaning entire files. Do not include unrelated pending changes in a branch or PR scope. When the scoped version differs from the checkout, do not overwrite current work with an older snapshot.

Exclude dependencies, vendored code, build outputs, and unrelated prose documentation. Treat maintained generators and dependency patches as project source.

## Decide What Deserves to Remain

Delete comments that:

- Describe a file, component, function, or operation already clear from its name and implementation.
- Translate code into prose or add obvious section labels.
- Repeat types, tests, or nearby explanations without adding context.
- Contain TODOs, speculative implementation plans, or commented-out dead code.
- Recount implementation history, abandoned approaches, or resolved debugging sessions.
- Refer to demonstrably obsolete plans, systems, or migrations.
- Contradict the implementation or make unsupported claims.
- Add generic advice, filler, or boilerplate without a concrete constraint.

When a TODO or historical explanation contains a still-relevant constraint, preserve that fact without the task or history.

Keep concise explanations of:

- Business decisions whose reasons are absent from the code.
- External contracts and compatibility requirements.
- Non-obvious platform or dependency behavior.
- Reasons for workarounds, ordering requirements, or unusual tradeoffs that a future edit could otherwise violate.

Distinguish what the code does from why it must do it. An obvious condition may still have a non-obvious business reason.

Evaluate substance, not authorship. A platform reference or generated comment is not automatically obsolete. Do not assume a migration is complete or a plan is retired without evidence or user direction.

Shorten retained comments to the minimum wording that preserves the constraint. Place them next to the code they explain.

## Preserve Functional and Required Content

Do not blindly delete license notices, legal attribution, shebangs, compiler directives, or comments consumed by tools. Treat docstrings as potentially observable runtime or public API content.

For lint or type-check suppressions:

- Identify the actual violation.
- Make the smallest correct fix that permits removing the suppression.
- Preserve intended behavior, including side-effect timing.
- Do not weaken rules, hide the violation, or replace it with an unsafe cast or another bypass.
- If removal requires a broader redesign or an unresolved dependency exception, retain it and report the reason.

A stale comment alone is not authority to change product behavior. Resolve uncertainty using the implementation, tests, and relevant contracts.

## Apply the Cleanup

Use language-aware tooling where available. Text search can locate candidates; regex-only bulk deletion can corrupt strings, templates, regular expressions, or syntax.

Remove empty comment containers and leftover whitespace without reformatting unrelated code.

Update generators when they would recreate removed commentary. For maintained patches, preserve applicability and update required metadata or checksums.

Preserve unrelated user changes. For staged scope, update only the selected staged content while retaining unstaged edits. Other scopes must not alter staging automatically.

Do not create branches, commit, push, or rewrite repository guidance unless requested.

## Verify

Use checks proportional to the changes:

- Verify parsing and inspect the diff for accidental code changes.
- For broad automated edits, compare parsed code before and after where practical, and separately review every intended executable change.
- Run relevant lint, type, and behavioral checks for suppression fixes.
- Verify affected generators and dependency patches.
- Distinguish pre-existing failures from regressions.

Do not rewrite tests merely to accept unintended behavior changes.

## Report

Summarize the scope covered, what was removed, what information was retained, any executable changes needed to eliminate suppressions, validation results, and unresolved exceptions.

Include counts only when measured reliably. Avoid an exhaustive comment inventory unless requested.
