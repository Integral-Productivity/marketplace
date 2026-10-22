# 2. Public plugin copies live in catalog folders

Date: 2026-10-22

## Status

Accepted (folders), 2026-10-22.

OQ1 and OQ3 pass. E7 stays open and is sampled during the pilot release; it can change update guidance, not the folder decision.

## Context

Integral Productivity wants a person with no access to the GitHub organization to install a plugin classified "public" in Claude Desktop (Chat, Cowork and Code) from a public catalog, without a zip upload. The plugin source repositories stay private or internal, because a public source repository would also publish plans, issues, pull requests, Actions logs and the full git history.

The design under test: on each `vX.Y.Z` tag of a public-classified plugin, CI in the source repository builds an allowlisted copy of the installable files, runs a privacy check, and opens a pull request that writes the copy to `plugins/<name>/` in a public catalog repository. The catalog entry uses a relative source, `"source": "./plugins/<name>"`.

Two questions must be answered before any build work starts:

- **OQ1.** Does Claude Desktop install a plugin whose source is a relative folder inside a custom catalog repository, on every surface?
- **OQ3.** Can the `ip-releaser` GitHub App be minted through `OP_RELEASER_PUBLIC_TOKEN`, from a workflow in a non-public source repository, with contents and pull-request write on the catalog repository, and is it free of any ruleset bypass there?

Placement between catalogs follows `marketplace-internal` ADR-0003: a plugin that ADR places in the core catalog goes to this repository (`marketplace`); a public-classified labs plugin goes to a public labs catalog. The plugin owner chose a new public repository, `Integral-Productivity/marketplace-labs-public` (catalog name `integral-productivity-labs-public`), seeded with one squashed commit. The private `marketplace-labs` stays unchanged as the internal catalog.

## Decision

Public plugin copies live in folders inside the public catalog repository (`plugins/<name>/`), and catalog entries use relative sources. If OQ1 fails on any Claude Desktop surface, fall back to one public repository per plugin with a `github` source at `ref: stable`. Anthropic's catalog also offers a `git-subdir` source type, which is a second fallback to evaluate before creating per-plugin repositories.

## Evidence

| # | Scenario | Result | Date | Evidence |
|---|---|---|---|---|
| E1 | Claude Code CLI 2.1.307 installs a relative-folder plugin from a local folder catalog | Pass | 2026-10-22 | `claude plugin install demo@route-test-catalog` installed version 0.1.0 read from `plugins/demo` |
| E2 | Claude Code CLI 2.1.307 installs a relative-folder plugin from a public GitHub catalog it cloned, with no GitHub authentication | Pass | 2026-10-22 | `claude plugin marketplace add anthropics/claude-code`, then `claude plugin install code-review@claude-code-plugins` installed 1.0.0; entries use `./plugins/<name>` |
| E3 | Anthropic's own directory uses relative sources | Observed | 2026-10-22 | `anthropics/claude-plugins-official` `marketplace.json`: 53 relative, 98 `git-subdir`, 164 `url` sources. This is indirect evidence for Cowork, not a test of a custom catalog |
| E4 | Cowork: "Add marketplace" on a public custom catalog with relative sources, install one plugin | Pass | 2026-10-22 | Plugin owner added `anthropics/claude-code` in Cowork and installed `code-review`; `/code-review` appears |
| E5 | Chat: same as E4 | Pass | 2026-10-22 | The plugin installed in Cowork appears in Chat; the two surfaces share one install |
| E6 | Claude Code in Claude Desktop: same as E4 | Pass (already installed) | 2026-10-22 | In the Code tab the marketplace and `code-review` were already present; `/plugin marketplace add` opened the marketplace UI. Not a fresh install, but the Code tab runs the same engine that passed E1 and E2 |
| E7 | A second version is offered by "Check for updates", with the observed delay | Deferred to the pilot | | Needs our own tags; sampled during the pilot release |
| E8 | A private `marketplace-internal` entry installs in Cowork, or fails with a recorded message (anthropics/claude-code#61271) | Fail for non-public sources, silent | 2026-10-22 | The private catalog adds, but Cowork lists only `integral-productivity-engineering`, the one entry whose source repository is public. Seven entries with internal or private sources (valid manifests, descriptions within upload limits, `stable` present) are omitted with no error. Inference: Cowork drops entries whose source it cannot read without the user's GitHub access |
| E9 | `ip-releaser` token mint with contents and pull-request write on `marketplace`, from a non-public repository's workflow, and a PR opened | Pass | 2026-10-22 | Probe workflow in an internal repository minted the token through `OP_RELEASER_PUBLIC_TOKEN` and opened, then closed, marketplace PR #40 |
| E10 | `ip-releaser` is not a bypass actor on any ruleset protecting `marketplace` `main` | Pass (as visible to a non-admin) | 2026-10-22 | `GET /repos/Integral-Productivity/marketplace/rulesets/20821950` ("Tier 3 — protect main"): `bypass_actors: []`, the only ruleset on `main`. An org admin should confirm, since the API can hide bypass actors |

Also observed for E10: the ruleset requires a pull request with 0 approvals and squash merge, and declares **no required status checks**. `validate-marketplace.yml` reports but does not gate. The catalog integrity check planned for public catalogs must be added as a required check, or it gates nothing.

## Consequences

- If accepted (folders): one public catalog repository per placement holds every public plugin copy, so there are fewer repositories and secrets to manage. The catalog repository's history becomes part of the public record, so the privacy check must pass before the first commit of any copy.
- If the fallback is taken: each public plugin needs its own public repository, its own `stable` branch and its own credential scope.
- Public labs plugins go to `marketplace-labs-public`.
- E8 confirms the premise of this design: a plugin reaches Cowork through a catalog only when its installable files are publicly readable. Private and internal plugins keep the zip upload route in Cowork.
