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

The last piece of the switch was shell wiring — one line in `~/.kshrc` so every env-respecting tool lands in Neovim instead of asking me which editor:

```bash
export EDITOR=nvim
```

`git commit`, `crontab -e`, `visudo` — all of them now open Neovim with my full config. Verify with `echo $EDITOR`. That worked immediately, which made the contrast sharper: the editor was ready in a minute, but *configuring* it took weeks. Which is why the config is centralized (next section) instead of living in my head.

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

Updates follow a fixed ritual: `:Lazy update` pulls, `lazy-lock.json` pins exact versions (versioned in git, so every machine resolves identically), then `:checkhealth` live in a tmux pane. Plugin-side false positives get ignored — I only fix errors from my own config. That discipline exists because an update once broke six plugins at once and I spent a day bisecting by hand; the lockfile plus healthcheck turned "update day" from dread into routine.

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

## 🧩 4. Finding anything (Telescope + Oil + Harpoon)

LSP understands code; Telescope finds everything else. `<leader>ff` fuzzy-finds files, `<leader>fg` live-greps contents — backed by `fd` and `rg`, both mandatory. I know they're mandatory because `ff` once returned zero results on a fresh machine: Debian ships the finder as `fdfind`, not `fd`, and `rg` wasn't installed at all, so both pickers failed *silently*. Now the dependency is documented and symlinked (`~/.local/bin/fd`), and the exclusion map keeps `node_modules`, venvs, and agent runtime dirs out of results.

Oil (`<leader>e`) replaced netrw for directory browsing — it edits directories like buffers (`..` goes up, `gx` opens with the system handler). Rule of thumb I settled on: Oil for browsing, Telescope for searching, Harpoon (`<leader>ha`/`<leader>hl`) for the 4 files I jump between all day. Three tools, three jobs, no overlap.

## 🧩 5. Git, completion, and the safety net

Neogit (`<leader>gs`) is a full magit-style status buffer; gitsigns paints the gutter with per-line blame and hunk actions. Completion is `nvim-cmp` fed by LSP, buffer words, paths, and LuaSnip snippets; `conform.nvim` formats on save (ruff, stylua, prettierd, shfmt — same tools as the LSP table, one source of truth). `trouble.nvim` (`<leader>xx`) collects diagnostics project-wide, `todo-comments.nvim` turns `TODO:`/`FIXME:` into a navigable list.

The UI layer stays quiet on purpose: catppuccin mocha transparent, lualine statusline, which-key showing me what `<leader>` can do next, indent guides I barely notice. Nothing animates, nothing nags.

## 🧩 6. Daily workflow (tmux + nvim + agent)

```
tmux window "dev":
   Pane 1 (75%) → nvim .          Pane 2 (25%) → opencode
   Ctrl+h / Ctrl+l to jump panes (vim-tmux-navigator, same keys as splits)
```

No editor-embedded agent plugin (opencode.nvim was purged) — the agent lives in the next pane, files are the shared surface. Navigation between editor splits and tmux panes uses the *same* keys (`Ctrl+h/j/k/l` via vim-tmux-navigator), so my fingers never care which side of the boundary they're on. `Ctrl+A r` kills whatever blocks a pane and relaunches it from shell history — the universal "unstick" gesture.

My most-used keys (`<Space>` leader):

| Key | Action |
|-----|--------|
| `<leader>ff` / `<leader>fg` | Telescope find files / live grep (`fd` + `rg` backends) |
| `<leader>e` | Oil file explorer (`..` parent, `gx` system-open) |
| `<leader>gs` | Neogit status · `K`/`gd`/`gr` LSP hover/definition/references |
| `<leader>ha` / `<leader>hl` | Harpoon add / menu |
| `dd` / `D` / `cc` / `C` | Delete/change — to FreeDesktop **trash**, not the void |
| `gc` | Open current file in VS Code (cursor position kept) |

The trash integration earned its place the week I `dd`'d a paragraph and `:w`'d before noticing. Deleted text now sits in `~/.local/share/Trash/`, visible from GNOME, surviving restarts.

One more bridge worth naming: `gc` opens the current file in VS Code *at my cursor position*, and `gC` opens the whole Oil directory there. I didn't abandon VS Code — I demoted it to a viewer for the rare case where a GUI helps (image-heavy diffs, unfamiliar codebases). Neovim edits, VS Code exhibits.

## 🔀 7. Before / after

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
