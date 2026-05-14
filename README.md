# Integral Productivity Marketplace

A [Claude Code plugin marketplace](https://docs.claude.com/en/docs/claude-code/plugin-marketplaces) for plugins maintained by [Integral-Productivity](https://github.com/Integral-Productivity). The marketplace metadata is public; individual plugin repositories may be **INTERNAL** or **PRIVATE** and require GitHub authentication to install.

## Catalog

| Plugin | Source | Visibility | Description |
|---|---|---|---|
| `lean-management` | [Integral-Productivity/lean-management](https://github.com/Integral-Productivity/lean-management) | INTERNAL | Lean management support for an organization |

## Prerequisites

Claude Code's plugin installer clones `source: github` entries over SSH (`git@github.com:owner/repo.git`), so SSH to GitHub must work before `/plugin install …@integral-productivity-tools` will succeed against any INTERNAL or PRIVATE repo in this marketplace.

### SSH (recommended for internal/private plugins)

Pick **one** of the two key-management options below, then run the verification step at the end of this section.

#### Option A: 1Password SSH agent (preferred)

1. **Create or reuse a GitHub SSH key in 1Password.** 1Password → New Item → SSH Key, or reuse an existing one. The 1Password app must have its SSH agent enabled (1Password → Settings → Developer → "Use the SSH agent").

2. **Upload the public key to GitHub.** Copy it from 1Password, or capture it from the running agent:
   ```sh
   ssh-add -L | pbcopy
   ```
   Then in the browser: GitHub → avatar → **Settings** → **SSH and GPG keys** → **New SSH key** → paste → **Add SSH key**.

3. **Persist `SSH_AUTH_SOCK` for GUI-launched processes** (Claude Code Desktop, IDEs launched from Dock / Spotlight). `~/.ssh/config`'s `IdentityAgent` directive covers anything that shells out to `ssh`, but tools that read `SSH_AUTH_SOCK` directly need the env var set at the launchd level.

   Add this to the top of `~/.zshrc`:
   ```sh
   export SSH_AUTH_SOCK="$HOME/Library/Group Containers/2BUA8C4S2C.com.1password/t/agent.sock"
   ```

   Create `~/Library/LaunchAgents/com.integralproductivity.ssh-auth-sock.plist` (replace `<YOUR_USERNAME>` with your macOS username):
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
   <plist version="1.0">
   <dict>
     <key>Label</key>
     <string>com.integralproductivity.ssh-auth-sock</string>
     <key>RunAtLoad</key>
     <true/>
     <key>ProgramArguments</key>
     <array>
       <string>/bin/launchctl</string>
       <string>setenv</string>
       <string>SSH_AUTH_SOCK</string>
       <string>/Users/<YOUR_USERNAME>/Library/Group Containers/2BUA8C4S2C.com.1password/t/agent.sock</string>
     </array>
   </dict>
   </plist>
   ```

   Load it for the current session and every future login, then apply to the running launchd session so you don't have to log out:
   ```sh
   launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.integralproductivity.ssh-auth-sock.plist
   launchctl setenv SSH_AUTH_SOCK "$HOME/Library/Group Containers/2BUA8C4S2C.com.1password/t/agent.sock"
   launchctl getenv SSH_AUTH_SOCK   # should print the 1Password agent path
   ```

   Fully quit and relaunch Claude Code (and any IDE) so they pick up the new environment.

#### Option B: Software Ed25519 key with macOS Keychain (fallback)

For teammates not using 1Password. Less secure than Option A but better than HTTPS + PAT.

1. Generate a key (set a strong passphrase when prompted):
   ```sh
   ssh-keygen -t ed25519 -C "you@integralproductivity.com" -f ~/.ssh/id_ed25519_github
   ```
2. Add a host block to `~/.ssh/config`:
   ```
   Host github.com
     User git
     IdentityFile ~/.ssh/id_ed25519_github
     IdentitiesOnly yes
     AddKeysToAgent yes
     UseKeychain yes
   ```
3. Load the key into ssh-agent and the Keychain:
   ```sh
   ssh-add --apple-use-keychain ~/.ssh/id_ed25519_github
   ```
4. Upload `~/.ssh/id_ed25519_github.pub` to GitHub: Settings → **SSH and GPG keys** → **New SSH key**.

#### SAML SSO authorization (optional today)

Integral-Productivity has SAML SSO **enabled** but **not required**, so SSH keys work for org-resource access without per-key SSO authorization. You can skip this step today.

If the org flips "Require SAML SSO authentication" on in the future, every SSH key that touches org resources must be authorized first: Settings → **SSH and GPG keys** → next to the key, click **Configure SSO** → **Authorize** beside **Integral-Productivity** → complete the IdP flow.

#### Verify

Before declaring success:
```sh
ssh -T git@github.com
# expect: Hi <your-handle>! You've successfully authenticated, …

git ls-remote git@github.com:Integral-Productivity/lean-management.git HEAD
# expect: a 40-char SHA on stdout
```

1Password may prompt for Touch ID / unlock on first use.

### HTTPS + `gh` (alternative)

> **Note:** Claude Code's plugin installer uses SSH for `source: github` entries, so this path alone is **not** sufficient to install plugins from INTERNAL or PRIVATE repos in this marketplace. Use the SSH path above for `/plugin install`. The HTTPS + `gh` setup remains useful for `gh` API calls and any HTTPS git remotes elsewhere on your machine.

1. Authenticate with the GitHub CLI, granting at least `repo` scope (covers private + internal reads):
   ```sh
   gh auth login
   ```
2. Wire `gh` into git's credential chain so HTTPS `git clone` calls reuse the same token:
   ```sh
   gh auth setup-git
   ```
3. Sanity-check HTTPS access to a target repo (replace as needed):
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
