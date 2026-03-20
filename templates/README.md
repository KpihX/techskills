# 📄 Templates

Ready-to-use config and boilerplate files. Copy, rename, fill placeholders.

---

| Template | Tool | Description |
|----------|------|-------------|
| [`github-pages-index.html`](https://github.com/kpihx/tutos_live/blob/master/templates/github-pages-index.html) | Docsify | `index.html` for Docsify + GitHub Pages. VS Code dark theme, search, sidebar, inline code fix. Rename to `index.html`. |
| [`pyproject.toml`](https://github.com/kpihx/tutos_live/blob/master/templates/pyproject.toml) | uv | Python project boilerplate: `uv_build` backend, `src/` layout, Typer + Rich, editable install. |
| [`waveterm-widgets.json`](https://github.com/kpihx/tutos_live/blob/master/templates/waveterm-widgets.json) | WaveTerm | Sidebar widgets: AI CLIs (claude, codex, gemini, copilot, vibe) + GitHub/GitLab web shortcuts. Copy to `~/.config/waveterm/widgets.json`, fill `BINARY_PATH_*` and `SESSION_ID_*`. |
| [`waveterm-ai-modes.json`](https://github.com/kpihx/tutos_live/blob/master/templates/waveterm-ai-modes.json) | WaveTerm | BYOK AI Modes: Groq Scout, Groq 120B, Groq Maverick, Mistral Large, Codestral, Pixtral. Set secrets via `wsh secret set`, paste into Wave Config → Wave AI Modes. |
| [`_sidebar.md`](https://github.com/kpihx/tutos_live/blob/master/templates/_sidebar.md) | Docsify | Generic Docsify sidebar template. Copy to repo root as `_sidebar.md`, create `.nojekyll`, fill placeholders. |
| [`config.py`](https://github.com/kpihx/tutos_live/blob/master/templates/config.py) | Python / uv | Canonical KpihX config skeleton: YAML + .env + login-shell secret resolution (`zsh -l -c`), `@lru_cache` singleton, `deep_merge`, `update_config`, dotted-path `get()`, `SecretsUnavailableError`. Copy to `src/<package>/config.py`, replace ALL_CAPS placeholders. |
| [`config.yaml.example`](https://github.com/kpihx/tutos_live/blob/master/templates/config.yaml.example) | Python / uv | Non-sensitive settings companion for `config.py`. Copy to `src/<package>/config.yaml`, fill project-specific keys. |
| [`config.env.example`](https://github.com/kpihx/tutos_live/blob/master/templates/config.env.example) | Python / uv | Secrets template companion for `config.py`. Copy to `src/<package>/.env`, fill for local dev. Never commit `.env`. |

---

*Each template is self-documented — open it, read the header comment, follow the instructions.*
