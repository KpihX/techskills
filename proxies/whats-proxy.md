# 💬 whats-proxy — WhatsApp Without Touching Your Phone (v0.7.2)

> **Machine:** KpihX-Ubuntu · **Install:** `bun link` in repo (Bun ≥ 1.1, runs on Node+tsx) · **Auth:** QR / pairing-code, Baileys session store
> **Repos:** [GitHub](https://github.com/kpihx-labs/whats-proxy) · [GitLab](https://gitlab.com/kpihx-labs/proxies/whats-proxy)

WhatsApp has no usable official API for individuals — the protocol is proprietary, business accounts only. The community answer is [Baileys](https://github.com/WhiskeySockets/Baileys), a reverse-engineered library that needs a *persistent local store*. An MCP server wrapping it (`whats-mcp`, 64 tools) worked inside agents and nowhere else. `whats-proxy` keeps Bun + Baileys for the store and exposes everything — 68 actions — as a flat CLI with a background daemon.

## 🧩 1. The daemon (the one piece you must respect)

```
first `do` call ──▶ daemon auto-spawns ──▶ serves the socket
      │                                            │
      └── second spawn attempt exits quietly ──────┘ (spawn guard, no hijack)
```

```bash
whats-proxy admin auth login                 # QR pairing
whats-proxy admin auth login --code          # phone-code pairing instead
whats-proxy admin auth login --start-service # pair + start daemon immediately
whats-proxy admin service status             # daemon + auth state
```

⚠️ Session credentials live in `~/.local/share/whats-proxy/<phone>/state/` — **never delete this folder** or you re-pair from scratch. `admin backup [phone]` takes an atomic VACUUM backup; that's the only safe copy.

That worked — until two daemons fought over one session. The spawn guard fixed it: if a daemon already serves the socket, newcomers exit quietly instead of hijacking.

## 🧩 2. Action map (68 actions)

| Domain | Actions |
|--------|---------|
| Chats | `chat-list` · `chat-read` · `chat-read-batch` · `chat-manage` (archive/pin/mute/…) · `chat-star` · `chat-disappearing` (24h/7d/90d) · `message-status` (delivery/read receipts) · `read-messages` |
| Send | `send-text` · `send-image` · `send-video` · `send-audio` · `send-document` · `send-sticker` · `send-contact` · `send-location` · `send-poll` · `send-reaction` · `send-batch` · `media-upload` · `media-download` · `forward-message` · `edit-message` · `delete-message` |
| Groups | `group-list` · `group-info` · `group-create` · `group-disband` · `group-leave` · `group-subject` · `group-description` · `group-settings` · `group-invite` · `group-participants` · `group-picture` |
| Communities | `community-list` · `community-info` · `community-create` · `community-leave` · `community-subject` · `community-description` · `community-invite` · `community-join` · `community-link` · `community-unlink` · `community-groups` · `community-participants` · `community-pending` |
| Contacts | `contact-list` · `contact-info` · `contact-check` (is it on WhatsApp?) · `contact-block` · `contact-business` · `contact-picture` · `contact-presence-check` · `contact-tags` |
| Profile / Presence | `profile-name` · `profile-about` · `profile-picture` · `profile-privacy` · `presence` · `connection-status` |
| Digest | `whatsup` (intent-first overview) · `find-messages` |
| Stories | `story-list` · `story-view` · `story-download` |
| Escape hatch | `raw` |

## 🧩 3. A real session

```bash
whats-proxy do chat-list
whats-proxy do chat-read '{"jid":"33612345678@s.whatsapp.net","limit":20}'
whats-proxy do send-text '{"jid":"33612345678@s.whatsapp.net","text":"Hello from the shell"}'
```

Receipt honesty, baked into `chat-read --help` so agents stop overclaiming: in 1:1 chats, `read_by` populates **only if the recipient enabled read receipts** (`read_receipts=all`); group receipts ignore that setting. Missing `read_by` means *unknown* — never *unread*.

## 🔀 4. Before / after

```
BEFORE (whats-mcp)                        AFTER (whats-proxy)
──────────────────                        ────────────────────
64 tools, MCP host required,              68 actions + raw Baileys hatch,
no daemon story,                          guarded daemon, atomic backups,
receipt semantics tribal knowledge        receipt caveats in --help
```

## 📚 References

- Full catalog: `whats-proxy do --help` · per-action docs: `whats-proxy do <action> --help`
- Live validation (needs a physical phone): `bun run scripts/live.ts --to <phone>`
- Spec: `~/KpihX-Labs/Proxies/whats-proxy/CONTRACT.md`
