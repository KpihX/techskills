# 🤖 tg-proxy — Telegram From the Shell (v1.2.1)

> **Machine:** KpihX-Ubuntu · **Install:** `uv tool install tg-proxy` · **Auth:** Telethon user session + Bot API tokens
> **Repos:** [GitHub](https://github.com/kpihx-labs/tg-proxy) · [GitLab](https://gitlab.com/kpihx-labs/proxies/tg-proxy)

I was running two Telegram lives: bots managed through BotFather in a browser, and my own chats reachable only from my phone. Every recap, every notification forward, every bot token rotation meant context-switching between three apps. I wanted one shell command for all of it — bot API *and* my user account — so I built `tg-proxy` on two layers that stay independent: Bot API for bots, Telethon MTProto for me.

## 🧩 1. Two layers, one binary

```
tg-proxy
   ├── Bot API layer (needs a bot token)
   │     bot-list · bot-info · bot-token · bot-create · bot-delete
   │     bot-send · bot-send-file · bot-photo · updates
   │     webhook-get · webhook-set · webhook-del · raw
   └── User layer (my Telethon session, acts AS ME)
         chat-list · chat-read · chat-send · chat-send-file
         chat-download · chat-delete · chat-delete-messages
         folder-list · folder-set · folder-delete · chat-move
```

The split matters: `bot-send` delivers *as the bot* (great for channel-style notifications to myself), while `chat-send` delivers *as me* (real conversations with people, groups, channels). Mixing them up sends messages from the wrong identity — the action names keep the two apart.

## 🧩 2. Setup — one HITL web form, then never again

I ran `tg-proxy admin setup` and a local web form walked me through phone login (OTP once), then wrote `~/.config/tg-proxy/.env`. Since then:

```bash
tg-proxy admin status    # identity, masked token inventory, session health (JSON)
tg-proxy do chat-list    # proves the session is alive
```

That worked — until I needed a new bot token at 11pm. Instead of opening BotFather on my phone, I now run the whole `/newbot` flow from the shell:

```bash
tg-proxy do bot-create '{"bots":[{"name":"KpihX Alerts","username":"kpihx_alerts_bot"}]}'
# → HITL, then token appended to .env as KPIHX_ALERTS_BOT=<token>
```

⚠️ **BotFather hard rule, learned the painful way:** the username MUST end with `bot` or creation fails. The action tells you this in its own `--help` — I put it there after burning one evening on `username_invalid`.

## 🧩 3. Daily flows

**Morning recap (read-only, no approval needed):**

```bash
tg-proxy do chat-list -f table
tg-proxy do chat-read '{"chat":"@KpihX","limit":20}'
```

**Send as me (HITL — I review before anything leaves):**

```bash
tg-proxy do chat-send '{"to":"@KpihX","message":"Deploy done at 15:00"}'
# → {"meta":{"status":"ok",...},"data":{"message_id":50,"chat":"@KpihX"}}
```

**Organize at scale with folders:**

```bash
tg-proxy do folder-list
tg-proxy do folder-set '{"title":"Homelab","chats":["@KpihX"]}'
tg-proxy do chat-move '{"chat":"@KpihX","folder":"Homelab"}'
```

## 🔀 4. Before / after

```
BEFORE                                    AFTER (tg-proxy, 24 actions)
──────                                    ────────────────────────────
BotFather in browser  ─┐                  bot-create / bot-token / bot-delete
phone app for chats   ─┼─▶ 3 apps,        chat-send / chat-read (as me)
manual token copy     ─┘   copy-paste     bot-send (as bot), tokens auto-appended
                                          to .env, folders managed as data
```

## 📚 References

- Full catalog: `tg-proxy do --help` · per-action docs: `tg-proxy do <action> --help`
- Escape hatch: `tg-proxy do raw` (mtproto/botapi/bf gateway, HITL-gated)
- Spec: `~/KpihX-Labs/Proxies/tg-proxy/CONTRACT.md`
