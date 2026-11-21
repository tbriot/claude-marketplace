# tbriot — Claude Code marketplace

Personal marketplace of Claude Code plugins.

## Installation

```
/plugin marketplace add tbriot/claude-marketplace
/plugin install git@tbriot
```

## Plugins

### `git`

Git workflow: branch, commits, push and merge request (squash).

- `/git:start-work` — pulls the default branch and creates a provisional `wip/<timestamp>` branch.
- `/git:ship [description]` — creates a `<type>/<name>` branch, makes small Conventional Commits, pushes, and opens (or updates) the MR/PR.
