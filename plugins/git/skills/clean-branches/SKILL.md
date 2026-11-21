---
name: clean-branches
description: Deletes local branches whose remote branch no longer exists (marked "gone" after a fetch --prune), typically branches already merged on the remote. Use when the user wants to clean up stale local branches.
disable-model-invocation: true
allowed-tools: Bash(git *)
---

# Clean branches: delete local branches whose remote is gone

Hypothesis: a local branch whose remote branch was deleted (e.g. auto-deleted when its PR was merged) can be deleted locally.

## Steps

1. **Switch to the default branch**: determine it (`git symbolic-ref --short refs/remotes/origin/HEAD`, falling back to `main`/`master`) and `git switch <default>` if not already on it, so that the previous current branch can be cleaned too.
   - If there are uncommitted changes, don't switch: stay on the current branch (it will be excluded from the candidates) and tell the user.

2. **Sync**: `git fetch --prune` so that deleted remote branches are detected.

3. **Find candidates**:
   ```
   git for-each-ref --format '%(refname:short) %(objectname:short) %(upstream:track)' refs/heads
   ```
   Keep the branches whose tracking status is `[gone]`. Exclude the current branch (`git branch --show-current`) and the default branch. If the current branch is `[gone]` (the switch was skipped), mention it as skipped.
   - If there are no candidates, say so and stop.

4. **Delete**: `git branch -D <branch>` for every candidate, without asking for confirmation. `-D` is required because squash-merged branches are not seen as merged by Git. Record each branch's short hash and last commit subject (`git log -1 --format=%s <branch>`) before deleting it.

5. **Summary**: list the deleted branches with their short hash, and remind that a deleted branch can be restored with `git branch <name> <hash>` (or via `git reflog`).

## Rules

- Never delete the current branch or the default branch.
- Never switch branches when there are uncommitted changes.
- Only delete branches marked `[gone]`; never touch branches that still have a remote.
- Never touch remote branches.
- Never mention Claude or any AI assistant anywhere.
- Don't ask questions: delete every candidate.
- Be concise.
