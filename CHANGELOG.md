# Changelog

## [0.9.1] — 2026-04-07

### Added
- [x] `automount_drive.md` — Universal Auto-Mount for External Disks tutorial: UUID, fstab, x-systemd.automount, manual mount-point creation

---

## [0.9.0] — 2026-03-22

### Added
- [x] `bun.md` — Bun JS/TS runtime: install, TypeScript strict mode, Biome, bun test, bun build, Docker, nvm coexistence
- [x] `desk-mcp` entry in `tools.md` — Desktop automation MCP (screenshot, mouse/keyboard)
- [x] `qmd.md` — QMD: Local AI-powered semantic search engine for notes and code

### Changed
- [x] `README.md`: added `bun.md` and `zsh_env.md` rows to Tutorials table
- [x] `_sidebar.md`: added Bun entry after npm-prefix
- [x] `tools.md`: Bun entry under Node.js section with quick-start and guide link

---

## [0.8.0] — 2026-03-22

### Added
- [x] `tg.md` — Telegram CLI: Bot API + Telethon user API, setup, commands, security
- [x] `m365-cli.md` — Microsoft 365 CLI: Azure App Registration, device code auth, mail/calendar/OneDrive
- [x] `cloudflare.md` — flarectl: DNS management, zone ops, CF_API_TOKEN via bw-env
- [x] `mcps/bw-mcp.md` — bw-mcp tutorial: architecture, 7-layer defense, ACID transactions, WAL
- [x] `mcps/tick-mcp.md` — tick-mcp tutorial: 71 tools, dual transport, API gotchas, workspace map
- [x] `mcps/whats-mcp.md` — whats-mcp tutorial: 64 tools, Baileys, operator surfaces, pairing

### Changed
- [x] `_sidebar.md`: added 3 tool tutorials + new "🔌 MCPs" section with 3 entries
- [x] `README.md`: 3 new tutorial rows + new MCPs table section
- [x] `tools.md`: entries for tg, m365-cli, flarectl, bw-mcp, tick-mcp, whats-mcp

## [0.7.0] — 2026-03-17

### Added
- [x] `clean.md` — Unified Cleanup Suite guide: 5 modules, 4 flag levels, systemd automation

### Changed
- [x] `_sidebar.md`: emoji labels on all Shell Commands entries; `clean.md` added
- [x] `README.md`: `clean.md` row in Tutorials table
- [x] `tools.md`: new `## 🧹 System Maintenance` section with clean entry
- [x] `sh/`: all 9 tutorials rewritten from French to English with ASCII diagrams [subagent]
- [x] `~/Work/sh/` git: committed deletion of `tutos/` folder

## [0.6.0] — 2026-03-17

### Added
- [x] `waveterm.md` — full WaveTerm guide: block paradigm, sidebar widgets, SSH, BYOK AI modes (Groq, Mistral), wsh secrets + embedded screenshots
- [x] `assets/` — images subfolder; `waveterm-connections.png`, `waveterm-secrets.png` (cropped via ffmpeg)
- [x] `sh/` — native shell command tutorials (from `~/Work/sh/tutos/`): awk, sed, grep/regex, tail, ps, kill, xargs, tee, globbing
- [x] `templates/waveterm-widgets.json` — sidebar widgets template (AI CLIs + web shortcuts)
- [x] `templates/waveterm-ai-modes.json` — BYOK AI Modes (Groq, Mistral)
- [x] `templates/_sidebar.md` — generic Docsify sidebar template

### Changed
- [x] `tools.md` updated: new `## 🖥️ Terminals` section (WaveTerm + Warp), Docsify `_sidebar.md` template added
- [x] `_sidebar.md` updated: WaveTerm entry + new `🐚 Shell Commands` section with all 9 tutorials
- [x] `README.md` updated: WaveTerm row + new `🐚 Shell Commands` table
- [x] `templates/README.md` updated: 3 new template rows
- [x] `.agent/AGENT.md` updated: `sh/` placement rule + updated structure diagram [CLAUDE]

## [0.5.0] — 2026-03-17

### Added
- [x] `npm-prefix.md` — nvm full guide + `~/.npm-global` fix for CLI auto-update EACCES
- [x] `bw-env.md` — quick reference + architecture diagram, points to GitHub/GitLab repos
- [x] `tools.md` updated: nvm/npm-prefix entry (new `## 📦 Node.js` section) + bw-env entry (with repo links)
- [x] `_sidebar.md` updated: npm-prefix and bw-env added under Tutorials
- [x] `README.md` updated: two new rows in Tutorials table

## [0.4.0] — 2026-03-17

### Added
- [x] `templates/` — reusable config/boilerplate folder
- [x] `templates/github-pages-index.html` — Docsify site template (VS Code dark, search, inline code fix)
- [x] `templates/pyproject.toml` — KpihX uv project template (`uv_build`, `src/`, Typer+Rich)
- [x] `templates/README.md` — template index
- [x] `tools.md` updated: Templates sections under uv and Docsify
- [x] `github-pages.md` updated: template callout at index.html section
- [x] `.agent/AGENT.md` updated: templates maintenance guide + tools.md rule

## [0.3.0] — 2026-03-17

### Added
- [x] `tools.md` — living inventory of KpihX Ubuntu tools (micro, plocate, Tailscale, uv, gh, glab, Docsify, Bitwarden, Zsh)

### Changed
- [x] All tutorials rewritten in narrative / intuition-formalism mode
- [x] `index.html` inline code color fix (light: `#c7254e` on `#f0f0f0`, dark: `#ce9178` on `#2d2d2d`)
- [x] `.agent/AGENT.md` style guide: narrative approach + `tools.md` maintenance rules

## [0.2.0] — 2026-03-17

### Added
- [x] `gh.md` — GitHub CLI full guide: install, auth, repos, PRs, issues, releases, CI, API
- [x] `glab.md` — GitLab CLI full guide: install, auth, repos, MRs, issues, pipelines, CI/CD, secrets
- [x] `github-pages.md` updated — classic (git + web UI) vs `gh` CLI methods for repo creation + Pages activation

## [0.1.0] — 2026-03-17

### Added
- [x] Project init: `git init`, Docsify setup, agent symlinks (`CLAUDE.md`, `GEMINI.md`, `AGENTS.md`)
- [x] `tailscale.md` — MagicDNS permanent DNS fix, SSH config with Bitwarden SSH agent
- [x] `github-pages.md` — Docsify local setup + GitHub Pages deployment guide
- [x] `README.md`, `TODO.md`, `CHANGELOG.md`, `_sidebar.md`, `index.html`
- [x] GitHub + GitLab public repos created and pushed
