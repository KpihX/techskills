# 🔒 Tailscale — SSH to Your Homelab, From Anywhere

> **Machine:** KpihX-Ubuntu (Ubuntu 25.10) · **Node:** `kpihx-labs` (homelab server)
> **Lived on:** 2026-03 · **Status:** Production-stable

---

Every day I connect to my homelab server with one command:

```bash
ssh kpihx-labs
```

Configured once in `~/.ssh/config`, forgotten since. Clean, simple, instant.

Then one evening it stopped working. Not "connection refused" — worse: it
connected to *something*, but not my homelab. A wrong-host-key warning,
a login prompt for a machine I didn't recognize. DNS was lying to me.

That was my introduction to Tailscale.

---

## What Tailscale actually does

Before the fix — let me explain what was broken, and why Tailscale is the right answer.

My homelab sits behind NAT, on a private network I don't fully control. Opening a port and exposing SSH to the internet is not something I want to do. VPNs are an option, but traditional VPNs route *all* traffic through a server, which is slow and complicated to maintain.

Tailscale is different. It builds a **WireGuard mesh** between your devices — each device talks directly to the others over encrypted tunnels, with no central server in the data path. Every machine gets a stable IP in the `100.x.x.x` range and a hostname resolvable via **MagicDNS** — Tailscale's built-in DNS resolver.

```
Without Tailscale:
  KpihX-Ubuntu                          kpihx-labs (homelab)
      │                                       │
      │  ssh kpihx-labs → DNS lookup fails    │
      │  or hits wrong host (NAT, firewall)   │
      └───────────── ❌ ────────────────────►│

With Tailscale:
  KpihX-Ubuntu                          kpihx-labs (homelab)
      │  tailscale0 (WireGuard tunnel)        │
      └──── encrypted, direct, stable ──────►│
            ssh kpihx-labs → MagicDNS
            → Tailscale IP → tunnel ✅
```

The result: `ssh kpihx-labs` works from any network — home, campus WiFi,
phone hotspot — with zero port forwarding, zero public exposure.

---

## Installing Tailscale

The official install script handles everything — GPG key, apt repo, daemon:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Once installed, connect to your tailnet and authenticate:

```bash
sudo tailscale up
```

A URL appears in the terminal. Open it in a browser, log in with your
Tailscale account, and authorize the machine. The daemon stays running
after reboot — nothing else to configure.

Verify both machines are visible to each other:

```bash
tailscale status
```

You should see `kpihx-labs` listed with status `active`. If it's there,
`ssh kpihx-labs` now works from anywhere.

---

## The SSH config

For `ssh kpihx-labs` to work cleanly, the hostname needs an entry in
`~/.ssh/config`. Mine looks like this:

```
Host kpihx-labs
    HostName kpihx-labs
    User ivann
    Port 2222
    ServerAliveInterval 60
    ServerAliveCountMax 5
```

`ServerAliveInterval` keeps the connection from dropping silently on idle
links — important when SSH-ing over cellular or flaky WiFi.

---

## When MagicDNS breaks on campus WiFi

For a few weeks, everything was perfect. Then I went back to campus.

On the École Polytechnique network, `ssh kpihx-labs` started resolving to
a wrong IP again — the same symptom as before Tailscale. But Tailscale was
running, connected, and `kpihx-labs` was showing as `active`.

The problem wasn't Tailscale. It was DNS ordering.

Here's what happens under the hood. Ubuntu uses `systemd-resolved` as a
stub DNS resolver. It builds a search domain list from two sources: your
global config, and whatever the current WiFi's DHCP pushes. On X campus,
the DHCP pushes `eleves.polytechnique.fr` as a `DefaultRoute=yes` domain —
meaning it wants to be consulted first for everything.

```
What systemd-resolved sees on campus WiFi:

  Global:   (nothing configured yet)
  Per-link: eleves.polytechnique.fr  ← DefaultRoute=yes, goes FIRST
            tail2527bd.ts.net        ← Tailscale, pushed second

  Resolution of "kpihx-labs":
    Try kpihx-labs.eleves.polytechnique.fr → campus DNS answers
    → returns some campus IP (catch-all)    → WRONG ❌
    (never reaches Tailscale's resolver)
```

The fix is to anchor Tailscale's DNS resolver at the **global** level —
before any per-link config from DHCP. Global entries always win over
per-link in systemd-resolved's priority model.

---

## The permanent DNS fix

Create one config file:

```bash
sudo mkdir -p /etc/systemd/resolved.conf.d
sudo nano /etc/systemd/resolved.conf.d/tailscale.conf
```

```ini
[Resolve]
DNS=100.100.100.100
Domains=tail2527bd.ts.net ~ts.net
```

Two things happening here:

- `DNS=100.100.100.100` — Tailscale's MagicDNS resolver, set at global level.
- `~ts.net` — a *routing domain*. The tilde prefix means: any `*.ts.net`
  query goes **only** to `100.100.100.100`, never to any other resolver.
  Campus DNS cannot intercept it.

