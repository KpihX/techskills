# 🦊 GitLab CLI (`glab`) — GitLab Without the Browser

> **Machine:** KpihX-Ubuntu (Ubuntu 25.10)
> **Lived on:** 2026-03 · **Status:** Daily driver

---

GitLab isn't just a GitHub mirror. The philosophy is different: CI/CD is
native and tightly integrated, Merge Requests have structured approval flows,
secrets have fine-grained protection (masked, protected branches only), and
you can self-host the entire thing — which I do with my homelab.

For a long time I was managing all of this through the web UI. It works, but
GitLab's interface has a lot of layers to navigate — especially for pipeline
status and variable management.

`glab` is the official GitLab CLI. Same idea as `gh` for GitHub: pull the
most common workflows into the terminal, skip the browser for everything
except code review comments.

Here's how I set it up and what I use it for.

---

## Installing on Ubuntu 25.10

The cleanest option is GitLab's official apt repo:

```bash
# Add GitLab's apt repo (handles GPG key automatically)
curl -fsSL https://packages.gitlab.com/install/repositories/gitlab/glab/script.deb.sh \
  | sudo bash

sudo apt install glab -y

glab --version
# → glab version 1.x.x
```

Alternative — direct binary if you prefer not to add the repo:

```bash
VERSION="1.53.0"  # check latest at gitlab.com/gitlab-org/cli/-/releases
curl -Lo /tmp/glab.tar.gz \
  "https://gitlab.com/gitlab-org/cli/-/releases/v${VERSION}/downloads/glab_${VERSION}_Linux_x86_64.tar.gz"
tar -xzf /tmp/glab.tar.gz -C /tmp
sudo install /tmp/bin/glab /usr/local/bin/glab
```

---

## Authenticating

```bash
glab auth login
```

```
? What GitLab instance do you want to log into?   gitlab.com
? How would you like to authenticate?             Token
? Paste your authentication token:               [hidden]
? Choose default git protocol:                    SSH
```

Get a token at GitLab → User Settings → Access Tokens → New token.
Scopes you need: `api`, `read_user`, `read_repository`, `write_repository`.

Check it worked:

```bash
glab auth status
# → ✓ Logged in to gitlab.com as kpihx
# → ✓ Git operations configured to use ssh protocol
```

If you also use a self-hosted GitLab instance (like I do with the homelab):

```bash
glab auth login --hostname gitlab.homelab.internal
```

`glab` handles both simultaneously — it reads the git remote URL of the
current directory and picks the right instance automatically.

---

## Repos

```bash
# Create a public repo from the current directory
glab repo create kpihx/techskills --public --description "Ubuntu live tutorials"

# Clone
glab repo clone kpihx/techskills

# Fork + clone someone else's project
glab repo fork gitlab-org/gitlab-runner --clone

# List your repos
glab repo list
glab repo list --member   # repos you're a member of
glab repo list --starred

# Open in browser when you need the full UI
glab repo view kpihx/techskills --web
```

---

## Merge Requests

GitLab calls them **Merge Requests** (MRs), not Pull Requests. Same concept,
slightly different vocabulary and a richer approval model.

The day-to-day flow for my own projects:

```bash
# I just pushed a feature branch. Open an MR from it:
glab mr create \
  --title "feat: add tailscale tutorial" \
  --description "Covers MagicDNS fix, SSH, Bitwarden SSH agent" \
  --target-branch main

# Draft (not ready for review yet)
glab mr create --draft --title "WIP: new tutorial"

# Push the current branch AND open the MR in one shot
glab mr create --push
```

Managing MRs I received for review:

```bash
# What's open
glab mr list
glab mr list --state closed
glab mr list --reviewer kpihx   # MRs assigned to me for review

# Read it
glab mr view 42
glab mr view 42 --web

# Checkout the branch to test locally
glab mr checkout 42

# Approve
glab mr approve 42

# Add a note
glab mr note 42 --message "LGTM — one nit on line 42"

# Merge (squash + delete source branch)
glab mr merge 42 --squash-before-merge --remove-source-branch
```

---

## Issues

```bash
glab issue create \
  --title "MagicDNS breaks on X WiFi" \
  --description "Root cause: search domain ordering in systemd-resolved" \
  --label "bug" \
  --assignee kpihx

glab issue list
glab issue list --label "bug"
glab issue list --assignee kpihx

glab issue note 7 --message "Fixed in MR !42"
glab issue close 7 --message "Fixed in MR !42"
```

---

## CI/CD pipelines — where GitLab shines

This is where `glab` earns its place. GitLab CI is native — every push can
trigger a pipeline defined in `.gitlab-ci.yml`. Managing pipelines from
the browser means a lot of clicking. `glab` makes it fast.

