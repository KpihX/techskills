# 🧰 KpihX Ubuntu — Tools & Apps

> A living inventory of the tools installed on KpihX-Ubuntu (25.10).
> Each entry: what problem it solves + how to install.
> Short installs are inline. Complex setups link to a dedicated tutorial.

---

## ✏️ Editors & Terminal

### micro

A modern terminal text editor. Replaces `nano` and avoids the learning curve
of `vim` for quick edits — mouse support, familiar shortcuts (`Ctrl+S`,
`Ctrl+Z`, `Ctrl+C` to copy), syntax highlighting out of the box.

```bash
sudo apt install micro
```

Set as default editor:

```bash
echo 'export EDITOR=micro' >> ~/.kshrc
```

---

## 🔍 File Search

### plocate

Ultra-fast file search by filename. Replaces `find` for "where is that file"
queries — results in milliseconds via a pre-built index.

```bash
sudo apt install plocate

# Update the index (runs automatically via systemd timer)
sudo updatedb

# Search
plocate tailscale.conf
```

---

## 🧹 System Maintenance

### clean — Unified Cleanup Suite

Modular disk cleanup system for Ubuntu. Five specialized scripts orchestrated
by a single entry point, with four flag levels from read-only analysis to deep
system purge. Covers Snap revisions, package caches (uv, npm, pnpm), Docker,
AI models (Ollama, HuggingFace), and `~/Work/` project artifacts.

```bash
~/Work/sh/clean/clean_all.sh --infos    # full analysis (safe, read-only)
~/Work/sh/clean/clean_all.sh --safe     # weekly safe cleanup
~/Work/sh/clean/clean_all.sh --purge    # deep clean (prompted)
```

→ **Full guide:** [clean.md](clean.md) — module breakdown, flag levels, automation via systemd.

---

## 🌐 Network & Connectivity

### Tailscale

WireGuard-based mesh VPN. Makes any device (homelab, phone, server) reachable
via a stable `hostname` from anywhere — no port forwarding, no public IP,
no VPN server to maintain. Each device gets a `100.x.x.x` IP and a MagicDNS
hostname resolvable from all other Tailscale nodes.

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

→ **Full guide:** [tailscale.md](tailscale.md) — MagicDNS fix, SSH config,
Bitwarden SSH agent, FQDN hardening.

---

## 📦 Node.js

### nvm — Node Version Manager

Manages multiple Node.js versions in isolation. Each project pins its Node
version via `.nvmrc`. Global packages stay per-version — no system Node, no
sudo required.

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.zshrc
nvm install 22
nvm alias default 22
```

→ **Full guide:** [npm-prefix.md](npm-prefix.md) — nvm setup, `.nvmrc` auto-switch hook, the system npm EACCES trap, and the `~/.npm-global` fix for daily-driver CLI tools that auto-update.

---

## 🐍 Python

### uv

Ultra-fast Python package manager and project tool. Single binary, replaces
`pip`, `pipx`, `virtualenv`, `poetry` — with a consistent opinionated
workflow for scripts, tools, and full projects.

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Common use:

```bash
uv tool install <package>          # install CLI tool globally (~/.local/bin)
uv tool install --editable .       # editable install for own packages
uv run --with <lib> script.py      # run script with ad-hoc dependency
uv init --package myproject        # scaffold a new project
```

**Templates:**
- [`pyproject.toml`](templates/pyproject.toml) — KpihX project template: `uv_build` backend, `src/` layout, Typer+Rich, editable install. Copy to project root and fill in the placeholders.

---

## 🐙 Git & Forges

### gh — GitHub CLI

GitHub's official CLI. Create repos, manage PRs, issues, releases, and access
the full GitHub REST API from the terminal. Replaces most github.com browser
interactions.

```bash
# Add apt repo first, then:
sudo apt install gh
gh auth login
```

→ **Full guide:** [gh.md](gh.md) — repos, PRs, issues, releases, API, CI.

### glab — GitLab CLI

GitLab's official CLI. Same philosophy as `gh` but for GitLab — including
native CI/CD pipeline management, job logs, artifacts, and protected variables.

```bash
# Add apt repo first, then:
sudo apt install glab
glab auth login
```

→ **Full guide:** [glab.md](glab.md) — repos, MRs, pipelines, CI/CD, secrets.

---

## 🌐 Web & Publishing

### Docsify (via npx)

Zero-build Markdown site generator. Drop one `index.html` in a repo of
Markdown files, push to GitHub Pages, and it's a full website. No build step,
no CI, no compiled output — Docsify fetches and renders `.md` files
client-side at runtime.

```bash
# No install needed — npx runs it on demand
npx docsify-cli serve .
```

→ **Full guide:** [github-pages.md](github-pages.md) — setup, `.nojekyll` trap, local preview, GitHub Pages activation.

**Templates:**
- [`github-pages-index.html`](templates/github-pages-index.html) — ready-to-use `index.html`: VS Code dark theme, search, sidebar, inline code color fix. Copy to repo root, rename to `index.html`, replace the 4 `SITE_*` placeholders.
- [`_sidebar.md`](templates/_sidebar.md) — generic Docsify sidebar with `.nojekyll` reminder. Copy to repo root as `_sidebar.md`.

---

## 🔐 Secrets & Auth

### bw-env

Shell wrapper that injects Bitwarden vault secrets into RAM at unlock time.
Secrets exist only while the session is active — lock the vault or sleep the
machine and they're purged. The vault lives in a self-hosted Vaultwarden
instance, reachable only via Tailscale.

```bash
bw-env unlock    # prompt for password → inject secrets to RAM
bw-env status    # check lock state and visible secrets
bw-env lock      # purge secrets from RAM
```

→ **Full guide:** [bw-env.md](bw-env.md) — architecture, commands, auto-lock on sleep.
**Repos:** [GitHub](https://github.com/KpihX/bw-env) · [GitLab](https://gitlab.com/kpihx/bw-env)

### Bitwarden CLI (`bw`)

Command-line interface for the Bitwarden / Vaultwarden password manager.
Used by `bw-env` to inject secrets into RAM. The CLI authenticates against
a self-hosted Vaultwarden instance.

```bash
# Download from Bitwarden releases or via snap
sudo snap install bw

