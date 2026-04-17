# cortex-command-plugins

Optional Claude Code plugins that complement the [cortex-command](https://github.com/charleshall888/cortex-command) agentic workflow framework. These plugins package skills that were previously bundled with `cortex-command` but are useful independently and can be adopted per-project.

## Plugins

- **cortex-ui-extras** — UI-focused skills for working on frontend/dashboard code (see the UI tooling reference at `docs/ui-tooling.md` in `cortex-command` for the underlying conventions).
- **cortex-pr-review** — Skill for reviewing GitHub pull requests.
- **android-dev-extras** — Personal Android-development skills vendored from [github.com/android/skills](https://github.com/android/skills) (Apache 2.0). See [plugins/android-dev-extras/HOW-TO-SYNC.md](plugins/android-dev-extras/HOW-TO-SYNC.md) for refresh procedure.

## Relationship to cortex-command

The `cortex-command` repo remains the source of truth for the core agentic layer (lifecycle, backlog, commit/pr skills, hooks, overnight runner). This repo hosts **optional** skills that not every project wants installed globally. Install this marketplace and enable only the plugins you need per project.

## Install

Add this repo as a plugin marketplace in Claude Code:

```
claude /plugin marketplace add https://github.com/charleshall888/cortex-command-plugins
```

## Enable per project

In a project's `.claude/settings.json`, opt in to specific plugins:

```json
{
  "enabledPlugins": {
    "cortex-ui-extras@cortex-command-plugins": true,
    "cortex-pr-review@cortex-command-plugins": true
  }
}
```

Enable only the plugins relevant to that project.

## License

MIT — see [LICENSE](LICENSE).
