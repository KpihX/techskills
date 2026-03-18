# 📦 Node.js Version & Package Management — nvm, npm, and the Global Prefix Trap

> **Machine:** KpihX-Ubuntu (Ubuntu 25.10)
> **Lived on:** 2026-03 · **Status:** Production-stable

---

When you install Node.js on Ubuntu, you typically end up with whatever version
is in the apt repo — often outdated, always system-managed, and shared with
everything. That works fine for a one-off script. It breaks the moment you have
two projects requiring different Node versions, or when you install a CLI tool
globally and it conflicts with a system package.

The solution most developers land on is **nvm** — the Node Version Manager.
Here's how it works, and why it's the right starting point.

---

## What nvm does

nvm lets you install and switch between multiple Node.js versions on the same
machine, completely isolated from the system's Node.js (if any).

```
Without nvm:
  /usr/bin/node        ← system Node, managed by apt
  /usr/lib/node_modules ← global packages, need sudo
  one version, shared by everything

With nvm:
  ~/.nvm/versions/node/
  ├── v18.20.0/         ← Node 18 LTS
  │   ├── bin/node
  │   ├── bin/npm
  │   └── lib/node_modules/  ← globals for this version only
  ├── v20.18.0/         ← Node 20 LTS
  └── v22.14.0/         ← Node 22 (current)
      ├── bin/node      ← active via PATH when v22 selected
      └── ...
```

Each version is completely self-contained. Global packages installed under
`v22` are invisible to `v18`. No sudo needed — everything lives in your home
directory.

---

## Installing nvm

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

The installer adds a loader block to `~/.zshrc` (or `~/.bashrc`). Reload:

```bash
source ~/.zshrc
```

Verify:

```bash
nvm --version
# → 0.40.1
```

---

## Installing and switching Node versions

```bash
# Install a specific version
nvm install 22

# Install the latest LTS
nvm install --lts

# List installed versions
nvm ls

# Switch to a version for the current session
nvm use 22

# Set a default version for all new shells
nvm alias default 22
```

After `nvm use 22`, the `node`, `npm`, and `npx` in your PATH all come from
`~/.nvm/versions/node/v22.x.x/bin/`. Completely transparent.

---

## Installing global CLI tools with nvm

With nvm active, `npm install -g` installs into the current version's bin
directory — no sudo, no system pollution:

```bash
nvm use 22
npm install -g @openai/codex
# → installed to ~/.nvm/versions/node/v22.14.0/bin/codex
```

This is clean and isolated. The tool is tied to Node v22. If you switch to
v18 for a project, `codex` won't be in PATH unless you install it there too.

That's the intended behavior. For project-specific tools, it's perfect.
For daily-driver CLI tools (AI assistants, build tools), it can become
surprising — `codex` disappears when you `nvm use 18`.

---

## Managing multiple projects with different Node versions

nvm's real power comes from `.nvmrc` files — a one-line file at the project
root that specifies the required Node version:

```bash
# In a project that needs Node 18:
echo "18" > .nvmrc

# Anyone (including you, next month) just runs:
nvm use
# → Found '.nvmrc' with version <18>
# → Now using node v18.20.0
```

You can automate this further — add a hook to your shell that calls
`nvm use` automatically when you `cd` into a directory containing `.nvmrc`:

```bash
# Add to ~/.zshrc (zsh users)
autoload -U add-zsh-hook
load-nvmrc() {
  local nvmrc_path
  nvmrc_path="$(nvm_find_nvmrc)"
  if [ -n "$nvmrc_path" ]; then
    local nvmrc_node_version
    nvmrc_node_version=$(nvm version "$(cat "${nvmrc_path}")")
    if [ "$nvmrc_node_version" = "N/A" ]; then
      nvm install
    elif [ "$nvmrc_node_version" != "$(nvm version)" ]; then
      nvm use
    fi
  fi
}
add-zsh-hook chpwd load-nvmrc
load-nvmrc
```

---

## The problem nvm doesn't solve — and when it bites you

nvm is excellent for project isolation. But it has one structural limitation:
**the global package directory is tied to the Node version.**

