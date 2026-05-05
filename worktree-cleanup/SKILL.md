---
name: worktree-cleanup
description: Safely audit, remove, and prune Git worktrees. Use when asked to list worktrees, clean up old worktrees, remove worktree directories, prune stale worktree metadata, or decide whether a worktree or branch is safe to remove after a merge or PR cleanup.
disable-model-invocation: false
user-invocable: true
argument-hint: "[optional repo path or worktree path]"
---

# Worktree Cleanup

Safely clean up extra Git worktrees while protecting uncommitted work and keeping branch deletion as a separate, explicit choice.

## Rules

- Treat the current checkout as the primary worktree unless the user says otherwise.
- Do not remove the primary/current worktree.
- Do not delete local or remote branches unless the user explicitly asks for branch deletion.
- Never use `rm -rf` as the first cleanup tool. Prefer `git worktree remove <path>` so Git unregisters metadata correctly.
- Never force-remove a dirty worktree unless the user has reviewed the exact dirty files and explicitly approves the loss.
- If a worktree directory is already gone but Git still lists it as prunable, use `git worktree prune`.
- Be clear when Git ancestry and file content disagree. A squash merge often leaves a branch "unmerged" by ancestry even when `git diff` is empty.

## Workflow

1. Inventory the repo:

```bash
git worktree list --porcelain
git status --short --branch
```

Summarize each worktree with its path, branch, HEAD, and whether it is the primary checkout.

2. Check each extra worktree before removal:

```bash
git -C <worktree-path> status --short --branch
```

If the status output contains anything beyond the `##` branch line, stop and report the dirty/untracked files. Ask before removing.

3. Check branch relationship to the base branch before saying it is safe:

```bash
git log --oneline <base>..<candidate-branch>
git log --oneline <candidate-branch>..<base>
git diff --stat <base> <candidate-branch>
git diff --name-status <base> <candidate-branch>
git cherry -v <base> <candidate-branch>
```

Use the repo's current primary branch as `<base>` unless context points to another integration branch. Explain the result plainly:

- Empty `git diff` means the file content matches.
- Non-empty `git log <base>..<branch>` means Git does not see those commits on the base branch by ancestry.
- A clean worktree plus empty diff to base is usually safe to remove, even if ancestry shows old commits from a squash merge.
- Non-empty diff means removal of the worktree directory may still be okay if clean, but the branch contains content not in base. Make that risk explicit.

4. Remove only after the safety summary:

```bash
git worktree remove <worktree-path>
git worktree list --porcelain
```

If removal deletes the directory but fails on stale metadata, run:

```bash
git worktree prune
git worktree list --porcelain
```

5. Optionally prune remote-tracking refs when the user believes a remote branch was deleted:

```bash
git fetch --prune origin
git branch -r --list origin/<branch-name>
```

Report whether the remote-tracking ref still exists. Do not delete the local branch unless asked.

## Reporting

Before removal, report:

- Which worktrees exist.
- Which ones are clean or dirty.
- Whether candidate branches are ahead/behind or content-identical to base.
- What will be removed and what will be left alone.

After removal, report:

- The final `git worktree list --porcelain` result in human terms.
- Whether local branches remain.
- Whether remote-tracking refs still exist if checked.
