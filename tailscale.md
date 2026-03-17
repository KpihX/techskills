# 🔒 Tailscale on Ubuntu 24.04

> **Machine:** KpihX-Ubuntu (Ubuntu 24.04 LTS) · **Node:** `kpihx-labs` (homelab server)
> **Lived on:** 2026-03 · **Status:** Production-stable

---

## 🧩 Context & Problem

Tailscale gives you a private mesh network (MagicDNS, WireGuard overlay) so you can SSH into `kpihx-labs` from anywhere — no port forwarding, no public IP needed.

Two classes of problems came up in practice:

```
Problem ①  MagicDNS intermittently broken
           → `ssh kpihx-labs` resolves to 129.104.201.11 (wrong IP — X network catch-all)
             instead of the Tailscale IP 100.123.81.21

Problem ②  SSH auth requires Bitwarden Desktop popup approval
           → Need SSH agent wired to Bitwarden before git push / SSH

Problem ③  `HostName kpihx-labs` is fragile under X WiFi DNS
           → FQDN `.tail2527bd.ts.net` is immune (routed ONLY via Tailscale)
```

---

## 🏗️ Architecture & Concepts

```
KpihX-Ubuntu
│
├── systemd-resolved  ← stub DNS at 127.0.0.53
│   ├── Global config: /etc/systemd/resolved.conf.d/tailscale.conf
│   │   └── DNS=100.100.100.100   (Tailscale's MagicDNS resolver)
│   │   └── Domains=tail2527bd.ts.net ~ts.net
│   │                              ↑ routing domain: bare ts.net queries
│   │                                go ONLY here, never to upstream
│   └── Per-link config (from NetworkManager / DHCP)
│       └── eleves.polytechnique.fr (from X WiFi DHCP)
│           ← danger zone: if this appears FIRST, kpihx-labs resolves wrong
│
├── Tailscale daemon (tailscaled)
│   ├── Interface: tailscale0
│   ├── MagicDNS resolver: 100.100.100.100
│   └── Search domain: tail2527bd.ts.net
│
└── ~/.kshrc
    └── SSH_AUTH_SOCK → ~/.bitwarden-ssh-agent.sock
        ← Bitwarden Desktop manages SSH keys
```

**Why does X WiFi break MagicDNS?**

`systemd-resolved` builds a search domain list. When X WiFi DHCP pushes `eleves.polytechnique.fr` as a `DefaultRoute=yes` domain, it can appear **before** `tail2527bd.ts.net` in the effective list.

Then `kpihx-labs` → tries `kpihx-labs.eleves.polytechnique.fr` first → gets `129.104.201.11` (X catch-all) → resolves to the WRONG machine.

**The fix:** anchor `100.100.100.100` at the **global** level, which always wins over per-link config in systemd-resolved's priority model.

---

## 🔧 Setup

### 1. Install Tailscale

```bash
# Official one-liner (adds apt repo + GPG key)
curl -fsSL https://tailscale.com/install.sh | sh

# Start and authenticate
sudo tailscale up

# Verify connection + MagicDNS
tailscale status
tailscale ip -4
```

Expected:
```
kpihx-labs  100.123.81.21  linux   active; ...
```

---

### 2. Fix MagicDNS Permanently (DNS Anchoring)

Without this fix, MagicDNS breaks intermittently on any network that pushes its own DNS search domains (eduroam, X WiFi, corporate VPNs).

**Create the override file:**

```bash
sudo mkdir -p /etc/systemd/resolved.conf.d
sudo nano /etc/systemd/resolved.conf.d/tailscale.conf
```

**Content:**

```ini
[Resolve]
DNS=100.100.100.100
Domains=tail2527bd.ts.net ~ts.net
```

> **`~ts.net`** is a *routing domain* — any `*.ts.net` query goes **exclusively** to `100.100.100.100`, never to upstream resolvers. This is the key: no other DNS server will ever intercept Tailscale hostnames.

**Apply without reboot:**

```bash
sudo systemctl daemon-reload
sudo systemctl restart systemd-resolved
```

