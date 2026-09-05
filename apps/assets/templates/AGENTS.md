# 📄 Templates

Ready-to-use config and boilerplate files. Copy, rename, fill placeholders.

---

| Template | Tool | Description |
|----------|------|-------------|
| [`github-pages-index.html`](https://gitlab.com/kpihx/agents/-/blob/main/skills/k-git-pages/assets/github-pages-index.html) | Docsify | `index.html` for Docsify + GitHub Pages. VS Code dark theme, search, sidebar, inline code fix. Rename to `index.html`. Source of truth: `.agents/skills/k-git-pages/assets/github-pages-index.html` (unplugged symlink 2026-08-15 — reference the gitlab URL above). |
| [`pyproject.toml`](https://gitlab.com/kpihx/agents/-/blob/main/skills/k-project/assets/pyproject.toml) | uv | Python project boilerplate: `uv_build` backend, `src/` layout, Typer + Rich + pyyaml + python-dotenv, authors, license. Source of truth: `.agents/skills/k-project/assets/pyproject.toml` (unplugged symlink 2026-08-15 — reference the gitlab URL above). |
| [`waveterm-widgets.json`](https://github.com/kpihx/techskills/blob/master/apps/assets/templates/waveterm-widgets.json) | WaveTerm | Sidebar widgets: AI CLIs (claude, codex, gemini, copilot, vibe) + GitHub/GitLab web shortcuts. Copy to `~/.config/waveterm/widgets.json`, fill `BINARY_PATH_*` and `SESSION_ID_*`. |
| [`waveterm-ai-modes.json`](https://github.com/kpihx/techskills/blob/master/apps/assets/templates/waveterm-ai-modes.json) | WaveTerm | BYOK AI Modes: Groq Scout, Groq 120B, Groq Maverick, Mistral Large, Codestral, Pixtral. Set secrets via `wsh secret set`, paste into Wave Config → Wave AI Modes. |
| [`_sidebar.md`](https://gitlab.com/kpihx/agents/-/blob/main/skills/k-git-pages/assets/_sidebar.md) | Docsify | Generic Docsify sidebar template. Source of truth: `.agents/skills/k-git-pages/assets/_sidebar.md` (unplugged symlink 2026-08-15 — reference the gitlab URL above). |
| [`config.py`](https://gitlab.com/kpihx/agents/-/blob/main/skills/k-project/assets/config.py) | Python / uv | Canonical KpihX config skeleton: YAML + .env + login-shell secret resolution (`zsh -l -c`), `@lru_cache` singleton, `deep_merge`, `update_config`, dotted-path `get()`, `SecretsUnavailableError`. Source of truth: `.agents/skills/k-project/assets/config.py` (unplugged symlink 2026-08-15 — reference the gitlab URL above). |

---

*Each template is self-documented — open it, read the header comment, follow the instructions.*
