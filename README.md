# Integral Productivity Marketplace

A [Claude Code plugin marketplace](https://docs.claude.com/en/docs/claude-code/plugin-marketplaces) for publicly-available plugins maintained by [Integral-Productivity](https://github.com/Integral-Productivity). All plugins listed here live in public repositories and install without additional authentication setup, so this marketplace works in both [Claude Code](https://docs.claude.com/en/docs/claude-code) CLI and Claude Desktop (Cowork).

> **Looking for org-internal plugins?** Plugins that should stay inside Integral-Productivity live in [`Integral-Productivity/marketplace-internal`](https://github.com/Integral-Productivity/marketplace-internal) (private). That marketplace is CLI-only — see its README for SSH setup. Cowork in Claude Desktop cannot sync marketplaces that reference private or internal plugin repos; see [anthropics/claude-code#61271](https://github.com/anthropics/claude-code/issues/61271) for the open feature request.

## Catalog

| Plugin | Source | Description |
|---|---|---|
| `lean-management` | [Integral-Productivity/lean-management](https://github.com/Integral-Productivity/lean-management) | Lean management support for an organization |

## Install

In Claude Code (CLI) or Claude Desktop (Cowork):

```text
/plugin marketplace add Integral-Productivity/marketplace
/plugin install <plugin-name>@integral-productivity-tools
```

Then restart Claude Code so the plugin's skills, commands, hooks, and MCP servers are picked up.

## Adding a plugin to the marketplace

1. Confirm the plugin repo is **public**. Cowork's marketplace sync runs server-side without GitHub authentication and rejects any private or internal plugin source — list internal plugins in `marketplace-internal` instead.
2. Open a PR against this repo editing [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json).
3. Add an entry under `plugins` with `name`, `source` (use `{ "source": "github", "repo": "<owner>/<name>" }`), `version`, and `description`.
4. Pin `version` to a value that matches the plugin repo's `.claude-plugin/plugin.json` (or a git tag, once tagging is in place).
5. Merge to `main` — users refresh with `/plugin marketplace update integral-productivity-tools`.