**Verify:**

```bash
resolvectl status | grep -A5 "Global"
# Should show:
#   DNS Servers: 100.100.100.100
#   DNS Domain:  tail2527bd.ts.net ~ts.net
```

This fix **survives** DHCP renews and WiFi reconnects because `/etc/systemd/resolved.conf.d/` (global config) always takes priority over per-link config injected by NetworkManager.

---

### 3. SSH Config for `kpihx-labs`

```bash
nano ~/.ssh/config
```

```sshconfig
Host kpihx-labs
    HostName kpihx-labs
    # Alternative (FQDN — immune to any DNS interference):
    # HostName kpihx-labs.tail2527bd.ts.net
    User ivann
    Port 2222
    ServerAliveInterval 60
    ServerAliveCountMax 5
```

**FQDN alternative explained:**

```
kpihx-labs                      → resolved via search domain expansion
                                   can hit X network catch-all intermittently

kpihx-labs.tail2527bd.ts.net    → explicit FQDN, hits ~ts.net routing domain
                                   goes ONLY to 100.100.100.100
                                   IMMUNE to any other DNS resolver
```

Use the FQDN if bare hostname remains unreliable despite the DNS fix.

---

### 4. Wire Bitwarden SSH Agent

Bitwarden Desktop manages SSH keys — it acts as a hardware-backed SSH agent. The socket lives at `~/.bitwarden-ssh-agent.sock`.

**Add to `~/.kshrc`** (universal shell hub, sourced by `.zshenv` and `.zprofile`):

```bash
# Bitwarden SSH Agent
export SSH_AUTH_SOCK="$HOME/.bitwarden-ssh-agent.sock"
```

**Reload:**

```bash
source ~/.kshrc
```

**Verify agent is available:**

```bash
echo $SSH_AUTH_SOCK
# → /home/kpihx/.bitwarden-ssh-agent.sock

ls -l ~/.bitwarden-ssh-agent.sock
# → srw------- (socket file, owned by kpihx)

ssh-add -l
# → Lists keys managed by Bitwarden Desktop
```

> ⚠️ **Bitwarden Desktop must be running and unlocked.** First SSH auth (or git push) to a new host triggers a Bitwarden Desktop popup — you must approve it.

---

## 🐛 Debugging

### SSH resolves to wrong IP

```bash
# Check what IP kpihx-labs resolves to
resolvectl query kpihx-labs
# Expected: 100.123.81.21
# Wrong: 129.104.201.11 (X catch-all)

# Check effective DNS config
resolvectl status

# Force Tailscale DNS and check routing domains
resolvectl domain
# Must show: ~ts.net (the routing domain)
```

### SSH auth fails / agent not found

```bash
# Check socket exists
ls -l ~/.bitwarden-ssh-agent.sock

# Check env var
echo $SSH_AUTH_SOCK

# Test auth
ssh -T git@github.com
# → Will trigger Bitwarden popup — approve it
```

### Tailscale IP not reachable

```bash
# Check Tailscale status
tailscale status

# Ping via Tailscale IP directly (bypasses DNS)
ping 100.123.81.21

# Check tailscale interface
ip addr show tailscale0
```

---

## ✅ Verification

```bash
# Full chain test:
ssh kpihx-labs "hostname && ip addr show tailscale0 | grep inet"

# Expected:
# kpihx-labs
# inet 100.123.81.21/32 scope global tailscale0
```

**Persistence test** — after reconnecting to X WiFi:

```bash
# Reconnect WiFi, then:
resolvectl query kpihx-labs
# Must still return 100.123.81.21 — not 129.104.201.11
```

---

## 📚 References

- [Tailscale install docs](https://tailscale.com/kb/1031/install-linux)
- [MagicDNS docs](https://tailscale.com/kb/1081/magicdns)
- [systemd-resolved routing domains](https://www.freedesktop.org/software/systemd/man/resolved.conf.html)
- [Bitwarden SSH agent](https://bitwarden.com/help/ssh-agent/)
