# 📝 Neovim — The Daily-Driver Editor (v0.12.0-dev)

> **Machine:** KpihX-Ubuntu · **Install:** apt / source build · **Config source of truth:** `k-nvim/assets/` (symlinked as `~/.config/nvim/`)
> **Skill:** `k-nvim` (v3.9.0) — full hub: plugins, LSP map, keymaps, healthcheck protocol

I used to edit everything in VS Code and drop to `nano` on servers. Two problems: VS Code needs a display and a project window for every quick config tweak, and `nano` has no LSP, no fuzzy search, no memory. I wanted one editor that opens instantly in any terminal, understands every language I touch, and shares its config across machines as versioned files. That editor is Neovim — with Vim kept as the universal fallback.

## 🧩 1. Two editors, two jobs

```
nvim  ── daily driver ──  Lazy.nvim (~38 plugins), Mason LSP, Telescope, tmux-aware
vim   ── fallback ──      SSH, containers, visudo, crontab -e (vim-plug: surround, commentary, fugitive, repeat)
```

Vim is deliberately dumb: four plugins, zero LSP. It exists so I can edit a file on any box without installing anything. Neovim is where real work happens.

## 🧩 2. Centralized config (edit once, symlinked everywhere)

I broke my config twice by editing `~/.config/nvim/` on two machines. Now the canonical source lives in the agents kernel and the runtime path is just symlinks:

```
k-nvim/assets/                  ← SOURCE OF TRUTH (edit here)
├── init.lua                    ← entrypoint, Lazy.nvim bootstrap
├── lazy-lock.json              ← plugin lockfile (versioned)
└── lua/kpihx/
    ├── plugins.lua             ← full Lazy plugin spec
    ├── lsp.lua                 ← LSP wiring
    ├── keymaps.lua             ← custom keybindings
    ├── tmux-navigator.lua      ← seamless Ctrl+h/j/k/l nvim ↔ tmux
    └── trash.lua               ← FreeDesktop trash for dd/D/cc/C

~/.config/nvim/                 ← SYMLINKS (read by nvim, never edited directly)
~/.local/share/nvim/lazy/       ← Lazy-managed plugin code (read-only, wiped on update)
```

⚠️ Rule I enforce on myself: never touch `~/.local/share/nvim/lazy/` — `:Lazy update` wipes it. I own `k-nvim/assets/` and nothing else.

## 🧩 3. LSP map (Mason + one uv exception)

```bash
nvim +Mason          # manage servers, formatters, linters
nvim +Lazy           # plugin manager
nvim                # :checkhealth after ANY config change
```

| Language | Server | Formatter | Install |
|----------|--------|-----------|---------|
| Python | basedpyright | ruff | `uv tool install basedpyright` (own bundled Node, NOT Mason) |
| TypeScript/JS | ts_ls | prettier | `:MasonInstall typescript-language-server` |
| Lua | lua_ls | stylua | `:MasonInstall lua-language-server` |
| Bash | bashls | shfmt | `:MasonInstall bash-language-server` |
| Markdown | marksman | — | `:MasonInstall marksman` |
| LaTeX | texlab | — | `:MasonInstall texlab` |
| JSON / YAML | jsonls / yamlls | — | `:MasonInstall json-lsp` / `yaml-language-server` |

That worked — until Python completions silently died. The cause: plugins needing Python were using system Python. The fix is a dedicated uv venv (`~/.local/share/nvim/uv-nvim/.venv/`, defined in `k-nvim/assets/uv-nvim/pyproject.toml`) wired as `python3_host_prog`. Zero system Python, zero pip — full story in the skill's `references/python-provider.md`.

## 🧩 4. Daily workflow (tmux + nvim + agent)

```
tmux window "dev":
   Pane 1 (75%) → nvim .          Pane 2 (25%) → opencode
   Ctrl+h / Ctrl+l to jump panes (vim-tmux-navigator, same keys as splits)
```

No editor-embedded agent plugin (opencode.nvim was purged) — the agent lives in the next pane, files are the shared surface. My most-used keys (`<Space>` leader):

| Key | Action |
|-----|--------|
| `<leader>ff` / `<leader>fg` | Telescope find files / live grep (`fd` + `rg` backends) |
| `<leader>e` | Oil file explorer (`..` parent, `gx` system-open) |
| `<leader>gs` | Neogit status · `K`/`gd`/`gr` LSP hover/definition/references |
| `<leader>ha` / `<leader>hl` | Harpoon add / menu |
| `dd` / `D` / `cc` / `C` | Delete/change — to FreeDesktop **trash**, not the void |
| `gc` | Open current file in VS Code (cursor position kept) |
| `export EDITOR=nvim` | In `~/.kshrc` — git, crontab, visudo all land in Neovim |

The trash integration earned its place the week I `dd`'d a paragraph and `:w`'d before noticing. Deleted text now sits in `~/.local/share/Trash/`, visible from GNOME, surviving restarts.

## 🔀 5. Before / after

```
BEFORE                                    AFTER (nvim + vim fallback)
──────                                    ────────────────────────────
VS Code per tweak, nano on servers,       one terminal editor everywhere,
config drift across machines,             versioned centralized config,
lost deletes, no cross-pane flow          LSP 8 languages, trash safety,
                                          tmux pane = agent copilot
```

## 📚 References

- Skill (hub): `k-nvim/SKILL.md` + `references/` (telescope, python-provider, healthcheck-protocol, trash-system)
- After any config change: `:checkhealth` live in tmux, fix KπX-side errors only
