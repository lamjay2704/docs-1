# Upgrading from Nitro 2

::: warning
If you’re running a pre-release version of Nitro 3, be sure to first follow the instructions for Building from Source, Building the Proxy, and Building the Images in [Nitro’s readme](https://github.com/craftcms/nitro/tree/3.0#craft-nitro).
:::

1. Open `~/.nitro/nitro.yaml`.
2. Change each site’s `version:` key to `php_version:`.
3. Change `sites:` to `apps:`.
4. Run `nitro apply`.