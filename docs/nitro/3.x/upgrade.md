# Upgrading from Nitro 2

::: warning
If you’re running a pre-release version of Nitro 3, be sure to first follow the instructions for Building from Source, Building the Proxy, and Building the Images in [Nitro’s readme](https://github.com/craftcms/nitro/tree/3.0#craft-nitro).
:::

1. Open `~/.nitro/nitro.yaml`.
2. Change each site’s `version:` key to `php_version:`.
3. Change `sites:` to `apps:`.
4. Run `nitro apply`.

## What’s New in Nitro 3

- `sites` are now `apps`.
- Each project can optionally provide its own `nitro.yaml` to be included by the global `~/.nitro/nitro.yaml`.
- Site name arguments now use an `--app` argument instead.
- Containers share a common home folder.
- The `db import` command is now `db restore`.
- `nitro npm` and `nitro composer` commands execute in their app containers.
- Contextual path detection has been improved.
- Image processing libraries are preinstalled.
- `node`/`npm` support has drastically improved.