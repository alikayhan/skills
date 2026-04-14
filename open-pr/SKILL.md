---
name: open-pr
description: Create a GitHub pull request for the current branch with a well-structured description. Use when asked to open a PR, create a PR, or make a pull request.
disable-model-invocation: false
user-invocable: true
argument-hint: "[optional base branch]"
---

# Open PR

Create a GitHub pull request for the current branch against the base branch.

## Rules

- Never mention AI, Claude, Anthropic, or any attribution in the PR title or body.
- PR title: **under 70 characters**, imperative mood, descriptive of the overall change.
- PR body must include a **Summary** section with bullet points and a **Test plan** section with a checklist.
- Summary bullets should explain **what changed and why** — not just list files.
- Test plan should include both automated tests (mark as checked if passing) and manual verification steps (leave unchecked).
- If `$ARGUMENTS` is provided, use it as the base branch. Otherwise detect the base branch from the repo (e.g. `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`).

## Process

1. Determine the base branch from `$ARGUMENTS` or the repo default.

2. Run these in parallel to understand the full scope:
   - `git log --oneline <base>..HEAD` to see all commits
   - `git diff <base>...HEAD --stat` for a file-level overview
   - `git diff <base>...HEAD` for the full diff
   - `git status` to check for uncommitted changes

3. If there are uncommitted changes, warn the user and stop. PRs should only contain committed work.

4. If the branch has no commits beyond the base, say so and stop.

5. **Draft the PR** by reading all commits and the full diff:
   - Title: concise summary of the overall change
   - Summary: 3-6 bullet points covering the key changes, grouped by theme
   - Test plan: list automated tests that cover the changes (checked), plus manual verification steps (unchecked)

6. Push the branch if needed (`git push -u origin <branch>` if no upstream).

7. Create the PR:
   ```
   gh pr create --base <base> --title "title" --body "$(cat <<'EOF'
   ## Summary
   
   - bullet points
   
   ## Test plan
   
   - [x] Automated tests passing
   - [ ] Manual verification steps
   EOF
   )"
   ```

8. Report the PR URL.

## Title examples

Good:
```
Improve dashboard copy and fix runway delta bugs
Add weekly check-in flow to dashboard
Fix profile screen horizontal scroll
```

Bad:
```
Updates
Various fixes and improvements
PR for issue #42
```
