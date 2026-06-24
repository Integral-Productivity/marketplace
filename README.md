# Integral Productivity Marketplace

A [Claude Code plugin marketplace](https://docs.claude.com/en/docs/claude-code/plugin-marketplaces) for publicly-available plugins maintained by [Integral-Productivity](https://github.com/Integral-Productivity). All plugins listed here live in public repositories and install without additional authentication setup, so this marketplace works in both [Claude Code](https://docs.claude.com/en/docs/claude-code) CLI and Claude Desktop (Cowork).

> **Looking for org-internal plugins?** Plugins that should stay inside Integral-Productivity live in [`Integral-Productivity/marketplace-internal`](https://github.com/Integral-Productivity/marketplace-internal) (private). That marketplace is CLI-only — see its README for SSH setup. Cowork in Claude Desktop cannot sync marketplaces that reference private or internal plugin repos; see [anthropics/claude-code#61271](https://github.com/anthropics/claude-code/issues/61271) for the open feature request.

## Catalog

_The public core catalog is currently empty._ The previously listed non-core plugins have moved to Integral-Productivity's labs marketplace, and org-internal plugins live in [`Integral-Productivity/marketplace-internal`](https://github.com/Integral-Productivity/marketplace-internal). New core plugins will be listed here as they are published — see [Adding a plugin to the marketplace](#adding-a-plugin-to-the-marketplace) below.

## Install

In Claude Code (CLI) or Claude Desktop (Cowork):

```text
/plugin marketplace add Integral-Productivity/marketplace
/plugin install <plugin-name>@integral-productivity-tools
```

Then restart Claude Code so the plugin's skills, commands, hooks, and MCP servers are picked up.

## Adding a plugin to the marketplace

This marketplace publishes plugins through a `stable` release channel rather than pinning specific versions — see [`holacracy-claude-plugin/docs/adr/0002`](https://github.com/Integral-Productivity/holacracy-claude-plugin/blob/main/docs/adr/0002-use-tag-driven-stable-branch-for-marketplace-channel-publication.md) for the reasoning. To list a new plugin:

1. Confirm the plugin repo is **public**. Cowork's marketplace sync runs server-side without GitHub authentication and rejects any private or internal plugin source — list internal plugins in `marketplace-internal` instead.
2. Confirm the plugin repo has a `stable` branch maintained by a tag-driven promotion workflow. See [`holacracy-claude-plugin/.github/workflows/promote-stable.yml`](https://github.com/Integral-Productivity/holacracy-claude-plugin/blob/main/.github/workflows/promote-stable.yml) for the template.
3. Open a PR against this repo editing [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json). Add an entry under `plugins` with `name`, `source` (`{ "source": "github", "repo": "<owner>/<name>", "ref": "stable" }`), and `description`. **Do not include a `version` field** — the marketplace tracks the plugin's `stable` channel; the plugin's own `.claude-plugin/plugin.json` is the source of truth for the user-visible version.
4. Merge to `main` — users refresh with `/plugin marketplace update integral-productivity-tools`.

### Release process for plugin maintainers

Once a plugin is listed and the promotion workflow is wired up, a release is:

1. Bump `version` in the plugin repo's `.claude-plugin/plugin.json` on a feature branch.
2. Merge to `main`.
3. Push tag `vX.Y.Z` matching the new version (e.g. `git tag -a v0.3.0 -m "..." && git push origin v0.3.0`).
4. The promotion workflow verifies the tag matches `plugin.json`, then fast-forwards `stable` to the tagged commit. Users get the new release on their next `/plugin marketplace update`.

Pre-release tags (`v0.3.0-rc.1`, etc.) intentionally do not advance `stable`.
