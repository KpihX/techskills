# 🦊 GitLab CLI (`glab`) — Ubuntu 24.04 Field Guide

> **Machine:** KpihX-Ubuntu (Ubuntu 24.04 LTS)
> **Lived on:** 2026-03 · **Status:** Daily driver

---

## 🧩 Context & Problem

```
Problem:  GitLab is not just a GitHub mirror — it has:
          - Native CI/CD (GitLab Pipelines)
          - Merge Requests (not PRs)
          - Container Registry
          - Environments & Deployments

Goal:     Control all of this from the terminal.
          Mirroring the gh workflow, but for GitLab.
```

**`glab`** is the official GitLab CLI — maintained by GitLab Inc. It covers
the full GitLab REST API and integrates tightly with Merge Requests, CI/CD,
and the Container Registry.

---

## 🏗️ Architecture

```
You (terminal)
    │
    ▼
glab CLI  ──── REST API ────▶  gitlab.com (or self-hosted)
    │                               │
    │        SSH (git ops)          │
    └──── git@gitlab.com ◀──────────┘
              │
         Your repos
```

`glab` supports both **gitlab.com** and **self-hosted GitLab** instances —
you can configure multiple hosts simultaneously.

---

## 🔧 Installation

### Option A — Official apt repo (recommended)

```bash
# 1. Add GitLab's apt repo
curl -fsSL https://packages.gitlab.com/install/repositories/gitlab/glab/script.deb.sh \
  | sudo bash

# 2. Install
sudo apt install glab -y

# 3. Verify
glab --version
# → glab version 1.x.x (built xxxx-xx-xx)
```

### Option B — Direct binary

```bash
VERSION="1.53.0"
curl -Lo /tmp/glab.tar.gz \
  "https://gitlab.com/gitlab-org/cli/-/releases/v${VERSION}/downloads/glab_${VERSION}_Linux_x86_64.tar.gz"
tar -xzf /tmp/glab.tar.gz -C /tmp
sudo install /tmp/bin/glab /usr/local/bin/glab
```

### Option C — npm (quickest for testing)

```bash
npm install -g @gitlab/glab
```

---

## 🔐 Authentication

```bash
glab auth login
```

Interactive flow — recommended answers:

```
? What GitLab instance do you want to log into?   gitlab.com
? How would you like to authenticate?             Token
? Paste your authentication token:               [hidden]
? Choose default git protocol:                    SSH
? Authenticate Git with your GitLab credentials?  Yes
```

**Getting a token:** GitLab → User Settings → Access Tokens → New token
- Scopes needed: `api`, `read_user`, `read_repository`, `write_repository`

**Check auth status:**

```bash
glab auth status
# → ✓ Logged in to gitlab.com as kpihx
#   ✓ Git operations for gitlab.com configured to use ssh protocol
```

### Self-hosted GitLab

```bash
glab auth login --hostname gitlab.my-company.com
```

All commands then work against both instances — `glab` picks the right
one based on the repo's remote URL.

---

## 📦 Repositories

### Create a repo

```bash
# Public, from current directory
glab repo create kpihx/my-project --public --description "My project"

# Private
glab repo create kpihx/my-project --private

# Interactive
glab repo create
```

### Clone a repo

```bash
glab repo clone kpihx/tutos_live

# Clone someone else's repo
glab repo clone gitlab-org/gitlab-runner
```

### Fork a repo

```bash
# Fork + clone
glab repo fork gitlab-org/gitlab-runner --clone

# Fork only
glab repo fork gitlab-org/gitlab-runner
```

### List repos

```bash
glab repo list
glab repo list --member          # repos you're a member of
glab repo list --starred         # your starred repos
```

### View repo

```bash
glab repo view kpihx/tutos_live
glab repo view kpihx/tutos_live --web
```

### Archive / delete

```bash
glab repo archive kpihx/old-project
glab repo delete kpihx/test-repo
```

