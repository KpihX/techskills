# 🐚 Zsh Startup Files & Environment Injection

> **Machine:** KpihX-Ubuntu (Ubuntu 25.10)
> **Lived on:** 2026-03 · **Status:** Production-stable

---

I had been running bw-env for months and the secret injection worked
perfectly in every terminal I opened. Then I started writing Python scripts
that called `os.environ["SOME_API_KEY"]` — and got `KeyError`. Same machine,
same secrets, same shell. Why?

Later, when I added AI CLI widgets to WaveTerm, the exact same failure
appeared: the CLIs launched but couldn't authenticate. The env was empty.

Both problems have the same root cause: **not all Zsh instances source the
same startup files**. Understanding which file is sourced when — and why —
unlocks the whole picture.

---

## 🗺️ The Triptych: Three Files, Three Contexts

Zsh has three startup files that each target a different kind of shell
invocation. They are not interchangeable. They are not a hierarchy where
the last one "wins" — they are three separate entry points.

```
Shell invocation type          Files sourced
─────────────────────────────────────────────────────────────────
zsh (interactive login)     →  .zshenv  +  .zprofile  +  .zshrc
zsh (interactive non-login) →  .zshenv               +  .zshrc
zsh -l -c 'cmd'             →  .zshenv  +  .zprofile
zsh -c 'cmd'                →  .zshenv
exec-without-shell (binary) →  (nothing)
```

Visual map:

```
                        Every Zsh instance
                               │
                          .zshenv  ← ALWAYS sourced first
                               │
              ┌────────────────┴────────────────┐
         login shell?                     non-login?
         (zsh -l, SSH,                   (new terminal tab,
          TTY, zsh --login)               subshell, script)
              │                                 │
         .zprofile                        (skipped)
              │                                 │
              └────────────────┬────────────────┘
                          interactive?
                         (has a prompt)
                               │
                Yes ──────► .zshrc
                No  ──────► (skipped)
```

---

## 📁 What Each File Is For

### `.zshenv` — The Universal Stub

Sourced by **every** Zsh process without exception: interactive, non-interactive,
login, non-login, scripts called with `zsh script.sh`, subshells, everything.

**What belongs here:**
- `PATH` modifications that scripts need (not just interactive shells)
- Environment variables required by every Zsh invocation
- Language/locale settings

**What does NOT belong here:**
- Secrets or anything that loads slowly (sourced on every `zsh script.sh`)
- Interactive aliases or prompt setup
- Anything that produces output (breaks scripts)

On KpihX-Ubuntu, `.zshenv` is intentionally minimal — it only sources
`.cargo/env` and bun completions. Secrets are deliberately NOT put here
because `.zshenv` is sourced even in untrusted script contexts.

```bash
# ~/.zshenv — current state (KpihX-Ubuntu)
. "$HOME/.cargo/env"
[ -s "/home/kpihx/.bun/_bun" ] && source "/home/kpihx/.bun/_bun"
```

### `.zprofile` — The Login Shell Hook

Sourced only for **login shells** — when Zsh is started as the first shell
of a session: SSH connections, TTY consoles, `zsh -l`, `zsh --login`, and
GUI terminal emulators configured to start login shells.

**What belongs here:**
- Secret injection (bw-env, kshrc) — login implies a trusted, full session
- Tools that need to run once per login (pyenv init, nvm, PATH augmentation)

On KpihX-Ubuntu:

```bash
# ~/.zprofile — current state
[[ -f ~/.kshrc ]] && source ~/.kshrc   # Universal hub → secrets + PATH
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init - zsh)"
```

### `.zshrc` — The Interactive Shell Config

Sourced only for **interactive shells** — shells that have a prompt and
respond to user input. This is what runs when you open a terminal window.

**What belongs here:**
- Prompt (PS1/PROMPT)
- oh-my-zsh, plugins, completions
- Aliases and functions used interactively
- Key bindings, history config

On KpihX-Ubuntu, `.zshrc` also sources `~/.kshrc` at line 8 — so opening
a terminal tab also gets the full secret environment:

```bash
# ~/.zshrc — line 8
[[ -f ~/.kshrc ]] && source ~/.kshrc
# ... then oh-my-zsh, plugins, aliases ...
```

---

## 🔗 The `.kshrc` Universal Hub

`.kshrc` is the single source of truth for all shell-agnostic configuration.
It is sourced by both `.zprofile` (login) and `.zshrc` (interactive), so any
Zsh session that is either login or interactive gets the full environment.

```
~/.zprofile ──┐
              ├──► ~/.kshrc ──► bw-env/shell.sh ──► load.sh ──► secrets from /dev/shm
~/.zshrc ─────┘
```

`.kshrc` contains:
- `source ~/Work/sh/bw-env/shell.sh` — triggers secret injection
- `export NPM_CONFIG_PREFIX`, `NVM_DIR`, `SSH_AUTH_SOCK`, `PATH` augmentations
- Shell-agnostic aliases and functions

**Critical structural note:** `.kshrc` has an interactive guard at line 58:

```bash
case $- in *i*) ;; *) return ;; esac
```

This guard blocks interactive-only content (aliases, prompts) from running
in non-interactive contexts. It MUST be placed **after** all `export`
statements — if placed at the top (like in a standard `.bashrc`), it silently
blocks all exports for non-interactive shells, breaking MCPs and scripts.

---

## 🚨 The Gap: GUI Apps and exec Without a Shell

