# 🐧 KpihX Ubuntu — Live Tutorials

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
| [🔑 bw-env — Secret Injection](bw-env.md) | Bitwarden-backed secrets in RAM: unlock, inject, auto-lock on sleep |
| [🌊 WaveTerm](waveterm.md) | Block-based terminal — sidebar widgets, SSH, BYOK AI modes (Groq, Mistral), wsh secrets |
| [🧹 Cleanup Suite](clean.md) | Modular disk cleanup — Snap, cache, Docker, AI models, Work artifacts — 4 risk levels |

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
