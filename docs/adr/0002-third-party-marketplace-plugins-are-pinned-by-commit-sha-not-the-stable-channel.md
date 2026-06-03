# 2. Third-party marketplace plugins are pinned by commit SHA, not the stable channel

Date: 2026-06-03

## Status

Accepted

## Context

This marketplace originally listed only plugins maintained inside the Integral-Productivity
org. Those plugins publish through a `stable` release channel: each plugin repo keeps a
`stable` branch that a tag-driven GitHub Actions workflow fast-forwards to a verified
release tag, and the marketplace entry uses `"ref": "stable"` so users always track the
latest vetted release without a marketplace edit. See holacracy-claude-plugin's
`docs/adr/0002` for that decision.

We now want to list a curated **third-party** plugin (`mattpocock/skills`). Two
constraints fall out of not controlling the upstream repo:

- We cannot add a `stable` branch or the tag-driven promotion workflow to a repo we
  don't own, so `"ref": "stable"` is not available.
- Tracking the upstream default branch (`"ref": "main"`, or omitting `ref`) would let
  arbitrary upstream commits — new skills, removed skills, breaking edits — reach our
  users with no review.

For `mattpocock/skills` specifically, the only branch carrying a valid
`.claude-plugin/plugin.json` is `main`; the `v1` branch is a pre-restructure layout with
no plugin manifest and is not installable as a Claude Code plugin.

## Decision

Third-party plugins (those in repos Integral-Productivity does not control) are pinned in
`.claude-plugin/marketplace.json` to a specific, reviewed **commit SHA** —
`"ref": "<full-commit-sha>"` — rather than `"ref": "stable"` or a moving branch.

Updating a third-party plugin to a newer upstream commit is an explicit pull request that
bumps the SHA, so every upstream change a user receives has passed through review. The
`stable`-channel pattern remains the standard for IP-maintained plugins.

The README's "Adding a plugin" section documents this exception so the SHA pin is not
mistaken for an oversight and "corrected" to a non-existent `stable` ref.

## Consequences

- **Safer by default:** users only receive upstream third-party changes that a maintainer
  has reviewed and merged here; a surprise upstream edit cannot reach them silently.
- **Reproducible:** the marketplace entry resolves to one immutable commit.
- **Manual upkeep:** picking up upstream improvements requires a deliberate SHA-bump PR.
  Third-party plugins will lag upstream until someone refreshes them; there is no
  automatic update path equivalent to the stable channel.
- **Two patterns coexist:** maintainers must recognize that `"ref": "stable"` (IP plugins)
  and `"ref": "<sha>"` (third-party plugins) are both correct, applied by ownership.