---

## 🔀 Merge Requests (MRs)

GitLab calls them **Merge Requests** — same concept as GitHub PRs.

```
feature branch → MR → review → approve → merge
      │                                     │
   git push                           glab mr merge
```

### Create an MR

```bash
# Interactive (reads branch, suggests title from commits)
glab mr create

# One-liner
glab mr create \
  --title "feat: add tailscale tutorial" \
  --description "Covers MagicDNS fix, SSH, Bitwarden SSH agent" \
  --target-branch main \
  --source-branch feat/tailscale

# Draft MR
glab mr create --draft --title "WIP: new tutorial"

# Push + create MR in one shot
glab mr create --push
```

### List MRs

```bash
glab mr list                         # open MRs
glab mr list --state closed
glab mr list --author kpihx
glab mr list --label "bug"
glab mr list --reviewer kpihx        # MRs assigned to you for review
```

### View an MR

```bash
glab mr view 42
glab mr view 42 --web
```

### Approve / reject

```bash
# Approve
glab mr approve 42

# Revoke approval
glab mr revoke 42
```

### Review — add a note

```bash
glab mr note 42 --message "LGTM! Minor nit on line 42"
```

### Checkout an MR branch locally

```bash
# Fetches the source branch and checks it out
glab mr checkout 42
```

### Merge an MR

```bash
# Standard merge
glab mr merge 42

# Squash
glab mr merge 42 --squash-before-merge

# Rebase
glab mr merge 42 --rebase-before-merge

# Remove source branch after merge
glab mr merge 42 --remove-source-branch

# All in one
glab mr merge 42 --squash-before-merge --remove-source-branch
```

### Close / reopen

```bash
glab mr close 42
glab mr reopen 42
```

---

## 🐛 Issues

### Create an issue

```bash
# Interactive
glab issue create

# One-liner
glab issue create \
  --title "MagicDNS breaks on X WiFi" \
  --description "Resolved by anchoring 100.100.100.100 in resolved.conf.d" \
  --label "bug" \
  --assignee kpihx \
  --milestone "v1.0"
```

### List issues

```bash
glab issue list
glab issue list --label "bug"
glab issue list --assignee kpihx
glab issue list --milestone "v1.0"
glab issue list --state closed
```

### View / comment

```bash
glab issue view 7
glab issue note 7 --message "Fixed in MR !42"
```

### Close / reopen

```bash
glab issue close 7
glab issue close 7 --message "Fixed in MR !42"
glab issue reopen 7
```

---

## ⚡ CI/CD Pipelines

This is where GitLab shines — CI is native, not a bolt-on.

```
git push → .gitlab-ci.yml triggers → pipeline
               │
         ┌─────┴──────┐
       build         test
         │             │
         └─────┬───────┘
             deploy
```

### View pipelines

```bash
# List pipelines for current branch
glab pipeline list

# List for a specific branch
glab pipeline list --branch main

# View a specific pipeline
glab pipeline view 123456

# Open in browser
glab pipeline view 123456 --web
```

### Watch a pipeline live

```bash
# Streams status until completion
glab pipeline ci view
```

```
Pipeline #123456 — running
  ✔ build:linux          1m23s
  ⚙ test:unit            running...
  ✔ test:lint            42s
  ⏸ deploy:staging       pending
```

### Trigger a pipeline

```bash
# On current branch
glab pipeline run

# On a specific branch with variables
glab pipeline run \
  --branch main \
  --variables "DEPLOY_ENV=staging,VERSION=1.2.0"
```

### Re-run / cancel

```bash
glab pipeline retry 123456    # retry all failed jobs
glab pipeline cancel 123456   # cancel a running pipeline
```

### View individual job logs

```bash
# List jobs in a pipeline
glab pipeline jobs 123456

# Stream logs of a specific job
glab job log 789012

# Retry a single job
glab job retry 789012

# Cancel a single job
glab job cancel 789012
```

