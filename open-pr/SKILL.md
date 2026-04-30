---
name: open-pr
description: Create a GitHub pull request for the current branch with a brief, human-readable description that focuses on why. Use when asked to open a PR, create a PR, or make a pull request.
disable-model-invocation: false
user-invocable: true
argument-hint: "[optional base branch]"
---

# Open PR

Create a GitHub PR for the current branch. Optimize the description for a human reviewer who has 30 seconds: lead with **why**, follow with a brief **how**, let the diff speak for the rest.

## Rules

- Never mention AI, Claude, Anthropic, or any attribution in the PR title or body.
- Title: **under 70 characters**, imperative mood, describes the change in plain language.
- Body has three sections in this order: **Context/Why**, **How**, **Test plan**. No "Summary" section, no file-by-file recap.
- **Context/Why**: 1–3 sentences (or up to 3 short bullets) explaining the motivation — the problem, the user impact, the bug being fixed, the constraint being addressed. If a single sentence covers it, use a single sentence.
- **How**: 1–3 sentences (or up to 3 short bullets) summarizing the **technical approach** at a high level — the strategy, the key decision, the shape of the solution. Not a file-by-file walkthrough. Mention specific files/functions only if a reviewer would otherwise miss the entry point. If the approach is obvious from the title, you can skip this section.
- Do **not** restate what the diff already shows in either section. Skip bullets like "added function X" or "updated file Y" — reviewers can read the diff. Only call out a *what* if it is non-obvious from the diff (a subtle behavior change, a deliberate non-change, a follow-up deferred).
- **Test plan**: short checklist. Automated checks (checked if passing), one or two manual verification steps (unchecked). Skip if the change genuinely needs no testing (e.g. a typo fix) — say so in one line instead.
- Plain prose over jargon. Write like a teammate, not a changelog generator.
- If `$ARGUMENTS` is provided, use it as the base branch. Otherwise detect with `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`.

## Process

1. Determine the base branch from `$ARGUMENTS` or the repo default.

2. Run these in parallel to understand the full scope:
   - `git log --oneline <base>..HEAD` — all commits
   - `git log <base>..HEAD` — full commit messages (often contain the *why*)
   - `git diff <base>...HEAD --stat` — file-level overview
   - `git diff <base>...HEAD` — full diff
   - `git status` — uncommitted changes

3. If there are uncommitted changes, warn the user and stop. PRs should only contain committed work.

4. If the branch has no commits beyond the base, say so and stop.

5. **Draft the PR.** Before writing, answer two questions in your head:
   - *Why does this change exist?* — pull from commit messages, linked issues, or the conversation. If you cannot articulate a clear why, ask the user rather than padding with bullets.
   - *What's the shape of the solution?* — the one-paragraph version a teammate would give at standup, not a list of files.

   Then:
   - **Title**: a human sentence, not a category. "Fix runway delta showing negative on first load" beats "Dashboard fixes."
   - **Context/Why**: lead with the user-visible problem or the motivation.
   - **How**: brief technical summary. Skip if title already conveys it.
   - **Test plan**: keep tight. 2–4 lines max for typical PRs.

6. Push the branch if needed (`git push -u origin <branch>` if no upstream).

7. Create the PR:
   ```
   gh pr create --base <base> --title "title" --body "$(cat <<'EOF'
   ## Context/Why

   Plain-prose explanation of the motivation. One short paragraph, or up to 3 bullets if there are distinct reasons.

   ## How

   One short paragraph (or up to 3 bullets) on the technical approach — the strategy, not the file list.

   ## Test plan

   - [x] Automated tests passing
   - [ ] Manual check: <one concrete step>
   EOF
   )"
   ```

8. Report the PR URL.

## Examples

### Good — bug fix

**Title:** `Fix runway delta showing negative on first load`

**Body:**
```
## Context/Why

The dashboard was rendering the runway delta before the previous-month
snapshot finished loading, so users saw a negative number flash before it
corrected itself. Reported by two users this week.

## How

Gate the delta render on the snapshot's loading state instead of falling
back to zero. The fallback was the source of the bad math.

## Test plan

- [x] Existing dashboard tests pass
- [ ] Manual: hard-refresh dashboard, confirm no negative flash
```

### Good — small feature

**Title:** `Add weekly check-in flow to dashboard`

**Body:**
```
## Context/Why

Users asked for a lightweight way to log progress without opening a full
review. The full review form is too heavy for a quick weekly note.

## How

A one-tap prompt on the dashboard that writes to the existing check-in
table — no schema change. Dismiss state stored in local prefs so the
prompt doesn't nag.

## Test plan

- [x] New tests in `check_in_test.ts` cover create + dismiss
- [ ] Manual: trigger prompt on Monday, confirm dismiss persists
```

### Good — refactor

**Title:** `Extract billing client so we can mock it in tests`

**Body:**
```
## Context/Why

Tests were hitting the live Stripe sandbox, which made CI flaky and slow.

## How

Pull the client behind an interface and add a fake implementation for
tests. Production wiring is unchanged — same client, same call sites.

## Test plan

- [x] Full suite passes against the new fake
- [x] Smoke-tested live path locally against Stripe sandbox
```

### Bad — what the skill should avoid

**Title:** `Updates to dashboard and billing modules`

**Body:**
```
## Summary

- Updated `Dashboard.tsx` to fix a rendering issue
- Modified `runway.ts` to handle the previous-month snapshot
- Refactored `billing/client.ts` into an interface
- Added a new file `billing/fake.ts`
- Updated tests in `check_in_test.ts`
- Bumped version in `package.json`

## Test plan

- [x] Tests
- [ ] Manual testing
```

Why it's bad: title is vague, body restates the diff, no motivation or strategy, test plan is empty calories. A reviewer learns nothing they couldn't get from `git diff`.

## Title examples

Good (specific, human):
```
Fix runway delta showing negative on first load
Add weekly check-in flow to dashboard
Extract billing client so we can mock it in tests
```

Bad (vague or category-only):
```
Updates
Various fixes and improvements
Dashboard changes
PR for issue #42
```
