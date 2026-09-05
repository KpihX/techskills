# 🔁 Proxies — Solo CLIs for Everything Agents Used to Do Over MCP

I kept hitting the same wall: my MCP servers (`tg-mcp`, `tick-mcp`, `whats-mcp`, `mail-mcp`) were great *inside* an MCP host — Claude Code, Codex, Gemini — and useless everywhere else. No shell. No scripts. No cron. No quick one-liner when I just wanted to check my inbox from a terminal.

So I rebuilt each MCP as a **proxy**: the same catalog, the same safety rails, but as a plain binary you call from any shell. No MCP runtime required. That worked — and then the proxies quietly became *better* than the MCPs they replaced: baked-in `--help` with examples on every action, human-in-the-loop approval on every write, read-back verification on every destructive call.

## 🧩 1. The shared ADN (learn once, drive all seven)

Every proxy is one binary with two namespaces. I learned this shape on `tg-proxy` and every later proxy copied it exactly:

```bash
<proxy> admin <command>            # setup | status | reset | purge (+ doctor, service…)
<proxy> do <action> '<json>'       # the RPC catalog
<proxy> do <action> --help         # full docs + examples, baked into the binary
```

That last line is the whole trick. There is no separate docs site to keep in sync — the binary *is* the docs:

```
You (or an agent)                    The proxy binary
      │                                    │
      │── <proxy> do --help ──────────────▶│  action catalog, one line each
      │── <proxy> do <action> --help ─────▶│  params + 3 worked examples
      │── <proxy> do <action> '{...}' ────▶│  → {"meta":{…},"data":{…}}
```

Every successful call returns the same envelope, so agents and scripts parse one shape forever:

```json
{"meta": {"status": "ok", "comment": "", "edited": false}, "data": {...}}
```

Options never carry business data — only presentation:

| Flag | Meaning |
|------|---------|
| `-f json\|table` | Output format (JSON default, table for humans) |
| `-o FILE` | Write the envelope to a file |
| payload as file | `do <action> ./payload.json` instead of inline JSON |

### Safety rails (identical everywhere)

```
read-only action          → runs immediately, returns JSON
        │
write action              → HITL web UI opens (600 s, fail-closed)
        │                   you review the exact payload, approve or edit
        ▼
destructive action        → HITL + preflight identity lock + read-back verification
                            result carries data.verification.ok — false + exit 1
                            on any silent partial write
```

Secrets live in `~/.config/<proxy>/.env` (0700 dir, 0600 files), injected per-process. Nothing secret ever touches the repo, the shell history, or the envelope.

## 📋 2. The seven proxies

| Proxy | Solo replaces | Actions | Install |
|-------|---------------|---------|---------|
| [🤖 tg-proxy](proxies/tg-proxy.md) | Telegram Bot API + Telethon user API | 24 | `uv tool install tg-proxy` |
| [✅ tick-proxy](proxies/tick-proxy.md) | TickTick V1 + V2 API | 52 | `uv tool install tick-proxy` |
| [🔒 ts-proxy](proxies/ts-proxy.md) | Tailscale API | 44 | `uv tool install ts-proxy` |
| [📧 mail-proxy](proxies/mail-proxy.md) | IMAP + SMTP (any provider) + Zimbra SOAP | 37 | `uv tool install mail-proxy` |
| [🌐 browser-proxy](proxies/browser-proxy.md) | Edge CDP + extension bridge | 71 | `uv tool install browser-proxy` |
| [💼 link-proxy](proxies/link-proxy.md) | LinkedIn REST v2 API | 10 | `bun link` (in repo) |
| [💬 whats-proxy](proxies/whats-proxy.md) | WhatsApp via Baileys | 68 | `bun link` (in repo) |

Python proxies ship on PyPI (`uv tool install <proxy>`); TypeScript proxies link locally (`bun link` in the repo, needs Bun ≥ 1.1).

## 🔀 3. Before / after — MCP vs proxy

```
BEFORE (MCP only)                          AFTER (proxy)
─────────────────                          ─────────────
agent ──MCP wire──▶ *-mcp server           agent ──shell──▶ *-proxy ──▶ API
   ▲                      │                   ▲                │
   │ usable ONLY inside   │                   │ usable from    │ same catalog
   │ an MCP host          │                   │ shell, script, │ + baked-in
   │                      │                   │ cron, ANY      │ --help
   │ no --help            │                   │ agent          │
   │ no HITL on writes    │                   │ every write: HITL
   │ silent API quirks    │                   │ quirks hardened
   │ (parentId ignored,   │                   │ (verified writes,
   │  groupId dropped)    │                   │  read-back proofs)
```

The proxies didn't just port the MCPs — they fixed them. TickTick's silently-ignored `parentId` became a verified `subtask-create` composite. Gmail's lying star became a documented IMAP-only verification caveat. Each fix is captured in the action's own `--help`, so the next agent inherits the lesson instead of rediscovering the bug.

## 📚 References

- Sources: [GitHub kpihx-labs](https://github.com/kpihx-labs) · [GitLab kpihx-labs/proxies](https://gitlab.com/kpihx-labs/proxies)
- Local checkouts: `~/KpihX-Labs/Proxies/<proxy>/` (`CONTRACT.md` = authoritative spec per proxy)
- This whole section was generated from live binaries: `<proxy> admin --help`, `<proxy> do --help`, `<proxy> do <action> --help` on every action.
