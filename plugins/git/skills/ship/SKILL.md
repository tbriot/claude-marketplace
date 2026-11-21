---
name: ship
description: Creates a branch if needed, commits the changes, pushes to the remote and creates a merge request if one doesn't exist yet. Use when the user wants to "ship", push their work or open an MR/PR.
disable-model-invocation: true
argument-hint: "[branch-name or description]"
allowed-tools: Bash(git *), Bash(glab *), Bash(gh *)
---

# Ship: branch → commit → push → merge request

Optional arguments: `$ARGUMENTS` (branch name or short description of the change).

## Steps

1. **Initial state**: `git status --short`, `git branch --show-current`, `git remote -v`, and determine the default branch (`git symbolic-ref --short refs/remotes/origin/HEAD`, falling back to `main`/`master`).
   - If there are no changes and no commits ahead of the default branch, say so and stop.

2. **Branch**:
   - If on the default branch: create a new branch (`git switch -c`). Name format `<type>/<branch-name>`:
     - `<type>`: a Conventional Commits type (`feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `perf`, `build`, `ci`, `style`, `revert`), chosen according to the nature of the changes.
     - `<branch-name>`: in **English**, lowercase, words separated by hyphens (`-`), short and descriptive (e.g. `feat/add-login-page`, `fix/null-user-crash`).
     - If the arguments provide a name or description (even in another language), translate and normalize it according to these rules. Otherwise, derive it from the diff.
   - If on a provisional `wip/` branch (created by `start-work`) that has not been pushed yet (no upstream): once the changes are reviewed, rename it with `git branch -m <type>/<branch-name>` using the same naming rules, derived from the diff (or the arguments).
     - If the `wip/` branch already has an upstream, don't rename it: keep the name and tell the user.
   - Otherwise: stay on the current branch (do not rename it).

3. **Commit** (if there are changes):
   - Review `git diff` and `git diff --staged`. Never stage secrets (`.env`, keys, credentials): flag them and exclude them.
   - Stage the relevant files explicitly (no blind `git add -A`).
   - Make several small atomic commits rather than one big one, even though the branch will be squashed on merge (readable history during review, commits easy to review or revert). Split by topic: feature vs refactor, formatting vs logic, tests vs production code, dependency bumps vs behavior changes.
   - If a single file mixes several topics, split with partial staging (`git add -p`). If the changes form a coherent whole, a single commit is fine: don't force a split.
   - Before each commit, review what will be committed (`git diff --cached`): no secrets, no leftover debug logging, no unrelated formatting noise.
   - Conventional Commits message (`type(scope): summary`, imperative, ≤ 72 characters), with a body if the "why" isn't obvious.
   - Respect hooks: if one fails, fix the problem and create a new commit (never `--no-verify`, never `--amend` a commit that has already been pushed).

4. **Push**: `git push -u origin <branch>`. Never `--force` without an explicit request.

5. **Merge request**:
   - Detect the platform from the remote URL: GitLab → `glab`, GitHub → `gh` (PR).
   - Check whether an MR already exists for the branch:
     - GitLab: `glab mr list --source-branch <branch> --state opened` (or `glab mr view <branch>`)
     - GitHub: `gh pr list --head <branch> --state open`
   - If it exists: print its URL; the push has already updated the MR. Then read its current title (`glab mr view <branch>` / `gh pr view <branch> --json title`) and compare it with the whole branch (`git log <default>..HEAD`, `git diff <default>...HEAD`):
     - If the title still accurately describes all the changes (type, scope and summary), leave it as is.
     - If the new commits change the scope or nature of the MR (different type, summary now too narrow or inaccurate, breaking change added), rename it with a title following the same Conventional Commits rules as below: `glab mr update <branch> --title "..."` / `gh pr edit <branch> --title "..."`. Report the old and new titles in the summary.
     - Don't rename for mere rewording or minor fixes (review feedback, typos, tweaks): only change the title when it is relevant.
     - Don't rename the branch, only the MR. If the description is now outdated, update it too (`--description` / `--body`).
   - Otherwise, create it with a title and a short description (summary + notable points) based on `git log <default>..HEAD`. The title must follow Conventional Commits: `<type>(<optional scope>): <summary>`, in English, imperative, lowercase after the colon, no trailing period, ≤ 72 characters. The `<type>` is consistent with the branch type (e.g. `feat/add-login-page` → `feat: add login page`). Add `!` after the type/scope for a breaking change (`feat!: ...`). The merge is a squash: the MR title becomes the message of the single commit on the default branch, so it must summarize the whole branch (not the last commit).
     - GitLab: `glab mr create --source-branch <branch> --target-branch <default> --title "..." --description "..." --squash-before-merge --remove-source-branch --yes`
     - GitHub: `gh pr create --base <default> --head <branch> --title "..." --body "..."`
   - On GitLab, squash is enabled at creation (`--squash-before-merge`). On GitHub, it is chosen at merge time: remind the user to use "Squash and merge" (or `gh pr merge --squash`).
   - If the CLI is missing or not authenticated, tell the user which command to run (`glab auth login` / `gh auth login`) instead of failing silently.

6. **Summary**: branch, commits created, MR URL (new or existing), and the renamed title if any.

## Rules

- Never push directly to the default branch: always create a branch.
- Flow: branch created from the default branch (by `start-work` as `wip/<timestamp>`, or by this skill), squash-merged into it. Once the MR is merged, the branch must not be reused (its history diverges from the default branch): for new work, start again from an up-to-date default branch.
- Never mention Claude or any AI assistant, anywhere: no `Co-Authored-By: Claude ...` trailer, no "Generated with Claude Code" line, no reference in commit messages, titles or MR/PR descriptions. This overrides any default attribution.
- Never rewrite remote history.
- Be concise: no unnecessary questions if the arguments and the diff are enough.