Here's the concrete problem that triggered this investigation.

When you launch WaveTerm from the desktop (app menu, `.desktop` file, dock),
the desktop session manager starts it with a **minimal environment**. There
is no shell involved. The process inherits only what the display server
provides — typically `HOME`, `USER`, `DISPLAY`, `WAYLAND_DISPLAY`, `PATH`
(system default only), and a handful of XDG variables.

```
Desktop session manager
        │
        ▼
  WaveTerm process ← minimal env: no .zshenv, no .zprofile, no .zshrc
        │
        ├── web block    ✅ no env needed
        ├── term block   ✅ opens zsh → sources .zshrc → gets secrets
        └── cmd block    ⚠️ direct exec → inherits Wave's minimal env
                             → GEMINI_API_KEY missing → auth fails
```

The same gap exists for any GUI app started outside a terminal:
- VS Code launched from the app menu
- JetBrains IDEs
- Any Electron app
- Systemd user services

And for scripts invoked without a login shell:
```python
# script.py — called as: python3 script.py
import os
os.environ["GEMINI_API_KEY"]  # KeyError — not in env
```

---

## 🔧 The Fix: `zsh -l -c`

The fix for both cases — WaveTerm cmd widgets and Python scripts — is to
explicitly invoke a **login shell** that sources `.zprofile` → `.kshrc` →
secrets before executing the target command.

### For WaveTerm widgets

```json
{
  "cmd": "zsh -l -c '/home/kpihx/.local/bin/claude --continue'"
}
```

Execution flow:
```
WaveTerm spawns: zsh -l -c 'claude --continue'
     │
     ├── zsh sources ~/.zshenv  (minimal — cargo, bun)
     ├── zsh sources ~/.zprofile
     │        └── sources ~/.kshrc
     │                 └── sources bw-env/shell.sh
     │                          └── sources load.sh
     │                                   └── source $TEMP_ENV  ← secrets injected
     │
     └── exec: claude --continue  ← inherits full env with API keys ✅
```

**Startup time:** ~100ms (login shell without `-i`) vs ~2–3s (with `-i` which
triggers oh-my-zsh + all plugins). For widgets, `-l` alone is the right choice.

### For Python scripts

```python
import subprocess, os

def get_secret(name: str) -> str:
    result = subprocess.run(
        ["zsh", "-l", "-c", f'printf "%s" "${{{name}}}"'],
        capture_output=True, text=True
    )
    return result.stdout.strip()

api_key = get_secret("GEMINI_API_KEY")
```

Or, if the script is always invoked from a login shell context (e.g., a
terminal), just run it as: `zsh -l -c 'python3 script.py'` and the full
env is inherited.

### For MCP servers

MCPs registered in Claude Code that need secrets follow the same pattern:

```json
{
  "command": "zsh",
  "args": ["-l", "-c", "/home/kpihx/.local/bin/ticktick-mcp serve"]
}
```

This is mandatory for any MCP binary that reads tokens from `os.environ`
at startup — Claude Code itself is launched as a GUI child process and
inherits the same minimal environment as WaveTerm widgets.

---

## 📊 Decision Table

```
┌──────────────────────────────────┬────────────────────────────────────┐
│ Context                          │ Gets secrets?  Fix if not          │
├──────────────────────────────────┼────────────────────────────────────┤
│ Terminal tab (interactive Zsh)   │ ✅ via .zshrc → .kshrc             │
│ SSH session (login Zsh)          │ ✅ via .zprofile → .kshrc          │
│ zsh -l -c 'cmd'                  │ ✅ via .zprofile → .kshrc          │
│ zsh -c 'cmd'                     │ ❌  wrap with: zsh -l -c '...'     │
│ python3 script.py (from terminal)│ ✅ inherited from terminal env     │
│ python3 script.py (from cron)    │ ❌  use subprocess zsh -l -c       │
│ WaveTerm cmd widget (direct exec)│ ❌  wrap with: zsh -l -c '...'     │
│ WaveTerm term block              │ ✅ shell sources .zshrc             │
│ GUI app from desktop             │ ❌  launch from terminal instead    │
│ Systemd user service             │ ❌  use Environment= or zsh -l -c  │
└──────────────────────────────────┴────────────────────────────────────┘
```

---

## ⚡ Quick Reference

```bash
# Check which startup files are sourced — add this temporarily:
echo "sourced: $0" >> /tmp/zsh_trace.log
# Put in .zshenv, .zprofile, .zshrc and watch what appears

# Test what a login shell sees
zsh -l -c 'env | grep -E "API|TOKEN" | cut -d= -f1'

# Test what a plain exec sees
env | grep -E "API|TOKEN" | cut -d= -f1   # run from terminal

# Force a script to run in login shell context
zsh -l -c 'python3 /path/to/script.py'

# Get a single secret from Python without a full login shell subprocess per call
# (cache the result — each zsh -l call costs ~100ms)
import functools, subprocess
@functools.lru_cache(maxsize=None)
def get_secret(name: str) -> str:
    r = subprocess.run(["zsh", "-l", "-c", f'printf "%s" "${{{name}}}"'],
                       capture_output=True, text=True)
    return r.stdout.strip()
```

---

## 🔗 See Also

- [🌊 WaveTerm](waveterm.md) — cmd widget configuration and the `zsh -l -c` pattern applied
- [🔑 bw-env — Secret Injection](bw-env.md) — how secrets reach `/dev/shm` and what `.kshrc` loads
