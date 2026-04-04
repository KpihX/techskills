# 🔐 bw-env — Bitwarden-backed Secret Injection

> **Machine:** KpihX-Ubuntu (Ubuntu 25.10)
> **Lived on:** 2026-03 · **Status:** Production-stable

---

I don't keep secrets in `.env` files, shell history, or environment variables
that persist across reboots. The attack surface for those is too large —
files can leak, history can be read, exported vars survive in process trees
longer than expected.

The approach I use: secrets live in **Vaultwarden** (self-hosted Bitwarden),
and get injected into RAM at unlock time via `tmpfs` (`/run/user/...`).
They exist only while the session is active. Lock the vault — or sleep the
machine — and they're gone.

That's what **`bw-env`** does. It's a thin shell layer on top of the
Bitwarden CLI that handles the unlock/inject/revoke lifecycle.

---

## The full documentation lives in the repo

Everything — architecture, install, config, commands, daemon setup,
auto-lock on sleep — is documented in the project README:

**GitHub:** [github.com/KpihX/bw-env](https://github.com/KpihX/bw-env)
**GitLab:** [gitlab.com/kpihx/bw-env](https://gitlab.com/kpihx/bw-env)

---

## Quick reference

```bash
bw-env unlock    # prompt for password → sync vault → inject secrets to RAM
bw-env status    # check lock state and which secrets are visible
bw-env sync      # force re-sync with Vaultwarden without full unlock
bw-env lock      # purge secrets from RAM + trigger global revocation
bw-env restart   # restart the background sync daemon (systemd)
bw-env logs      # view recent log entries (alias: bw-env logs -n 50)
```

---

## How it fits into the shell architecture

`bw-env unlock` is called once per session — from `~/.kshrc`, which is
sourced by every shell (Zsh via `~/.zshenv`, Bash via `~/.bashrc`).
After unlock, secrets are available as environment variables in all
subsequent shells and subprocesses.

```
┌─────────────────────────────────────────────────────┐
│  bw-env unlock                                      │
│       │                                             │
│       ▼                                             │
│  Bitwarden CLI ──── HTTPS ────► Vaultwarden         │
│  (bw sync + bw unlock)          (self-hosted)       │
│       │                                             │
│       ▼                                             │
│  Secrets written to RAM (tmpfs, not disk)           │
│  └── available as $ENV_VARS in all shells           │
│                                                     │
│  On lock / sleep / screen-lock:                     │
│  └── D-Bus hook triggers bw-env lock               │
│      └── secrets purged from RAM                   │
└─────────────────────────────────────────────────────┘
```

> **Vaultwarden** (`vault.kpihx-labs.com`) is reachable via Tailscale only —
> intentionally private. If Tailscale is down, `bw-env unlock` will fail.
> See [tailscale.md](tailscale.md) for the connectivity setup.

---

## Gotchas & Recovery Runbook

### "Incorrect Master Password" — but the password is right

This one is confusing. `bw-env unlock` can return "Authentication failed:
Incorrect Master Password" even when the password is correct, if the `bw`
CLI has lost its session.

**Root cause:** `bw-env` calls `bw unlock --raw` internally. If the CLI is
in `unauthenticated` state (e.g., after a RAM wipe, a machine restart, or
first install), `bw unlock --raw` exits non-zero with "You are not logged in."
— which `bw-env` catches as an auth failure.

**The fix:** re-authenticate the CLI independently of the desktop app.
The Bitwarden desktop app and the `bw` CLI are completely separate processes
with separate sessions. One being unlocked does not help the other.

```
Bitwarden Desktop ─── manages → ~/.config/Bitwarden/   (app session)
bw CLI            ─── manages → ~/.config/Bitwarden CLI/ (CLI session)
                                ↑
                         These are INDEPENDENT
```

```bash
# 1. Point the CLI to your self-hosted Vaultwarden (only once)
bw config server https://vault.kpihx-labs.com

# 2. Log in (uses email + master password, creates a local CLI session)
bw login

# 3. Now bw-env unlock works normally
bw-env unlock
```

After `bw login`, the CLI caches a local vault copy. `bw-env unlock` calls
`bw unlock --raw` on that cache — it never needs to hit the network again
until you explicitly `bw sync`.

---

### Tray icon stuck red after recovery

The tray polls every 30 seconds. A stuck red state after a recovery is
cosmetic — just restart the tray:

```bash
systemctl --user restart bw-env-tray.service
# or
bw-env tray restart
```

If the tray was running before the latest `bw-env` update (install, sync,
or `bw-env unlock` changes), restart anyway so it picks up the new binary.

---

### Cold-start / offline recovery (no live Vaultwarden connection)

If you have an encrypted local backup and the vault is unreachable:

```bash
bw-env decrypt
```

This re-derives the GPG key from your master password (PBKDF2 100k
iterations + INTERNAL_SALT) and restores secrets to RAM from the encrypted
local cache at `~/.bw/env/cache.env.gpg`. No network required.

The daemon will stay `PAUSED` until a live `bw login + bw-env unlock` is done.

---

## Resource profile (after optimization)

The initial implementation used `bw status` (spawns Node.js, ~4s cold start)
for every hot-path check — idempotency guards, status polling, lock checks.
Under the 5-second tray polling interval this caused a Node.js process accumulation
and multi-GB RAM leak over hours.

**After 2026-04 optimization:**

```
Hot-path check            Before              After
──────────────────────────────────────────────────────
status --json             ~4s + Node.js       ~226ms, 0 Node.js
unlock idempotency        bw status           [[ -f $TEMP_ENV ]] && [[ -f $SESSION_FILE ]]
lock idempotency          bw status           [[ ! -f $TEMP_ENV ]] && [[ ! -f $SESSION_FILE ]]
tray poll interval        5s                  30s
subprocess timeout        none (leak!)        10s (TimeoutExpired caught)
```

The only remaining `bw status` call is the **unauthenticated guard** before
`bw unlock --raw` — user-initiated, rare, intentional.

**RAM profile (tray process):**
- Baseline: ~180MB RSS (Python + GTK + AppIndicator — this is the stack minimum)
- No longer grows over time with correct timeout + 30s interval
- Peak was 646MB after 6h with the 5s/no-timeout configuration