This becomes a problem for tools that:
1. Are installed globally (CLI tools you use everywhere, not per-project)
2. Have an **auto-update mechanism that runs `npm install -g` on startup**
3. Run in environments where nvm might not be sourced (system services, cron,
   agent subprocesses, MCP server launchers)

In those cases, `npm install -g` might resolve to a *different* npm than the
one that installed the tool originally — or worse, to the **system npm**
(owned by root, prefix `/usr`), triggering an EACCES permission error.

---

## My specific case: the Codex update loop

Here's exactly what happened on my machine.

I had Codex installed via nvm's Node v22:

```
~/.nvm/versions/node/v22.14.0/bin/codex   ← the binary I use daily
```

On startup, Codex checks for updates and runs `npm install -g @openai/codex`.
This is supposed to update itself. Instead, it was failing with:

```
npm error code EACCES
npm error syscall mkdir
npm error path /usr/local/lib/node_modules
npm error Error: EACCES: permission denied, mkdir '/usr/local/lib/node_modules'
```

The root cause: the update ran in a context where nvm was **not sourced**.
Without nvm, the system `npm` at `/usr/local/bin/npm` was used — with its
default prefix `/usr/local`, which requires root. My user doesn't have root
for npm, so it blew up.

```
What Codex's auto-updater saw:

  Shell context when Codex launched: nvm NOT sourced
            │
            ▼
  npm → /usr/local/bin/npm (system npm)
  npm prefix → /usr/local  (requires sudo)
            │
  npm install -g @openai/codex
            │
            ▼
  EACCES: permission denied ❌
```

---

## The fix: a version-agnostic global prefix

The solution is to give npm a **permanent, user-owned prefix** that doesn't
depend on which Node version is active or whether nvm is sourced.

```bash
# Create the directory
mkdir -p ~/.npm-global

# Tell npm to always use it — this file is read regardless of shell state
echo "prefix=/home/kpihx/.npm-global" >> ~/.npmrc
```

Add it to PATH in `~/.kshrc` (universal shell hub, sourced everywhere):

```bash
# After the nvm block — takes priority over nvm-versioned bin
export PATH="$HOME/.npm-global/bin:$PATH"
```

```
After the fix:

  Any npm invocation (sourced or not, nvm or not)
            │
            ▼
  reads ~/.npmrc → prefix = ~/.npm-global
            │
  npm install -g @openai/codex
            │
            ▼
  ~/.npm-global/bin/codex   ← user-owned, always writable ✅
```

Now Codex's auto-updater writes to `~/.npm-global/bin/codex`, which it owns.
No EACCES. No sudo. Works with or without nvm sourced.

For tools like Codex, Gemini CLI, or any daily-driver binary that auto-updates
or gets used outside nvm context — install into the global prefix, not into nvm:

```bash
npm install -g @openai/codex
# → ~/.npm-global/bin/codex  ✅

# vs the nvm way (version-tied, context-dependent)
nvm use 22 && npm install -g @openai/codex
# → ~/.nvm/versions/node/v22.14.0/bin/codex  (breaks if nvm not sourced)
```

> **Rule of thumb:**
> - Project dependencies → nvm (per-project isolation, `.nvmrc`)
> - Daily-driver CLI tools → `~/.npm-global` (always reachable, auto-update safe)

---

## Migrating existing nvm globals to npm-global

If you already have tools installed in nvm's bin that should live in
`~/.npm-global`, reinstall them with the prefix set:

```bash
# With nvm not active (or in a fresh shell where ~/.npmrc is already set):
npm install -g @openai/codex
npm install -g @google/gemini-cli

# Remove the old nvm-tied binaries to avoid confusion
rm ~/.nvm/versions/node/v22.14.0/bin/codex
rm ~/.nvm/versions/node/v22.14.0/bin/gemini
```

For your own local packages (not from the npm registry), use `npm link`
instead — it's the npm equivalent of `uv tool install --editable .`:

```bash
cd ~/Work/AI/MCPs/my-mcp-server
npm link
# → creates ~/.npm-global/bin/my-mcp-server → ~/Work/AI/MCPs/my-mcp-server
# live source, no reinstall needed on code change
```

---

## ✅ Verification

