# Integral Productivity Marketplace

A [Claude Code plugin marketplace](https://docs.claude.com/en/docs/claude-code/plugin-marketplaces) for plugins maintained by [Integral-Productivity](https://github.com/Integral-Productivity). The marketplace metadata is public; individual plugin repositories may be **INTERNAL** or **PRIVATE** and require GitHub authentication to install.

## Catalog

| Plugin | Source | Visibility | Description |
|---|---|---|---|
| `lean-management` | [Integral-Productivity/lean-management](https://github.com/Integral-Productivity/lean-management) | INTERNAL | Lean management support for an organization |

## Prerequisites

To install any plugin from this marketplace you need GitHub credentials that can read the underlying plugin repository.

1. Authenticate with the GitHub CLI, granting at least `repo` scope (covers private + internal reads):
   ```sh
   gh auth login
   ```
2. Wire `gh` into git's credential chain so Claude Code's `git clone` calls reuse the same token:
   ```sh
   gh auth setup-git
   ```
3. Sanity-check access to a target repo (replace as needed):
   ```sh
   git ls-remote https://github.com/Integral-Productivity/lean-management.git HEAD
   ```

## Install

In Claude Code:

```text
/plugin marketplace add Integral-Productivity/marketplace
/plugin install <plugin-name>@integral-productivity-tools
```

Then restart Claude Code so the plugin's skills, commands, hooks, and MCP servers are picked up.

## Adding a plugin to the marketplace

1. Open a PR against this repo editing [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json).
2. Add an entry under `plugins` with `name`, `source` (use `{ "source": "github", "repo": "<owner>/<name>" }`), `version`, and `description`.
3. Pin `version` to a value that matches the plugin repo's `.claude-plugin/plugin.json` (or a git tag, once tagging is in place).
4. Merge to `main` — users refresh with `/plugin marketplace update integral-productivity-tools`.