### Download artifacts

```bash
# Download artifacts from the latest pipeline on current branch
glab pipeline artifact

# From a specific branch
glab pipeline artifact --branch main

# From a specific job
glab job artifact 789012
```

---

## 🚀 Releases

### Create a release

```bash
glab release create v1.0.0 \
  --name "v1.0.0 — Initial release" \
  --notes "First stable release"

# With milestone
glab release create v1.0.0 \
  --name "v1.0.0" \
  --milestone "v1.0"

# Upload assets
glab release upload v1.0.0 dist/myapp-linux-amd64
```

### List / view

```bash
glab release list
glab release view v1.0.0
```

### Download

```bash
glab release download v1.0.0
```

### Delete

```bash
glab release delete v1.0.0
```

---

## ⚙️ Variables & Secrets

CI/CD variables (equivalent to GitHub secrets for pipelines):

```bash
# Set a variable (prompts for value, hidden)
glab variable set MY_API_KEY

# Set with value inline (careful: shell history)
glab variable set MY_API_KEY --value "abc123"

# Set as file variable (for certs, keys)
glab variable set MY_CERT < mycert.pem

# Masked (hidden in logs) + protected (only on protected branches)
glab variable set MY_API_KEY \
  --masked --protected

# List variables (names + metadata, never values)
glab variable list

# Delete a variable
glab variable delete MY_API_KEY
```

---

## 🔧 Raw API Access

Same as `gh api` — full access to GitLab REST API:

```bash
# Get project info
glab api projects/kpihx%2Ftutos_live

# Enable Pages (if you have a Pages-capable GitLab instance)
glab api projects/kpihx%2Ftutos_live/pages \
  --method POST

# List project members
glab api projects/kpihx%2Ftutos_live/members

# Update project settings
glab api projects/kpihx%2Ftutos_live \
  --method PUT \
  -f description="Ubuntu field notes"
```

> **URL encoding:** GitLab API uses `owner%2Frepo` (slash encoded as `%2F`)
> for project paths in URLs.

---

## 🛠️ Useful Shortcuts

```bash
# Open current repo in browser
glab repo view --web

# Open current MR in browser
glab mr view --web

# List labels for current repo
glab label list

# Create a label
glab label create "docs" --color "#0075ca" --description "Documentation changes"

# List milestones
glab milestone list

# Create a milestone
glab milestone create --title "v1.0" --description "First stable release"
```

---

## 🔄 Multi-host Workflow

If you have both `gitlab.com` and a self-hosted instance:

```bash
# Login to self-hosted
glab auth login --hostname gitlab.company.com

# Status for both
glab auth status

# glab auto-detects which host to use from git remote
cd ~/Work/company-project
glab mr list   # → hits gitlab.company.com

cd ~/Work/tutos_live
glab mr list   # → hits gitlab.com
```

---

## 🐛 Debugging

### Auth fails / token expired

```bash
glab auth login --hostname gitlab.com
# → re-authenticate, overwrites stored token
```

### Wrong remote detected

```bash
# Check what remote glab is using
glab repo view

# Override host
GITLAB_HOST=gitlab.company.com glab mr list
```

### API rate limit

```bash
# GitLab rate limit info is in response headers — use verbose mode
glab api projects/kpihx%2Ftutos_live --verbose 2>&1 | grep -i rate
```

---

## ✅ Verification

```bash
# Full health check
glab auth status
glab repo list --limit 5
glab api user | python3 -c "import sys,json; u=json.load(sys.stdin); print(u['username'], u['name'])"
```

---

## 📚 References

- [glab documentation](https://gitlab.com/gitlab-org/cli/-/blob/main/docs/source/index.md)
- [GitLab REST API](https://docs.gitlab.com/ee/api/rest/)
- [glab releases](https://gitlab.com/gitlab-org/cli/-/releases)
