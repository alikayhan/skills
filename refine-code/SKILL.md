---
name: refine-code
description: Review changed code for reuse, quality, and efficiency, then fix any issues found. Use when asked to refine, polish, simplify, clean up, or do a final quality pass on code changes.
disable-model-invocation: false
user-invocable: true
argument-hint: "[optional focus]"
effort: max
---

# Refine Code

Review all changed files for reuse, quality, and efficiency. Fix any issues found.

If `$ARGUMENTS` is provided, treat it as additional focus for the review and cleanup pass.

## Rules

- Keep the scope tight: improve the current changes, not the whole codebase.
- Preserve existing behavior unless the user explicitly asked for a behavior change.
- Do not rewrite working code just to make it different.
- Prefer existing project conventions and utilities over new abstractions.
- If a finding is a false positive or not worth addressing, note it and move on.

## Phase 1: Identify Changes

Run `git status`, `git diff`, and `git diff --staged` to see what changed. If there are staged changes, also review `git diff HEAD` so the full working tree delta is visible.

If there are no git changes, review the most recently modified files that the user mentioned or that you edited earlier in the conversation.

## Phase 2: Run Three Review Passes

If the harness supports subagents or parallel tasks, launch all three review passes concurrently and pass each reviewer the full diff plus the optional focus. If not, perform the three passes yourself in sequence before editing.

### Pass 1: Code Reuse Review

For each change:

1. **Search for existing utilities and helpers** that could replace newly written code. Look for similar patterns elsewhere in the codebase — common locations are utility directories, shared modules, and files adjacent to the changed ones.
2. **Flag any new function that duplicates existing functionality.** Suggest the existing function to use instead.
3. **Flag any inline logic that could use an existing utility** — hand-rolled string manipulation, manual path handling, custom environment checks, ad-hoc type guards, and similar patterns are common candidates.

### Pass 2: Code Quality Review

Review the same changes for hacky patterns:

1. **Redundant state**: state that duplicates existing state, cached values that could be derived, observers/effects that could be direct calls
2. **Parameter sprawl**: adding new parameters to a function instead of generalizing or restructuring existing ones
3. **Copy-paste with slight variation**: near-duplicate code blocks that should be unified with a shared abstraction
4. **Leaky abstractions**: exposing internal details that should be encapsulated, or breaking existing abstraction boundaries
5. **Stringly-typed code**: using raw strings where constants, enums, string unions, or branded types already exist in the codebase
6. **Unnecessary JSX nesting**: wrapper Boxes/elements that add no layout value — check if inner component props such as `flexShrink` and `alignItems` already provide the needed behavior
7. **Nested conditionals**: ternary chains (`a ? x : b ? y : ...`), nested if/else, or nested switch 3+ levels deep — flatten with early returns, guard clauses, a lookup table, or an if/else-if cascade
8. **Unnecessary comments**: comments explaining WHAT the code does, narrating the change, or referencing the task/caller — delete them; keep only non-obvious WHY such as hidden constraints, subtle invariants, or workarounds

### Pass 3: Efficiency Review

Review the same changes for efficiency:

1. **Unnecessary work**: redundant computations, repeated file reads, duplicate network/API calls, N+1 patterns
2. **Missed concurrency**: independent operations run sequentially when they could run in parallel
3. **Hot-path bloat**: new blocking work added to startup or per-request/per-render hot paths
4. **Recurring no-op updates**: state/store updates inside polling loops, intervals, or event handlers that fire unconditionally — add a change-detection guard so downstream consumers are not notified when nothing changed. Also, if a wrapper function takes an updater/reducer callback, verify it honors same-reference returns or the project's established no-change signal
5. **Unnecessary existence checks**: pre-checking file/resource existence before operating (TOCTOU anti-pattern) — operate directly and handle the error
6. **Memory**: unbounded data structures, missing cleanup, event listener leaks
7. **Overly broad operations**: reading entire files when only a portion is needed, loading all items when filtering for one

## Phase 3: Fix Issues

Aggregate the findings and fix each issue directly. Make focused edits that preserve behavior and improve the changed code.

After editing:

1. Re-run `git diff` to review your own changes.
2. Run the most relevant lightweight validation available in the project, if obvious from the repo (for example, typecheck, lint, or targeted tests). If validation is expensive or unclear, skip it and say why.
3. Briefly summarize what was fixed, or confirm the code was already clean.