```
After the fix:

  Global:   DNS=100.100.100.100  Domains=tail2527bd.ts.net ~ts.net ← anchored
  Per-link: eleves.polytechnique.fr  ← still pushed by DHCP, but lower priority

  Resolution of "kpihx-labs":
    Global routing domain ~ts.net matches → goes to 100.100.100.100
    → Tailscale resolves → correct Tailscale IP → tunnel ✅
```

Apply and verify:

```bash
sudo systemctl daemon-reload
sudo systemctl restart systemd-resolved

# Confirm global section shows Tailscale's resolver
resolvectl status | grep -A8 "Global"
```

This fix survives DHCP renews and WiFi reconnects — `/etc/systemd/resolved.conf.d/`
has permanent priority over per-link config.

> **`tail2527bd.ts.net`** is your Tailscale tailnet's search domain. Find
> yours in the Tailscale admin panel, or run `tailscale status` — it appears
> in the DNS section. It does not change unless you change your tailnet name.

---

## SSH keys and Bitwarden

With DNS fixed, `ssh kpihx-labs` resolves correctly and Tailscale routes
the connection. But authentication still needs an SSH key.

I don't store SSH keys as files on disk. I use **Bitwarden Desktop** as my
SSH agent — keys live in the vault, encrypted at rest, and Bitwarden
exposes them through a local socket when unlocked.

The socket path is `~/.bitwarden-ssh-agent.sock`. To wire it as the SSH
agent for all shells, I add this to `~/.kshrc` (my universal shell hub,
sourced by both Zsh and Bash):

```bash
# Bitwarden SSH Agent
export SSH_AUTH_SOCK="$HOME/.bitwarden-ssh-agent.sock"
```

Reload and verify:

```bash
source ~/.kshrc
ssh-add -l
# → Lists SSH keys currently loaded from Bitwarden
```

When you first SSH to a new host — or push to GitHub for the first time —
Bitwarden Desktop pops up a confirmation dialog. Approve it, and the
connection proceeds. Subsequent connections to the same host are automatic
until you lock the vault.

> **Bitwarden Desktop must be running and unlocked** for the socket to exist.
> If `ssh-add -l` returns "Could not open a connection to your authentication
> agent", open Bitwarden Desktop and unlock it.

---

## Going fully immune: the FQDN alternative

The DNS fix above solves the ordering problem. But there's a scenario where
even that can fail: if you're running another VPN that overrides all routing,
`systemd-resolved`'s config may be bypassed entirely.

There's a simpler solution: use the **fully qualified domain name** instead
of the bare hostname.

```
kpihx-labs                      → resolved via search domain expansion
                                   can potentially be affected by routing

kpihx-labs.tail2527bd.ts.net    → explicit FQDN, hits ~ts.net routing domain
                                   goes only to 100.100.100.100, no expansion
                                   immune to all other DNS resolvers
```

To use it, update `~/.ssh/config`:

```
Host kpihx-labs
    # Bare hostname (works with the DNS fix above):
    HostName kpihx-labs

    # FQDN alternative (immune to all DNS interference):
    # HostName kpihx-labs.tail2527bd.ts.net

    User ivann
    Port 2222
    ServerAliveInterval 60
    ServerAliveCountMax 5
```

I keep the bare hostname since the `resolved.conf.d` fix is stable enough.
The FQDN is there commented as a fallback.

---

## Verification

```bash
# 1. Tailscale is connected and sees kpihx-labs
tailscale status | grep kpihx-labs

# 2. DNS resolves to a Tailscale IP (100.x.x.x range)
resolvectl query kpihx-labs

# 3. SSH connects to the right machine
ssh kpihx-labs "hostname && tailscale ip -4"
# → kpihx-labs
# → 100.x.x.x

# 4. Fix survives WiFi reconnect (test after switching networks)
resolvectl query kpihx-labs
# → should still return the Tailscale IP, not a campus/ISP catch-all
```

---

## 🐛 Quick fixes

**`resolvectl query kpihx-labs` returns a non-Tailscale IP**
→ `/etc/systemd/resolved.conf.d/tailscale.conf` is missing or not applied.
Recreate it and run `sudo systemctl restart systemd-resolved`.

**`ssh-add -l` says "Could not open a connection to your authentication agent"**
→ Bitwarden Desktop is locked or not running. Unlock it, then retry.

**First SSH to a host hangs or fails**
→ Bitwarden popup is waiting for your approval. Check the desktop — there's
a dialog asking you to confirm the key use.

**`tailscale status` shows `kpihx-labs` as offline**
→ Tailscale is not running on the homelab side. SSH in via another method
and run `sudo systemctl start tailscaled`.

---

## 📚 References

- [Tailscale install docs](https://tailscale.com/kb/1031/install-linux)
- [MagicDNS](https://tailscale.com/kb/1081/magicdns)
- [systemd-resolved routing domains](https://www.freedesktop.org/software/systemd/man/resolved.conf.html)
- [Bitwarden SSH agent](https://bitwarden.com/help/ssh-agent/)