A typical day: I push a change, something fails, I diagnose and fix from the terminal.

```bash
# What ran recently
glab pipeline list
glab pipeline list --branch main

# Watch the current branch's pipeline live
glab pipeline ci view
```

The `ci view` output streams pipeline status in real time:

```
Pipeline #123456 — running
  ✔ build:docker       1m12s
  ⚙ test:unit          running...
  ✔ lint               38s
  ⏸ deploy:staging     pending (waiting for test:unit)
```

When a job fails:

```bash
# See which jobs failed
glab pipeline jobs 123456

# Stream the logs of the failing job
glab job log 789012

# Retry just that job (not the whole pipeline)
glab job retry 789012

# Or retry the whole pipeline
glab pipeline retry 123456
```

Triggering pipelines manually:

```bash
# On current branch
glab pipeline run

# On a specific branch, with CI variables
glab pipeline run \
  --branch main \
  --variables "DEPLOY_ENV=staging,VERSION=1.2.0"
```

Downloading artifacts (build outputs, test reports):

```bash
# Latest pipeline on current branch
glab pipeline artifact

# From a specific job
glab job artifact 789012
```

---

## Variables and secrets

GitLab CI variables are the equivalent of GitHub Actions secrets. `glab`
handles them well, with fine-grained control over masking and protection.

```bash
# Set a variable (prompts for value, hidden input)
glab variable set MY_API_KEY

# Masked (hidden in job logs) AND protected (only available on protected branches)
glab variable set MY_API_KEY --masked --protected

# List variables — names and metadata, never values
glab variable list

# Delete
glab variable delete MY_API_KEY
```

```
Variable protection levels:
  ┌─────────────┬─────────────────────────────────────────┐
  │  (none)     │  Available in all pipelines             │
  │  masked     │  Hidden in logs, but any branch         │
  │  protected  │  Only on protected branches (e.g. main) │
  │  both       │  Hidden in logs + only on main ← ideal  │
  └─────────────┴─────────────────────────────────────────┘
```

---

## Raw API access

Like `gh api` for GitHub, `glab api` gives access to the full GitLab REST API.

```bash
# Get project info
glab api projects/kpihx%2Ftechskills

# Update project description
glab api projects/kpihx%2Ftechskills \
  --method PUT \
  -f description="Ubuntu live tutorials"

# List project members
glab api projects/kpihx%2Ftechskills/members
```

> **URL encoding:** GitLab API uses `owner%2Frepo` (slash encoded as `%2F`)
> in project path parameters. `kpihx/techskills` becomes `kpihx%2Ftechskills`.

---

## Releases

```bash
glab release create v1.0.0 \
  --name "v1.0.0 — Initial release" \
  --notes "First stable release"

# With milestone
glab release create v1.0.0 \
  --name "v1.0.0" \
  --milestone "v1.0"

# Upload assets to an existing release
glab release upload v1.0.0 dist/myapp-linux-amd64

glab release list
glab release view v1.0.0
glab release download v1.0.0
```

---

## Working with multiple GitLab instances

I have both gitlab.com and a self-hosted instance. `glab` handles this
transparently — it reads the git remote of the current directory:

```bash
glab auth status
# → ✓ Logged in to gitlab.com as kpihx
# → ✓ Logged in to gitlab.homelab.internal as ivann

# In a repo with git@gitlab.com:... remote
glab mr list   # → hits gitlab.com

# In a repo with git@gitlab.homelab.internal:... remote
glab mr list   # → hits self-hosted automatically
```

---

## 🐛 Quick fixes

**Auth fails or token expired**
```bash
glab auth login --hostname gitlab.com
```

**Wrong instance being used**
```bash
# Check what remote glab detected
glab repo view

# Override with env var
GITLAB_HOST=gitlab.homelab.internal glab mr list
```

**Pipeline won't trigger**
Make sure `.gitlab-ci.yml` is at repo root and the branch has CI enabled
in project settings (Settings → CI/CD → General pipelines).

---

## ✅ Quick health check

```bash
glab auth status
glab repo list --limit 5
glab api user | python3 -c "import sys,json; u=json.load(sys.stdin); print(u['username'], '-', u['name'])"
```

---

## 📚 References

- [glab documentation](https://gitlab.com/gitlab-org/cli/-/blob/main/docs/source/index.md)
- [GitLab REST API](https://docs.gitlab.com/ee/api/rest/)
- [glab releases](https://gitlab.com/gitlab-org/cli/-/releases)
- [GitHub CLI (`gh`) guide](dev/gh.md)