```bash
# npm uses the right prefix regardless of nvm state
npm config get prefix
# → /home/kpihx/.npm-global

# Binaries land in the right place
npm install -g @openai/codex
which codex
# → /home/kpihx/.npm-global/bin/codex

# nvm still works for project isolation
nvm use 18
node --version
# → v18.x.x (nvm-managed)

npm config get prefix
# → /home/kpihx/.npm-global (unchanged — reads ~/.npmrc)
```

---

---

## ⚠️ Gotcha — the `.npmrc` prefix vs nvm warning (keep `prefix=`, ignore the warning)

Once you set `prefix=~/.npm-global` in `~/.npmrc`, every shell that sources
nvm will print a warning on startup:

```
Your user's .npmrc file has a prefix setting which is incompatible with nvm.
Run `nvm use --delete-prefix v22.x.x --silent` to unset it.
```

**Do not run that command** — it would remove the prefix you just set.

**Do not remove `prefix=` from `~/.npmrc` either** — that approach seems
cleaner (use `NPM_CONFIG_PREFIX` env var instead) but breaks in practice:
`NPM_CONFIG_PREFIX` is only visible to shells that source `~/.kshrc`. Any
tool that auto-updates itself by spawning `npm install -g` in a subprocess
(Codex, Gemini CLI…) won't inherit the env var and will fall back to the
system prefix `/usr/local` → EACCES.

```
Without prefix= in ~/.npmrc:

  Codex auto-updater subprocess
       │  (no env var inherited)
       ▼
  npm → /usr/local/bin/npm
  npm prefix → /usr/local  (system, requires sudo)
       │
  npm install -g @openai/codex
       │
       ▼
  EACCES: permission denied ❌

With prefix= in ~/.npmrc:

  ANY npm invocation (subprocess, MCP, auto-updater, non-interactive shell)
       │
       ▼  reads ~/.npmrc unconditionally
  npm prefix → ~/.npm-global  (user-owned, always writable)
       │
  npm install -g @openai/codex
       │
       ▼
  ~/.npm-global/bin/codex  ✅
```

The nvm warning is **cosmetic** — it appears on every new interactive terminal
but has no functional impact. The correct `~/.npmrc`:

```bash
# ~/.npmrc
# Read by ALL npm invocations regardless of shell/nvm state.
# ⚠️ nvm warns about this — that warning is harmless. Do NOT remove this line.
prefix=/home/kpihx/.npm-global
```

Also keep `NPM_CONFIG_PREFIX` in `~/.kshrc` as a belt-and-suspenders redundancy
for interactive shells — it reinforces the same value at the env level:

---

## ⚠️ Gotcha — the interactive guard that silently breaks everything

If your universal shell hub (`~/.kshrc`) was bootstrapped from a `.bashrc`
template, it likely contains this near the top:

```bash
# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac
```

This causes `.kshrc` to **exit immediately** for any non-interactive shell
(`zsh -l -c`, scripts, MCP server launchers). Everything below — PATH, nvm,
`NPM_CONFIG_PREFIX`, `SSH_AUTH_SOCK` — is silently skipped.

The fix: move that guard to **after** all `export` statements. Only aliases,
history settings, and color support need to be interactive-only.

```bash
# ── Universal exports — always run, all shell types ─────────────────────────

if [ -z "$NVM_LOADED" ]; then           # idempotency guard (prevents double-
    export NVM_DIR="$HOME/.nvm"         # sourcing when zshrc + zprofile both
    [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # source kshrc)
    nvm use default --silent 2>/dev/null
    export NVM_LOADED=1
fi
export NPM_CONFIG_PREFIX="$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
export SSH_AUTH_SOCK=~/.bitwarden-ssh-agent.sock
# ... all other exports ...

# ── Interactive-only — skip for scripts, MCPs, zsh -l -c ────────────────────
case $- in *i*) ;; *) return ;; esac

HISTCONTROL=ignoreboth
alias rm='trash-put'
# ... aliases, colors ...
```

---

## 📚 References

- [nvm repository](https://github.com/nvm-sh/nvm)
- [npm config docs](https://docs.npmjs.com/cli/v10/configuring-npm/npmrc)
