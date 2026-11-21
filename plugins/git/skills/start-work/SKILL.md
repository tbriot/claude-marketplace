---
name: start-work
description: Starts a new piece of work from an up-to-date default branch by creating a provisional `wip/<timestamp>` branch. The `ship` skill later renames it from the diff. Use when the user wants to start working on something new.
disable-model-invocation: true
allowed-tools: Bash(git *)
---

# Start work: up-to-date default branch → provisional branch

The branch name can't be chosen yet since nothing has changed. Create a provisional `wip/` branch; the `ship` skill renames it to `<type>/<branch-name>` from the diff before the first push.

## Steps

1. **Initial state**: `git status --short`, `git branch --show-current`, and determine the default branch (`git symbolic-ref --short refs/remotes/origin/HEAD`, falling back to `main`/`master`).
   - If there are uncommitted changes, stop and tell the user: they would be silently carried onto the new branch.

2. **Sync**: `git switch <default>` then `git pull --ff-only`.
   - If the pull is not a fast-forward, stop and report it (local default branch has diverged); never merge or reset on the user's behalf.

3. **Branch**: `git switch -c wip/$(date +%Y-%m-%d-%H%M)` (e.g. `wip/2026-11-21-1432`). If that name already exists, wait for the next minute or append `-2`.

4. **Summary**: print the branch name and remind the user to run `/git:ship` when done.

## Rules

- Never commit or push in this skill.
- Never mention Claude or any AI assistant anywhere.
- Be concise: no questions needed.
