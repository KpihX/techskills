# ☁️ Cloudflare CLI (flarectl)

> **Agent skill (SOT):** `$HOME/.agents/skills/k-cloudflare/SKILL.md` · **Setup:** `k-cloudflare/references/setup.md`  
> **Workstation:** `flarectl` only · **`cloudflared` quarantined** → `k-cloudflare/references/cloudflared-quarantine.md` (quick tunnel / localhost)  
> **Email Routing (live):** `k-cloudflare/references/email-routing-ops.md` — `kpihx@kpihx-labs.com` → Gmail via API (`curl`, not `flarectl`)

Manage Cloudflare DNS, zones, and records from the terminal — no browser, no dashboard. Prod tunnel connector runs on **homelab server**, not the dev laptop.

Managing `kpihx-labs.com` DNS from the browser gets tedious fast — especially when you need to add or update records for Traefik, Let's Encrypt, or new homelab services. `flarectl` is Cloudflare's official Go CLI: it reads a single API token and gives you full zone and DNS management from your terminal.

---

## 🏗️ Architecture

```
flarectl (Go binary)
       │
       │  CF_API_TOKEN env var (injected by bw-env)
       ↓
Cloudflare API v4 (api.cloudflare.com)
       │
       ├── Zone management (list, details)
       ├── DNS records (list, create, update, delete)
       ├── Firewall rules
       └── SSL/TLS settings
```

---

## 📦 Install

`flarectl` is a Go binary — install via `go install`:

```bash
go install github.com/cloudflare/cloudflare-go/cmd/flarectl@latest
```

The binary lands in `~/go/bin/flarectl`. Symlink it to `~/.local/bin/` so it's in PATH everywhere:

```bash
ln -sf ~/go/bin/flarectl ~/.local/bin/flarectl
flarectl --version
```

---

## 🔑 Authentication

`flarectl` reads `CF_API_TOKEN` from the environment. Store it in Bitwarden `GLOBAL_ENV_VARS` and let bw-env inject it:

```bash
# In Bitwarden GLOBAL_ENV_VARS secure note:
CF_API_TOKEN=<your-cloudflare-api-token>
```

After `bw-env unlock`, the token is live in your session. Verify:

```bash
env | grep -E "CF_" | cut -d= -f1
# → CF_API_TOKEN   (value hidden)
```

### Creating an API token (Cloudflare dashboard)

1. `dash.cloudflare.com` → Profile → API Tokens → **Create Token**
2. Use template: **Edit zone DNS** — gives full DNS management
3. Scope: specific zone (`kpihx-labs.com`) or all zones
4. Copy the token → store in Bitwarden

---

## 🌐 Zone management

```bash
# List all zones (domains) in your account
flarectl zone list

# Details for a specific zone
flarectl zone info --zone kpihx-labs.com
```

---

## 🔧 DNS management

```bash
# List all DNS records for a zone
flarectl dns list --zone kpihx-labs.com

# Create an A record
flarectl dns create --zone kpihx-labs.com \
  --name "api" --type A --content "100.x.x.x" --ttl 120

# Create a CNAME
flarectl dns create --zone kpihx-labs.com \
  --name "n8n" --type CNAME --content "homelab.kpihx-labs.com" --ttl 120

# Update a record (by ID — get ID from dns list)
flarectl dns update --zone kpihx-labs.com \
  --id <record-id> --content "100.x.x.x"

# Delete a record
flarectl dns delete --zone kpihx-labs.com --id <record-id>
```

### Common DNS record types for homelab

| Type | Use |
|------|-----|
| `A` | Point subdomain to an IP |
| `CNAME` | Alias one subdomain to another |
| `TXT` | Let's Encrypt DNS-01 challenge, SPF, DKIM |
| `MX` | Mail routing |

```
DNS-01 challenge flow (Traefik + Let's Encrypt):
  Traefik requests certificate
       ↓
  Let's Encrypt asks for TXT _acme-challenge.subdomain
       ↓
  Traefik creates TXT record via CF API (CF_API_TOKEN)
       ↓
  Validation passes → certificate issued
       ↓
  Traefik deletes the TXT record
```

---

## 📋 Useful workflows

### Audit all DNS records

```bash
flarectl dns list --zone kpihx-labs.com | grep -E "(A|CNAME|TXT)"
```

### Bulk-inspect from JSON

```bash
flarectl --format json dns list --zone kpihx-labs.com | \
  python3 -c "
import json,sys
records = json.load(sys.stdin)
for r in records:
    print(f\"{r['type']:6} {r['name']:40} {r['content']}\")
"
```

### Check that a record exists before creating it

```bash
flarectl --format json dns list --zone kpihx-labs.com | \
  python3 -c "
import json,sys
records = json.load(sys.stdin)
existing = [r['name'] for r in records]
print('vault' in str(existing))
"
```

---

## 🔐 Security

The `CF_API_TOKEN` injected by bw-env has scope limited to DNS edit on `kpihx-labs.com` — not account-wide admin. This is the principle of least privilege: if the token leaks (e.g., in shell history), an attacker can only edit DNS on that one zone.

```
Scope strategy:
  CF_API_TOKEN → "Edit zone DNS" permission
  Scoped to:    kpihx-labs.com only (not all zones)
  Lifetime:     permanent (Cloudflare doesn't expire API tokens)
  Storage:      Bitwarden GLOBAL_ENV_VARS → bw-env → RAM only
```

---

## 🗺️ Credentials stored in Vault

| Variable | Purpose | Vault location |
|---|---|---|
| `CF_API_TOKEN` | Cloudflare API token (DNS edit) | `GLOBAL_ENV_VARS` |
