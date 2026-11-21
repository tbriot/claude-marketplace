# tbriot — Claude Code marketplace

Personal marketplace of Claude Code plugins.

## Installation

```
/plugin marketplace add tbriot/claude-marketplace
/plugin install git@tbriot
```

## Updating

```
/plugin marketplace update tbriot
/reload-plugins
```

The first command pulls the latest marketplace from GitHub and updates the installed plugins; the second loads them in the current session.

Plugins are cached by version. If `/plugin` answers "already at the latest version" after a change, the `version` in the plugin's `.claude-plugin/plugin.json` was not bumped: bump it with every change to a plugin (new or modified skill) so that the update is picked up.

## Plugins

### `git`

Git workflow: branch, commits, push and merge request (squash).

- `/git:start-work` — pulls the default branch and creates a provisional `wip/<timestamp>` branch.
- `/git:ship [description]` — creates a `<type>/<name>` branch, makes small Conventional Commits, pushes, and opens (or updates) the MR/PR.
- `/git:clean-branches` — switches to the default branch, then deletes all local branches whose remote branch no longer exists.
