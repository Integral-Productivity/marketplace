# Integral Productivity Marketplace

A [Claude Code plugin marketplace](https://docs.claude.com/en/docs/claude-code/plugin-marketplaces) for publicly-available plugins maintained or curated by [Integral-Productivity](https://github.com/Integral-Productivity) — both plugins built inside the org and vetted third-party plugins we recommend. All plugins listed here live in public repositories and install without additional authentication setup, so this marketplace works in both [Claude Code](https://docs.claude.com/en/docs/claude-code) CLI and Claude Desktop (Cowork).

> **Looking for org-internal plugins?** Plugins that should stay inside Integral-Productivity live in [`Integral-Productivity/marketplace-internal`](https://github.com/Integral-Productivity/marketplace-internal) (private). That marketplace is CLI-only — see its README for SSH setup. Cowork in Claude Desktop cannot sync marketplaces that reference private or internal plugin repos; see [anthropics/claude-code#61271](https://github.com/anthropics/claude-code/issues/61271) for the open feature request.

## Catalog

| Plugin | Source | Description |
|---|---|---|
| `lean-management` | [Integral-Productivity/lean-management](https://github.com/Integral-Productivity/lean-management) | Lean management support for an organization |
| `model-framework-integration` | [Integral-Productivity/model-framework-integration](https://github.com/Integral-Productivity/model-framework-integration) | Skills for integrating across major models and frameworks (Integral Theory / AQAL, Ego Development, Lean) |
| `holacracy` | [Integral-Productivity/holacracy-claude-plugin](https://github.com/Integral-Productivity/holacracy-claude-plugin) | Engage with Holacracy — Facilitator, Secretary, Lead Link, Rep Link co-pilots, a governance-aware operating frame, and the GlassFrog MCP connector |
| `metawork` | [Integral-Productivity/metawork-claude-plugin](https://github.com/Integral-Productivity/metawork-claude-plugin) | (Alpha/preview) Teach, run, set up, and coach the Meta Work methodology across project, area, domain, and identity scopes |
| `mattpocock-skills` | [mattpocock/skills](https://github.com/mattpocock/skills) | Matt Pocock's "Skills for Real Engineers" — composable engineering & productivity skills (curated third-party) |

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

> **Third-party plugins.** A plugin in a repo Integral-Productivity does not control (e.g. [`mattpocock/skills`](https://github.com/mattpocock/skills)) cannot host the tag-driven `stable`-channel workflow. Pin its entry to a reviewed commit SHA — `"ref": "<full-commit-sha>"` — instead of `"ref": "stable"`. Bumping to a newer upstream commit is then an explicit, reviewable PR. Do not "fix" such an entry to `ref: "stable"`; that branch does not exist upstream. See [`docs/adr/0002`](docs/adr/0002-third-party-marketplace-plugins-are-pinned-by-commit-sha-not-the-stable-channel.md).

### Release process for plugin maintainers

Once a plugin is listed and the promotion workflow is wired up, a release is:

1. Bump `version` in the plugin repo's `.claude-plugin/plugin.json` on a feature branch.
2. Merge to `main`.
3. Push tag `vX.Y.Z` matching the new version (e.g. `git tag -a v0.3.0 -m "..." && git push origin v0.3.0`).
4. The promotion workflow verifies the tag matches `plugin.json`, then fast-forwards `stable` to the tagged commit. Users get the new release on their next `/plugin marketplace update`.

Pre-release tags (`v0.3.0-rc.1`, etc.) intentionally do not advance `stable`.
