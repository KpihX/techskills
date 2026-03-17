# 🐙 GitHub CLI (`gh`) — Ubuntu 24.04 Field Guide

> **Machine:** KpihX-Ubuntu (Ubuntu 24.04 LTS)
> **Lived on:** 2026-03 · **Status:** Daily driver

---

## 🧩 Context & Problem

```
Problem:  GitHub's web UI is slow for repetitive tasks:
          - Creating repos
          - Opening / merging PRs
          - Checking CI status
          - Configuring settings (Pages, secrets, branch rules)

Goal:     Do everything from the terminal.
          One tool, full GitHub API access.
```

**`gh`** is GitHub's official CLI — it wraps the REST + GraphQL APIs and
integrates with `git`. Anything you can do on github.com, you can do with `gh`.

---

## 🏗️ Architecture

```
You (terminal)
    │
    ▼
 gh CLI  ──── REST/GraphQL ────▶  api.github.com
    │                                   │
    │        SSH (git ops)              │
    └──── git@github.com ◀─────────────┘
              │
         Your repos
```

`gh` handles auth, then delegates actual git operations to your system `git`
(with your SSH keys). It never touches your SSH keys directly.

---

## 🔧 Installation

### Option A — Official apt repo (recommended, auto-updates)

```bash
# 1. Add GitHub CLI apt repo
(type -p wget >/dev/null || sudo apt install wget -y) \
  && sudo mkdir -p -m 755 /etc/apt/keyrings \
  && wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg \
     | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
  && sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
     | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null

# 2. Install
sudo apt update && sudo apt install gh -y

# 3. Verify
gh --version
# → gh version 2.x.x (2024-xx-xx)
```

### Option B — Direct binary (no apt)

```bash
# Get latest release URL from: https://github.com/cli/cli/releases
VERSION="2.68.0"
curl -Lo /tmp/gh.tar.gz \
  "https://github.com/cli/cli/releases/download/v${VERSION}/gh_${VERSION}_linux_amd64.tar.gz"
tar -xzf /tmp/gh.tar.gz -C /tmp
sudo install /tmp/gh_${VERSION}_linux_amd64/bin/gh /usr/local/bin/gh
```

---

## 🔐 Authentication

```bash
gh auth login
```

Interactive prompts — recommended answers for KpihX setup:

```
? What account do you want to log into?   GitHub.com
? What is your preferred protocol?        SSH
? Upload your public SSH key to GitHub?   Skip (Bitwarden manages keys)
? How would you like to authenticate?     Login with a web browser
```

> Opens browser → paste one-time code → authorize.
> Token is stored in `~/.config/gh/hosts.yml`.

**Check auth status:**

```bash
gh auth status
# → ✓ Logged in to github.com account KpihX (keyring)
#   ✓ Git operations for github.com configured to use ssh protocol
```

**Token scope** (what gh can access):

```bash
gh auth status --show-token   # shows scopes, not the token value
```

---

## 📦 Repositories

### Create a repo

```bash
# Public, from current directory
gh repo create kpihx/my-project --public --description "My cool project"

# Private, with README + gitignore
gh repo create kpihx/my-project --private --add-readme --gitignore Python

# Interactive (asks all questions)
gh repo create
```

### Clone a repo

```bash
# Clone your own repo (uses SSH automatically)
gh repo clone kpihx/tutos_live

# Clone someone else's repo
gh repo clone cli/cli
```

### Fork a repo

```bash
# Fork + clone in one shot
gh repo fork cli/cli --clone

# Fork only (no clone)
gh repo fork cli/cli
```

### List your repos

```bash
# Public repos, sorted by push date
gh repo list kpihx --limit 20

# Filter by language
gh repo list kpihx --language python

# Only private repos
gh repo list kpihx --visibility private
```

### View repo info

```bash
# Summary in terminal
gh repo view kpihx/tutos_live

# Open in browser
gh repo view kpihx/tutos_live --web
```

### Delete a repo

```bash
gh repo delete kpihx/test-repo --confirm
```

### Rename / transfer

```bash
gh repo rename new-name
gh repo transfer kpihx/project target-org
```

---

## 🔀 Pull Requests

```
Local branch → PR → review → merge
      │                         │
    git push               gh pr merge
```

### Create a PR

```bash
# From current branch, interactive
gh pr create

# One-liner (title + body + target branch)
gh pr create \
  --title "feat: add tailscale tutorial" \
  --body "Covers MagicDNS fix, SSH config, Bitwarden SSH agent" \
  --base main \
  --head feat/tailscale-tuto

# Draft PR
gh pr create --draft --title "WIP: new tutorial"
```

### List PRs

```bash
# All open PRs
gh pr list

# Filter by author / label / state
gh pr list --author kpihx
gh pr list --label bug
gh pr list --state closed
```

### View a PR

```bash
# By number
gh pr view 42

# Open in browser
gh pr view 42 --web

# Show diff
gh pr diff 42
```

### Review a PR

```bash
# Add a comment
gh pr review 42 --comment --body "LGTM, one nit below"

# Approve
gh pr review 42 --approve

# Request changes
gh pr review 42 --request-changes --body "Please add tests"
```

### Checkout a PR locally