# Or via npm (latest)
npm install -g @bitwarden/cli

# Authenticate
bw login
bw unlock
```

### Bitwarden Desktop

Desktop app that acts as a hardware-backed **SSH agent** — SSH keys live in
the vault, exposed through a local socket (`~/.bitwarden-ssh-agent.sock`)
when unlocked. Replaces `ssh-agent` and key files on disk.

Download: https://bitwarden.com/download/ (AppImage or `.deb`)

Wire the SSH agent in `~/.kshrc`:

```bash
export SSH_AUTH_SOCK="$HOME/.bitwarden-ssh-agent.sock"
```

---

## 🖥️ Terminals

### WaveTerm

A modern GPU-accelerated terminal with a block-based layout — each command,
SSH session, or web view lives in its own independently scrollable block.
Sidebar widgets launch AI CLIs (claude, codex, gemini) or open URLs in one
click. BYOK AI chat built in via Groq, Mistral, or any OpenAI-compatible API.

Download the `.deb` from [waveterm.dev](https://waveterm.dev):

```bash
sudo dpkg -i waveterm_*.deb
```

→ **Full guide:** [waveterm.md](waveterm.md) — sidebar widgets, SSH connections, BYOK AI modes (Groq, Mistral), wsh secrets.

**Templates:**
- [`waveterm-widgets.json`](templates/waveterm-widgets.json) — AI CLI sidebar widgets (claude, codex, gemini, copilot, vibe) + GitHub/GitLab web shortcuts. Copy to `~/.config/waveterm/widgets.json`, fill `BINARY_PATH_*` and `SESSION_ID_*`.
- [`waveterm-ai-modes.json`](templates/waveterm-ai-modes.json) — BYOK AI Modes (Groq Scout, Groq Maverick, Mistral Large, Codestral, Pixtral). Set secrets via `wsh secret set`, paste into Wave Config → Wave AI Modes.

### Warp

AI-first terminal for Ubuntu — built-in AI command suggestions, natural-language
command generation (`Ctrl+I`), and collaborative Warp Drive for sharing runbooks.

Download the `.deb` from [warp.dev](https://warp.dev):

```bash
sudo dpkg -i warp-terminal_*.deb
```

---

## 🖥️ Shell

### Zsh + Oh My Zsh

`zsh` is the interactive shell. `oh-my-zsh` adds plugins, themes, and sane
defaults. `~/.kshrc` is the universal hub for secrets and aliases, sourced
by both Zsh and Bash via `~/.zshenv` and `~/.bashrc`.

```bash
sudo apt install zsh
chsh -s $(which zsh)

# Oh My Zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

---

*Add new tools following the format above. For installs that need significant
explanation, create a dedicated `.md` with the narrative approach and link
from this file. See `.agent/AGENT.md` for the `tools.md` maintenance guide.*
