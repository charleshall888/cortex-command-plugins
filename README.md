# cortex-command-plugins (archived)

This repository has been sunset. All plugins previously hosted here have been vendored into the [cortex-command](https://github.com/charleshall888/cortex-command) marketplace, which is now the single source of truth.

## Migration

If you previously installed plugins from this marketplace, run the steps below in order. **Uninstall first, then install** — running both marketplaces simultaneously produces unspecified slash-command resolution behavior when skills collide.

For `android-dev-extras`:

```
/plugin uninstall android-dev-extras@cortex-command-plugins
/plugin install android-dev-extras@cortex-command  # new marketplace
```

For `cortex-dev-extras`:

```
/plugin uninstall cortex-dev-extras@cortex-command-plugins
/plugin install cortex-dev-extras@cortex-command  # new marketplace
```

If you have not already added the cortex-command marketplace, add it first:

```
/plugin marketplace add https://github.com/charleshall888/cortex-command
```

## Where things went

- `android-dev-extras` → vendored into `cortex-command`
- `cortex-dev-extras` → vendored into `cortex-command`

See the [cortex-command README](https://github.com/charleshall888/cortex-command) for the full plugin catalog and installation instructions.
