# 🐙 GitHub CLI (`gh`) — Stop Using the Browser for Daily GitHub Tasks

> **Machine:** KpihX-Ubuntu (Ubuntu 25.10)
> **Lived on:** 2026-03 · **Status:** Daily driver

---

For a long time I was doing everything through github.com: creating repos,
opening pull requests, checking CI status, managing releases. One tab per
action, forms to fill, pages to navigate. Fine when you do it occasionally.
When it's part of your daily flow, the browser round-trips add up.

`gh` is GitHub's official CLI. After installing it, I barely open github.com
anymore — except to read code review comments, which is still better in a browser.

Here's what I actually use it for, in the order I started using it.

---

## Installing on Ubuntu 25.10

The official apt repo is the cleanest option — it gets updates automatically.

```bash
# Add GitHub's apt repo
(type -p wget >/dev/null || sudo apt install wget -y) \
  && sudo mkdir -p -m 755 /etc/apt/keyrings \
  && wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg \
     | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
  && sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
  && echo "deb [arch=$(dpkg --print-architecture) \
     signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] \
     https://cli.github.com/packages stable main" \
     | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null

sudo apt update && sudo apt install gh -y

gh --version
# → gh version 2.x.x
```

---

## Authenticating

```bash
gh auth login
```

`gh` walks you through an interactive setup. The answers that make sense for
my setup:

```
? What account do you want to log into?   GitHub.com
? What is your preferred protocol?        SSH
? Upload your public SSH key to GitHub?   Skip  (Bitwarden manages my keys)
? How would you like to authenticate?     Login with a web browser
```

A code appears in the terminal. Copy it, the browser opens automatically,
paste it, authorize. Done — token is stored in `~/.config/gh/hosts.yml`.

Check it worked:

```bash
gh auth status
# → ✓ Logged in to github.com account KpihX
# → ✓ Git operations configured to use ssh protocol
```

---

## Creating repos

This was the first thing I replaced. Instead of:

> github.com → New → fill form → copy URL → terminal → git remote add → push

I now do:

```bash
# From inside the project directory
gh repo create kpihx/techskills \
  --public \
  --description "Ubuntu-focused live tutorials"

# Then add the remote and push as usual
git remote add github git@github.com:kpihx/techskills.git
git push github HEAD
```

Other useful variations:

```bash
# Private repo
gh repo create kpihx/private-notes --private

# Interactive (asks all questions)
gh repo create

# Clone someone else's repo (uses SSH automatically)
gh repo clone cli/cli

# Fork + clone in one shot
gh repo fork cli/cli --clone
```

Once a repo exists, `gh repo view` gives a quick summary, and `--web` opens
it in the browser if you need the full UI:

```bash
gh repo view kpihx/techskills
gh repo view kpihx/techskills --web

# List your repos
gh repo list kpihx --limit 20
gh repo list kpihx --language python
```

---

## Pull requests

PRs were next. The `gh pr create` flow is fast enough that I stopped
reaching for the browser entirely.

```bash
# From a feature branch, interactive
gh pr create

# Or one-liner
gh pr create \
  --title "feat: add tailscale tutorial" \
  --body "Covers MagicDNS fix, SSH config, Bitwarden SSH agent" \
  --base main

# Draft (mark as not ready for review)
gh pr create --draft --title "WIP: new tutorial"
```

On the other end — reviewing and managing PRs I received:

```bash
# See what's open
gh pr list

# Read a specific PR (with diff)
gh pr view 42
gh pr diff 42

# Checkout the branch to test it locally
gh pr checkout 42

# Approve
gh pr review 42 --approve

# Merge (squash is my default)
gh pr merge 42 --squash --remove-branch
```

```bash
# Close / reopen without merging
gh pr close 42
gh pr reopen 42
```

---

## Issues

```bash
# Open an issue
gh issue create \
  --title "MagicDNS breaks on X WiFi" \
  --body "Root cause: search domain ordering in systemd-resolved" \
  --label bug \
  --assignee kpihx

# List open issues
gh issue list
gh issue list --label "bug"
gh issue list --state closed

# Comment and close
gh issue comment 7 --body "Fixed in #42"
gh issue close 7 --comment "Fixed in commit abc1234"
```

---

## Releases

When a project reaches a stable point, I tag and release from the terminal:

```bash
# Tag + release with auto-generated notes from merged PRs
gh release create v1.0.0 \
  --title "v1.0.0 — Initial stable release" \
  --generate-notes \
  --latest

# With file assets (binaries, archives)
gh release create v1.0.0 \
  --title "v1.0.0" \
  dist/myapp-linux-amd64 dist/myapp-darwin-arm64

# Draft first, publish when ready
gh release create v1.0.0 --draft
```

```bash
gh release list
gh release view v1.0.0
gh release download v1.0.0 --pattern "*.tar.gz"
```

---

## The `gh api` escape hatch

Everything above covers the most common workflows. For anything else —
settings, webhooks, branch protection, Pages config — `gh api` exposes the
full GitHub REST API directly from the terminal.

I use it most often for two things.

**Enabling GitHub Pages on a new repo:**

```bash
gh api repos/kpihx/techskills/pages \
  --method POST \
  -f "source[branch]=master" \
  -f "source[path]=/"
```

**Setting repo topics:**

```bash
gh api repos/kpihx/techskills/topics \
  --method PUT \
  -f "names[]=ubuntu" -f "names[]=docsify" -f "names[]=tutorials"
```

**Managing secrets (Actions):**

```bash
gh secret set MY_API_KEY    # prompts for value, hidden input
gh secret list              # names only — values never shown
gh secret delete MY_API_KEY
```

**Branch protection:**

```bash
gh api repos/kpihx/my-project/branches/main/protection \
  --method PUT \
  --input - <<'EOF'
{
  "required_status_checks": null,
  "enforce_admins": false,
  "required_pull_request_reviews": { "required_approving_review_count": 1 },
  "restrictions": null
}
EOF
```

---

## GitHub Actions — checking CI without a browser

```bash
# List recent workflow runs
gh run list

# Watch a run in real time
gh run watch

# View logs of a failed run
gh run view 1234567 --log

# Re-run only failed jobs
gh run rerun 1234567 --failed

# Trigger a workflow manually
gh workflow run deploy.yml --ref main
```

---

## A few shortcuts I use daily

```bash
# Open current repo in browser (for when I do need the UI)
gh browse

# Open current repo's issues directly
gh browse --issues

# List and manage SSH keys on GitHub
gh ssh-key list
gh ssh-key add ~/.ssh/id_ed25519.pub --title "KpihX-Ubuntu"

# Check API rate limit (5000 requests/hour authenticated)
gh api rate_limit | jq '.rate'
```

---

## 🐛 Quick fixes

**Auth fails or token expired**
```bash
gh auth login  # re-authenticates, overwrites stored token
```

**`gh` uses HTTPS instead of SSH for git operations**
```bash
gh config set git_protocol ssh --host github.com
```

**Rate limited**
```bash
gh api rate_limit | jq '{limit: .rate.limit, remaining: .rate.remaining}'
# If remaining is 0, wait until .rate.reset (Unix timestamp)
```

---

## ✅ Quick health check

```bash
gh auth status
gh repo list kpihx --limit 5
gh api user | jq '{login, name, public_repos}'
```

---

## 📚 References

- [GitHub CLI manual](https://cli.github.com/manual/)
- [GitHub REST API](https://docs.github.com/en/rest)
- [GitLab CLI (`glab`) guide](dev/glab.md)
