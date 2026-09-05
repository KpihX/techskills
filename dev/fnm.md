# 🦀 Node.js Version Management — fnm (Fast Node Manager)

> **Machine:** KpihX-Ubuntu (Ubuntu 26.04)
> **Live since:** 2026-07-24 — replaced nvm for dynamic version switching
> **Status:** Production-stable

---

fnm is a Rust-based Node version manager. It solves the same problem as nvm
(install and switch Node.js versions) but with two crucial differences:
- **~15ms shell startup** vs ~700ms for nvm (no shell function — it's a binary)
- **Dynamic multishell PATH** — no hardcoded version in `.kshrc`
- **Native `--use-on-cd`** — auto-switches when you `cd` into a project

```
          nvm (bash)                          fnm (Rust)
  ┌─────────────────────┐           ┌──────────────────────────┐
  │ Shell function       │           │ Precompiled binary       │
  │ ~700ms per shell     │           │ ~15ms per shell          │
  │ Must be sourcéd      │           │ eval "$(fnm env ...)"    │
  │ Version hardcoded in │           │ Multishell symlink       │
  │ PATH or .kshrc       │           │ (always dynamic)         │
  └─────────────────────┘           └──────────────────────────┘
```

---

## Why fnm instead of nvm

The main motivation: **dynamic version resolution** without shell startup cost.

With nvm, you had two choices, neither ideal:

| Approach | Startup cost | Version changes | Maintenance |
|----------|-------------|----------------|-------------|
| **Chemin fixe** (`export PATH="$NVM_DIR/.../v24.18.0/bin:$PATH"`) | 0ms | Éditer `.kshrc` à la main | Manuelle |
| **Sourcer nvm.sh** (`. "$NVM_DIR/nvm.sh"`) | ~700ms | `nvm use 26` → immédiat | Aucune |

fnm eliminates the tradeoff:
| Approach | Startup cost | Version changes | Maintenance |
|----------|-------------|----------------|-------------|
| **fnm env** | ~15ms | `fnm use 26` → immédiat | Aucune |

---

## Installation

```bash
curl -fsSL https://fnm.vercel.app/install | bash -s -- --install-dir "$HOME/.local/share/fnm"
```

This installs the `fnm` binary to `~/.local/share/fnm/fnm`.

---

## Shell configuration (`.kshrc`)

fnm is loaded in the universal hub (`.kshrc`) — the same place nvm was before:

```bash
# ~/.kshrc — fnm (Fast Node Manager)
FNM_PATH="$HOME/.local/share/fnm"
if [ -d "$FNM_PATH" ]; then
    export PATH="$FNM_PATH:$PATH"
    eval "$(fnm env --use-on-cd --shell zsh 2>/dev/null)"
fi
```

`--use-on-cd` adds a `chpwd` hook to zsh: when you `cd` into a directory with
`.nvmrc` or `.node-version`, fnm auto-switches to the correct Node version.
If no version file exists, nothing happens — zero overhead per `cd`.

### What `fnm env` actually sets

```bash
$ fnm env --shell zsh

export PATH="/run/user/1000/fnm_multishells/79142_1772644096835/bin:$PATH"
export FNM_MULTISHELL_PATH="/run/user/1000/fnm_multishells/79142_1772644096835"
export FNM_DIR="/home/kpihx/.local/share/fnm"
```

The `multishells/<id>/bin/` directory is a **symlink farm**:
```
/run/user/1000/fnm_multishells/79142_.../bin/
  ├── node → ~/.local/share/fnm/node-versions/v24.18.0/installation/bin/node
  ├── npm  → ~/.local/share/fnm/node-versions/v24.18.0/installation/bin/npm
  └── npx  → ~/.local/share/fnm/node-versions/v24.18.0/installation/bin/npx
```

Each shell gets its **own** multishell directory. When you `fnm use 26`, fnm
creates a **new** multishell pointing to v26 and updates the shell's PATH.
Old multishells accumulate on disk — see "Known issues" below.

---

## Daily commands

```bash
# Install a version
fnm install 24            # latest v24.x
fnm install 22.14.0       # exact version
fnm install --lts         # latest LTS

# Switch version for current shell
fnm use 24
fnm use 22

# Set default for new shells
fnm default 24

# List installed
fnm list                  # or fnm ls

# Remote versions
fnm ls-remote --filter 24

# Uninstall
fnm uninstall 22

# Current active version
fnm current
```

---

## npm-global compatibility

fnm works with npm-global **identically** to nvm. The same `.npmrc` prefix
config applies — no changes needed:

```
# ~/.npmrc
prefix=/home/kpihx/.npm-global
```

```
# ~/.kshrc — belt-and-suspenders for interactive shells
export NPM_CONFIG_PREFIX="$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

Why this works:
- `.npmrc` is read by **every** npm invocation — shell, subprocess, systemd, MCP
- NPM_CONFIG_PREFIX is a redundant safety net for interactive shells
- fnm doesn't warn about the prefix (unlike nvm which prints a warning)
- Global CLIs are accessible from any context, regardless of fnm state

---

## Auto-switch on `cd` (`.nvmrc` / `.node-version`)

With `--use-on-cd`, fnm reads `.nvmrc` or `.node-version` when you change
directories:

```
~ $ node --version                    v24.18.0 (fnm default)

~ $ cd ~/legacy-app/
  → fnm detects .nvmrc = "18"
  → fnm use 18
~/legacy-app $ node --version         v18.20.0 (auto-switch)

~ $ cd ~/modern-app/
  → fnm detects .nvmrc = "22"
  → fnm use 22
~/modern-app $ node --version         v22.14.0 (auto-switch)

~ $ cd
  → fnm detects no .nvmrc
  → stays on default 24
~ $ node --version                     v24.18.0
```

This works with:
- `.nvmrc` (single line: `18`, `22.14.0`, `lts/hydrogen`)
- `.node-version` (same format)
- `package.json` `engines.node` (requires `FNM_RESOLVE_ENGINES=1`)

---

## Using fnm in systemd services (OpenClaw Gateway)

systemd does **not** source `.kshrc`, so it cannot use fnm's multishell PATH.
Use an **absolute path** to the fnm-managed node binary in the service file:

```ini
# openclaw-gateway.service.d/exec-override.conf
[Service]
ExecStart=
ExecStart=/home/kpihx/.local/bin/openclaw gateway --port 18789
Environment=PATH=/home/kpihx/.local/share/fnm/node-versions/v24.18.0/installation/bin:/home/kpihx/.npm-global/bin:/home/kpihx/.local/bin:/usr/local/bin:/usr/bin:/bin
```

The `Environment=PATH` override ensures the gateway finds the correct node
binary, regardless of the user shell's fnm state.

---

## Updating Node.js

```bash
# Install latest v24.x
fnm install 24
fnm default 24

# Verify
node --version     # v24.x.x
fnm current        # v24.x.x
```

The previous version remains on disk and can be listed with `fnm list`.
Old versions can be removed with `fnm uninstall 22` to save space.

---

## Known issues

| Issue | Description | Impact | Mitigation |
|-------|-------------|--------|------------|
| **Multishell accumulation** (#1478, #1515) | Each `fnm env` or `fnm use` creates a multishell symlink in `/run/user/.../fnm_multishells/`. These are never auto-cleaned. Over months, thousands of symlinks accumulate. | Minimal disk waste (tiny symlinks). Occasional `/run/user/` quota concerns. | Cleanup: `rm -rf /run/user/1000/fnm_multishells/` (fnm recreates as needed) |
| **systemd PATH bleed** (#1551) | `fnm env` without interactive guard can leak into systemd user session PATH on KDE. | Low — KπX doesn't use KDE. | Interactive guard in `.kshrc` handles this. |
| **Semver `x` → `0`** (#1532) | `engine.node: "<=24.x.x"` parsed as `<=24.0.0`. | Low — `.nvmrc` with `24` works correctly. | Use `fnm install 24` or `echo "24" > .nvmrc` |
| **npm version tied to default node** (#434) | Unlike nvm, npm version is not isolated per node version. The global npm is shared. | Low — you use a single node version (v24). | Not a problem for KπX. |

---

## Migration from nvm

To migrate from nvm to fnm:

```bash
# 1. Install fnm + desired node version
curl -fsSL https://fnm.vercel.app/install | bash
fnm install 24
fnm default 24

# 2. Switch .kshrc from nvm to fnm
#    → Remove nvm lines, add fnm block (see "Shell configuration" above)

# 3. Update any systemd service paths
#    Replace ~/.nvm/versions/node/... with ~/.local/share/fnm/node-versions/...

# 4. Remove nvm (optional — frees ~300MB)
rm -rf ~/.nvm
```

npm-global tools in `~/.npm-global/` are **unaffected** — no reinstall needed.

```bash
# Verify after migration
fnm current        # → v24.18.0
node --version     # → v24.18.0
npm prefix -g      # → /home/kpihx/.npm-global
npm ls -g --depth=0 | wc -l   # → 15 (all tools present)
```

---

## Comparison: nvm vs fnm on KpihX-Ubuntu

| Aspect | nvm (previous) | fnm (current) |
|--------|---------------|---------------|
| Binary type | Shell function | Rust binary |
| Shell init cost | ~700ms (or 0ms with hardcoded PATH) | ~15ms |
| Version in PATH | Hardcoded `v22.22.1` in `.kshrc` | Dynamic multishell |
| Auto-switch on cd | Manual hook (15 lines) | `--use-on-cd` (native) |
| npm prefix conflict | Warning (cosmetic) | None |
| Dotfiles | `.nvm/` (1.2GB) | `.local/share/fnm/` (~500MB) |
| Global tools | `~/.npm-global/` | `~/.npm-global/` (unchanged) |

---

## References

- [npm-prefix.md](dev/npm-prefix.md) — npm global prefix setup, the EACCES trap, and why `.npmrc` prefix is the correct approach
- [tools.md](tools.md) — tool inventory
- [fnm GitHub](https://github.com/Schniz/fnm) — upstream repository (25K stars)
