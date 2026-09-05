# TechSkills

> **Ubuntu-focused knowledge base.** Real problems, real fixes, documented as they happen on KpihX-Ubuntu (Ubuntu 25.10).

No theory, no made-up examples. Everything here was lived on my machine and documented after the fact.

## 🌐 Live Site

- **GitHub Pages:** [https://kpihx.github.io/techskills/](https://kpihx.github.io/techskills/)

---

## 🧰 Reference

| Page | Content |
|------|---------|
| [🧰 Tools & Apps](tools.md) | All installed tools — brief description + install command or tutorial link |

## 📚 Tutorials

### 🖥️ System

| Tutorial | Topic |
|----------|-------|
| [💾 Automount External Disk](system/automount_drive.md) | Universal "lazy" mount via fstab + x-systemd.automount — mount it when you touch it |
| [🧹 Cleanup Suite](system/clean.md) | Modular disk cleanup — Snap, cache, Docker, AI models, Work artifacts — 4 risk levels |
| [🐚 Zsh + env](system/zsh_env.md) | `.zshenv`/`.zprofile`/`.zshrc` triptych, GUI app secret gap, `zsh -l -c` injection pattern |
| [🗂️ Nautilus Default](system/nautilus_default_file_manager.md) | Restore GNOME Nautilus as default file manager, purge Nemo |
| [🖥️ ASUS VE228 Stretch Fix](system/ubuntu-asus-ve228-stretch-fix.md) | Prevent vertical resolution stretching on Wayland using Tiling Shell |
| [🖼️ Recover deleted images](system/ubuntu-deleted-files-recovery.md) | ext4 undelete runbook — thumbnails, PhotoRec, extundelete |
| [💿 Windows 11 USB Install](system/win11_usb_install.md) | Portable SSD install — installer bug + policy block bypass |
| [🍷 Wine](system/wine.md) | 32-bit isolation and HiDPI display for Windows apps |
| [🔑 bw-env — Secret Injection](system/bw-env.md) | Bitwarden-backed secrets in RAM: unlock, inject, auto-lock on sleep *(planned — see TODO.md)* |

### 🧑‍💻 Dev

| Tutorial | Topic |
|----------|-------|
| [🐙 GitHub CLI (gh)](dev/gh.md) | Repos, PRs, issues, releases, CI, raw API — all from the terminal |
| [🦊 GitLab CLI (glab)](dev/glab.md) | Repos, MRs, issues, pipelines, CI/CD, secrets — GitLab native CLI |
| [🌐 GitHub Pages + Docsify](dev/github-pages.md) | Turn a Markdown repo into a Docsify site — local + online, classic vs `gh` CLI |
| [📦 Node.js & npm prefix](dev/npm-prefix.md) | nvm, `.nvmrc`, and the `~/.npm-global` fix for tools that auto-update without nvm sourced |
| [🥟 Bun — JS/TS Runtime](dev/bun.md) | Sovereign JS/TS toolchain — runtime, package manager, test runner, bundler in one binary |
| [📦 fnm — Fast Node Manager](dev/fnm.md) | Rust Node version manager — dynamic PATH, `--use-on-cd`, replaces nvm |
| [📝 Neovim](dev/nvim.md) | Daily-driver editor — Lazy.nvim, Mason LSP, Telescope, tmux workflow |
| [☁️ Azure CLI](dev/azure-cli.md) | Deactivated — no subscription; `m365` only |

### 🌐 Net

| Tutorial | Topic |
|----------|-------|
| [🔒 Tailscale](net/tailscale.md) | MagicDNS permanent fix, SSH config, Bitwarden SSH agent |
| [☁️ Cloudflare CLI (flarectl)](net/cloudflare.md) | Zone and DNS management from the terminal — CF_API_TOKEN via bw-env |
| [☁️ Cloud hub](net/cloud-hub.md) | Cloud & productivity tool map — skill routing |

### 📱 Apps

| Tutorial | Topic |
|----------|-------|
| [🌊 WaveTerm](apps/waveterm.md) | Block-based terminal — sidebar widgets, SSH, BYOK AI modes (Groq, Mistral), wsh secrets |
| [🤖 Edge agent CDP setup](apps/edge-agent-cdp-setup.md) | Visible Edge profile + MCP bridge for browser automation |
| [📧 m365 CLI — Microsoft 365](apps/m365-cli.md) | Mail, calendar, OneDrive via Graph API — device code auth, personal accounts |
| [🌐 gws CLI — Google Workspace](apps/gws-cli.md) | Gmail, Drive, Calendar, Sheets from the terminal — Discovery-based JSON CLI |
| [🧠 qmd — Local Semantic Search](apps/qmd.md) | AI-powered semantic search for your Markdown notes and code |
| [🤖 tg — Telegram CLI](apps/tg.md) | Bot API + Telethon user API — send, read, manage bots and chats *(planned — see tools.md for repos)* |

## 🔌 MCPs

| Tutorial | Topic |
|----------|-------|
| [🔐 bw-mcp](mcps/bw-mcp.md) | Bitwarden AI-blind vault proxy — 7-layer security, ACID transactions, WAL |
| [✅ tick-mcp](mcps/tick-mcp.md) | TickTick MCP — 71 tools, stdio + HTTP transport, V1+V2 API |
| [💬 whats-mcp](mcps/whats-mcp.md) | WhatsApp MCP — 64 tools, Baileys, dual transport, Telegram admin bridge |

## 🔁 Proxies — the solo successors

MCPs work inside MCP hosts only. The **proxies** port the same catalogs to plain shell binaries — no MCP runtime, baked-in `--help` per action, HITL approval on writes, verified destructive calls. For solo (human + script + cron) usage they are strictly more reliable than the MCPs they replace.

| Tutorial | Topic |
|----------|-------|
| [🔁 Proxies](proxies/proxies.md) | Shared ADN: `do`/`admin`, meta+data envelope, HITL, autosave — learn once, drive all seven |
| [🤖 tg-proxy](proxies/tg-proxy.md) | Telegram Bot API + Telethon user API — 24 actions, folders, webhooks |
| [✅ tick-proxy](proxies/tick-proxy.md) | TickTick V1+V2 — 52 actions, verified composites around silent API drops |
| [🔒 ts-proxy](proxies/ts-proxy.md) | Tailscale API — 44 schema-validated actions, payload-driven CLI |
| [📧 mail-proxy](proxies/mail-proxy.md) | IMAP+SMTP+Zimbra — 37 actions, safety-ladder deletes, send-as aliases |
| [🌐 browser-proxy](proxies/browser-proxy.md) | Visible Edge automation — 71 actions, daemon + extension bridge |
| [💼 link-proxy](proxies/link-proxy.md) | LinkedIn REST v2 — 10 actions, 60-day token ritual |
| [💬 whats-proxy](proxies/whats-proxy.md) | WhatsApp via Baileys — 68 actions, guarded daemon, receipt honesty |

## 🐚 Shell Commands

| Tutorial | Topic |
|----------|-------|
| [awk](sh/awk_tutorial.md) | Column-aware text processing — field extraction, filtering, arithmetic on streams |
| [sed](sh/sed_tutorial.md) | Stream editor — in-place substitution, deletion, line selection |
| [grep & regex](sh/regex_tutorial.md) | Pattern search and regular expressions — universal skill across bash, Python, and more |
| [tail](sh/tail_tutorial.md) | Live log following and last-N-lines extraction |
| [ps](sh/ps_tutorial.md) | Process inspection — listing, filtering, reading process state |
| [kill](sh/kill_tutorial.md) | Signal sending — SIGTERM, SIGKILL, pkill, killall |
| [xargs](sh/xargs_tutorial.md) | Pipe parallelism — turning stdin into command arguments |
| [tee](sh/tee_tutorial.md) | Stream splitting — simultaneously write to file and stdout |
| [globbing vs regex](sh/globbing_vs_regex.md) | Shell glob patterns vs regular expressions — when each applies |

---

## 🧰 Reference Environment

- **Machine:** KpihX-Ubuntu — Ubuntu 25.10
- **Shell:** Zsh + `.kshrc` (universal hub for secrets & aliases)
- **Secrets:** Bitwarden Desktop + `bw-env` (RAM-injected secrets)
- **Network:** Tailscale mesh — homelab node `kpihx-labs`
- **Author:** KAMDEM Ivann (`KpihX` / `KπX`) — École Polytechnique X24
