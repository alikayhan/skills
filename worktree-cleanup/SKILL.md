---
name: worktree-cleanup
version: 1.0.0
description: Safely audit, remove, and prune Git worktrees. Use when asked to list worktrees, clean up old worktrees, remove worktree directories, prune stale worktree metadata, or decide whether a worktree or branch is safe to remove after a merge or PR cleanup.
disable-model-invocation: false
user-invocable: true
argument-hint: "[optional repo path or worktree path]"
tags:
  - coding
  - git
---

# Worktree Cleanup

Safely clean up extra Git worktrees while protecting uncommitted work and keeping branch deletion as a separate, explicit choice.

If `$ARGUMENTS` is provided, prefix every `git` command in this skill with `git -C $ARGUMENTS`. After step 1's inventory, if `$ARGUMENTS` matches a non-primary worktree path in the listing, scope steps 2–4 to that single worktree only. Otherwise treat `$ARGUMENTS` as the repo root and proceed with the full inventory.

## Rules

- Treat the current checkout as the primary worktree unless the user says otherwise.
- Do not remove the primary/current worktree.
- Do not delete local or remote branches unless the user explicitly asks for branch deletion.
- Rely on `git worktree remove`'s built-in safety net — by default it refuses dirty or locked worktrees and requires `-f` to override. Only reach for `-f` after the user has reviewed the exact dirty files and explicitly approved the loss.
- If a worktree directory is already gone but Git still lists it, use `git worktree prune`.
- If `git worktree remove` reports a lock, surface the lock reason from `git worktree list --porcelain` and ask before running `git worktree unlock`.
- Be clear when Git ancestry and file content disagree. A squash merge often leaves a branch "unmerged" by ancestry even when `git diff` is empty.

## Workflow

1. Inventory the repo:

   ```bash
   git worktree list --porcelain
   git status -sb
   ```

   Summarize each worktree with its path, branch (or detached HEAD), HEAD, locked status, and whether it is the primary checkout.

2. For each extra worktree, check cleanliness — this alone determines removal safety:

   ```bash
   git -C <worktree-path> status -sb
   ```

   If the output contains anything beyond the `##` branch line, stop and report the dirty/untracked files. Ask before removing. Removing a worktree directory does not touch the branch ref, so a clean worktree is safe to remove regardless of branch state.

3. Optionally summarize the branch's content relationship to base, so the user can decide separately whether to delete the branch later. Skip this step for detached-HEAD worktrees.

   Determine the base branch:

   ```bash
   gh repo view --json defaultBranchRef -q .defaultBranchRef.name 2>/dev/null \
     || git symbolic-ref --short refs/remotes/origin/HEAD | sed 's@^origin/@@'
   ```

   Then:

   ```bash
   git log --oneline <base>..<candidate-branch>
   git diff --stat <base> <candidate-branch>
   git cherry -v <base> <candidate-branch>
   ```

   Explain the results plainly:

   - Empty `git diff --stat` means the branch's tip content matches base.
   - Non-empty `git log <base>..<branch>` means Git ancestry shows commits not in base — common after a squash merge.
   - `git cherry -v` marks each commit `+` (not yet upstream) or `-` (patch already in base), which catches squash-merged content.
   - Clean worktree + empty diff to base = both worktree and branch can be safely discarded.
   - Clean worktree + non-empty diff = worktree is still safe to remove, but the branch carries content not in base; let the user decide whether to keep it.

4. Remove only after the safety summary:

   ```bash
   git worktree remove <worktree-path>
   git worktree list --porcelain
   ```

   If removal errors because the directory is already gone, run:

   ```bash
   git worktree prune
   git worktree list --porcelain
   ```

   If removal errors because the worktree is locked, report the lock reason and stop. Only run `git worktree unlock <worktree-path>` after the user confirms.

## Optional follow-ups

Run only if the user asks — these are adjacent to worktree cleanup but not part of it.

- **Prune remote-tracking refs** when the user believes a remote branch was deleted:

   ```bash
   git fetch --prune origin
   git branch -r --list origin/<branch-name>
   ```

   Report whether the remote-tracking ref still exists. Do not delete the local branch unless asked.

## Reporting

Before removal, report:

- Which worktrees exist (path, branch or detached HEAD, locked status).
- Which ones are clean or dirty.
- For branch-checked-out worktrees: whether the branch is ahead/behind or content-identical to base.
- What will be removed and what will be left alone.

After removal, report:

- The final `git worktree list --porcelain` result in human terms.
- Whether local branches remain.
- Whether remote-tracking refs still exist if checked.