```bash
# Fetches the branch and checks it out
gh pr checkout 42
```

### Merge a PR

```bash
# Merge commit
gh pr merge 42

# Squash merge
gh pr merge 42 --squash

# Rebase merge
gh pr merge 42 --rebase

# Auto-merge when checks pass
gh pr merge 42 --auto --squash
```

### Close / reopen

```bash
gh pr close 42
gh pr reopen 42
```

---

## 🐛 Issues

### Create an issue

```bash
# Interactive
gh issue create

# One-liner
gh issue create \
  --title "MagicDNS breaks on X WiFi" \
  --body "Resolved by anchoring 100.100.100.100 in resolved.conf.d" \
  --label bug \
  --assignee kpihx
```

### List issues

```bash
gh issue list
gh issue list --label "bug"
gh issue list --assignee kpihx
gh issue list --state closed
```

### View / comment

```bash
gh issue view 7
gh issue comment 7 --body "Fixed in #42"
```

### Close / reopen

```bash
gh issue close 7
gh issue close 7 --comment "Fixed in commit abc1234"
gh issue reopen 7
```

---

## 🚀 Releases

### Create a release

```bash
# Tag + release notes from merged PRs (auto-generated)
gh release create v1.0.0 \
  --title "v1.0.0 — Initial stable release" \
  --notes "First production-ready version" \
  --latest

# With file assets
gh release create v1.0.0 \
  --title "v1.0.0" \
  dist/myapp-linux-amd64 \
  dist/myapp-darwin-arm64

# Draft release (not published yet)
gh release create v1.0.0 --draft

# Pre-release
gh release create v1.0.0-beta.1 --prerelease
```

### List / view

```bash
gh release list
gh release view v1.0.0
```

### Download assets

```bash
# Download all assets of a release
gh release download v1.0.0

# Specific file pattern
gh release download v1.0.0 --pattern "*.tar.gz"
```

---

## ⚙️ Repository Settings via API

`gh` exposes the **full GitHub REST API** via `gh api`. Anything not in the
standard subcommands can be done this way.

### GitHub Pages

```bash
# Enable Pages (branch + root)
gh api repos/kpihx/tutos_live/pages \
  --method POST \
  -f "source[branch]=master" \
  -f "source[path]=/"

# Check Pages status
gh api repos/kpihx/tutos_live/pages

# Update Pages source
gh api repos/kpihx/tutos_live/pages \
  --method PUT \
  -f "source[branch]=main" \
  -f "source[path]=/"
```

### Secrets (Actions)

```bash
# Set a repo secret
gh secret set MY_API_KEY
# → prompts for value (hidden input)

# List secrets (names only — values never shown)
gh secret list

# Delete a secret
gh secret delete MY_API_KEY
```

### Branch protection rules

```bash
# Require PR reviews before merge on main
gh api repos/kpihx/my-project/branches/main/protection \
  --method PUT \
  --input - <<'EOF'
{
  "required_status_checks": null,
  "enforce_admins": false,
  "required_pull_request_reviews": {
    "required_approving_review_count": 1
  },
  "restrictions": null
}
EOF
```

### Topics / description

```bash
# Set topics
gh api repos/kpihx/tutos_live/topics \
  --method PUT \
  -f "names[]=ubuntu" -f "names[]=docsify" -f "names[]=tutorials"

# Update description
gh api repos/kpihx/tutos_live \
  --method PATCH \
  -f description="Ubuntu field notes — real problems, real fixes"
```

---

## 🔄 GitHub Actions / CI

```bash
# List workflows
gh workflow list

# View runs for a workflow
gh run list --workflow=ci.yml

# Watch a run in real time
gh run watch

# View logs of a specific run
gh run view 1234567 --log

# Re-run failed jobs
gh run rerun 1234567 --failed

# Trigger a workflow manually (workflow_dispatch)
gh workflow run deploy.yml --ref main
```

---

## 🛠️ Useful Shortcuts

```bash
# Open current repo in browser
gh browse

# Open specific file/issue/PR in browser
gh browse README.md
gh browse --issues
gh browse --settings

# SSH key management
gh ssh-key list
gh ssh-key add ~/.ssh/id_ed25519.pub --title "KpihX-Ubuntu"
gh ssh-key delete <key-id>

# GPG key management
gh gpg-key list
gh gpg-key add < ~/.gnupg/mykey.gpg
```

---

## 🐛 Debugging

### Auth fails

```bash
# Re-authenticate
gh auth login

# Refresh token scopes
gh auth refresh --scopes repo,read:org
```

### SSH vs HTTPS mismatch

```bash
# Force SSH for all GitHub git operations
gh config set git_protocol ssh --host github.com
```

### Rate limit

```bash
# Check remaining API calls
gh api rate_limit | jq '.rate'
# → { "limit": 5000, "remaining": 4987, "reset": 1710000000 }
```

---

## ✅ Verification

```bash
# Full health check
gh auth status
gh repo list kpihx --limit 5
gh api user | jq '{login, name, public_repos}'
```

---

## 📚 References

- [GitHub CLI docs](https://cli.github.com/manual/)
- [GitHub REST API](https://docs.github.com/en/rest)
- [gh release notes](https://github.com/cli/cli/releases)
