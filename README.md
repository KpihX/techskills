# TechSkills

> **Ubuntu-focused knowledge base.** Real problems, real fixes, documented as they happen on KpihX-Ubuntu (Ubuntu 25.10).

No theory, no made-up examples. Everything here was lived on my machine and documented after the fact.

---

## 🧰 Reference

| Page | Content |
|------|---------|
| [🧰 Tools & Apps](tools.md) | All installed tools — brief description + install command or tutorial link |

## 📚 Tutorials

| Tutorial | Topic |
|----------|-------|
| [🔒 Tailscale](tailscale.md) | MagicDNS permanent fix, SSH config, Bitwarden SSH agent |
| [🌐 GitHub Pages + Docsify](github-pages.md) | Turn a Markdown repo into a Docsify site — local + online, classic vs `gh` CLI |
| [🐙 GitHub CLI (gh)](gh.md) | Repos, PRs, issues, releases, CI, raw API — all from the terminal |
| [🦊 GitLab CLI (glab)](glab.md) | Repos, MRs, issues, pipelines, CI/CD, secrets — GitLab native CLI |
| [📦 Node.js & npm prefix](npm-prefix.md) | nvm, `.nvmrc`, and the `~/.npm-global` fix for tools that auto-update without nvm sourced |
| [🥟 Bun — JS/TS Runtime](bun.md) | Sovereign JS/TS toolchain — runtime, package manager, test runner, bundler in one binary |
| [🧠 qmd — Local Semantic Search](qmd.md) | AI-powered semantic search for your Markdown notes and code |
| [🐚 Zsh + env](zsh_env.md) | `.zshenv`/`.zprofile`/`.zshrc` triptych, GUI app secret gap, `zsh -l -c` injection pattern |
| [🔑 bw-env — Secret Injection](bw-env.md) | Bitwarden-backed secrets in RAM: unlock, inject, auto-lock on sleep |
| [🌊 WaveTerm](waveterm.md) | Block-based terminal — sidebar widgets, SSH, BYOK AI modes (Groq, Mistral), wsh secrets |
| [💾 Automount External Disk](automount_drive.md) | Universal "lazy" mount via fstab + x-systemd.automount — mount it when you touch it |
| [🧹 Cleanup Suite](clean.md) | Modular disk cleanup — Snap, cache, Docker, AI models, Work artifacts — 4 risk levels |
| [🤖 tg — Telegram CLI](tg.md) | Bot API + Telethon user API — send, read, manage bots and personal chats from the terminal |
| [📧 m365 CLI — Microsoft 365](m365-cli.md) | Mail, calendar, OneDrive via Graph API — device code auth, personal accounts |
| [☁️ Cloudflare CLI (flarectl)](cloudflare.md) | Zone and DNS management from the terminal — CF_API_TOKEN via bw-env |

## 🔌 MCPs

| Tutorial | Topic |
|----------|-------|
| [🔐 bw-mcp](mcps/bw-mcp.md) | Bitwarden AI-blind vault proxy — 7-layer security, ACID transactions, WAL |
| [✅ tick-mcp](mcps/tick-mcp.md) | TickTick MCP — 71 tools, stdio + HTTP transport, V1+V2 API |
| [💬 whats-mcp](mcps/whats-mcp.md) | WhatsApp MCP — 64 tools, Baileys, dual transport, Telegram admin bridge |

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
