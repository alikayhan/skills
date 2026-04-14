---
name: commit-and-push
description: Stage unstaged changes into meaningful commits and push. Use when asked to commit and push, or land changes.
disable-model-invocation: false
user-invocable: true
argument-hint: "[optional commit scope or message hint]"
---

# Commit and Push

Create multiple meaningful commits from the current unstaged/staged changes and push to the remote.

## Rules

- Each commit must be a **self-contained logical unit** — it should make sense on its own if someone reads `git log`.
- Commit messages: **imperative mood, lowercase after prefix, 50-char subject line max**. No periods at end of subject. Body only if the "why" isn't obvious from the subject.
- Every commit message must start with a **type prefix**: `feat:`, `refactor:`, `fix:`, `chore:`, `cleanup:`, `docs:`, or `misc:`. Pick the one that best describes the intent of the commit.
- Never lump unrelated changes into one commit. Splitting too fine (one file per commit) is also wrong — group by intent, not by file.
- Never mention AI, Claude, Anthropic, or any attribution in commit messages.
- Never include generated or build artifacts unless they are checked-in project files that the team maintains.
- If there are no changes to commit, say so and stop.

## Process

1. Run `git status` and `git diff` (staged + unstaged) to understand all pending changes.

2. Run `git log --oneline -5` to match the repo's existing commit message style.

3. **Plan the commits** — mentally group changes by logical intent. Common groupings:
   - Renames/moves (do these first so diffs are clean)
   - Model/type changes
   - Implementation changes
   - Test infrastructure (fixtures, mocks, helpers)
   - Tests
   - Config/build system changes
   - Bug fixes
   
   If `$ARGUMENTS` is provided, use it as context for scoping or message hints.

4. **Execute commits** in dependency order (foundations first, things that depend on them after). For each commit:
   - `git add` only the specific files for that logical unit
   - `git commit -m "message"` — use a HEREDOC for multi-line messages
   - Verify the commit succeeded before moving to the next

5. After all commits, `git push` to the current remote tracking branch. If no upstream is set, push with `-u origin <branch>`.

6. Report: show the final `git log --oneline` for the new commits.

## Commit message examples

Good:
```
feat: add retry logic to API client timeout handling
fix: off-by-one in pagination cursor calculation
refactor: extract shared validation into FormValidator
chore: bump dependencies to latest patch versions
cleanup: remove unused auth middleware
docs: add setup instructions for local dev
misc: rename ProfileStore to AccountStore across modules
```

Bad:
```
update files
fix stuff
changes to multiple files across the codebase
refactor (with no indication of what was refactored)
feat (missing colon and description)
```
