---
name: clean-branches
description: Deletes local branches whose remote branch no longer exists (marked "gone" after a fetch --prune), typically branches already merged on the remote. Use when the user wants to clean up stale local branches.
disable-model-invocation: true
argument-hint: "[--yes]"
allowed-tools: Bash(git *)
---

# Clean branches: delete local branches whose remote is gone

Hypothesis: a local branch whose remote branch was deleted (e.g. auto-deleted when its PR was merged) can be deleted locally.

Optional arguments: `$ARGUMENTS` (`--yes` skips the confirmation).

## Steps

1. **Sync**: `git fetch --prune` so that deleted remote branches are detected.

2. **Find candidates**:
   ```
   git for-each-ref --format '%(refname:short) %(objectname:short) %(upstream:track)' refs/heads
   ```
   Keep the branches whose tracking status is `[gone]`. Exclude the current branch (`git branch --show-current`) and the default branch (`git symbolic-ref --short refs/remotes/origin/HEAD`, falling back to `main`/`master`). If the current branch is `[gone]`, mention it as skipped.
   - If there are no candidates, say so and stop.

3. **Show the list**: one line per branch with its short hash and last commit subject (`git log -1 --format=%s <branch>`).

4. **Confirm**: ask for confirmation before deleting, unless `--yes` was passed.

5. **Delete**: `git branch -D <branch>` for each confirmed branch. `-D` is required because squash-merged branches are not seen as merged by Git.

6. **Summary**: list the deleted branches with their short hash, and remind that a deleted branch can be restored with `git branch <name> <hash>` (or via `git reflog`).

## Rules

- Never delete the current branch or the default branch.
- Only delete branches marked `[gone]`; never touch branches that still have a remote.
- Never touch remote branches.
- Never mention Claude or any AI assistant anywhere.
- Be concise.
