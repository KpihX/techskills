# 📧 mail-proxy — Every Inbox From One Shell (v0.8.0)

> **Machine:** KpihX-Ubuntu · **Install:** `uv tool install mail-proxy` · **Auth:** per-account IMAP/SMTP, OAuth2 or app passwords
> **Repos:** [GitHub](https://github.com/kpihx-labs/mail-proxy) · [GitLab](https://gitlab.com/kpihx-labs/proxies/mail-proxy)

I live across five mailboxes — Polytechnique, Hotmail, two Gmails, Zimbra — and my morning routine was five logins in a browser. `mail-mcp` (25 MCP tools) fixed it inside agents, then I refactored it into `mail-proxy` on the exact `tick-proxy` ADN — and the CLI grew past its parent: 37 actions, Zimbra SOAP tags, send-as aliases, bounce probes. This is now the transport behind the `k-mail` skill.

## 🧩 1. Action map (37 actions)

| Domain | Actions |
|--------|---------|
| Inbox | `inbox-check` · `inbox-digest` |
| Messages | `message-list` · `message-info` · `message-search` · `message-thread` · `message-mark` · `message-move` · `message-archive` · `message-trash` · `message-spam` · `message-delete` |
| Compose | `message-send` · `message-reply` · `message-forward` · `message-draft` |
| Folders | `folder-list` · `folder-create` · `folder-rename` · `folder-delete` |
| Attachments | `attachment-download` |
| Labels / Signatures | `label-list` · `label-set` · `label-delete` · `signature-list` · `signature-create` · `signature-update` · `signature-delete` · `signature-default` · `signature-get` |
| Zimbra | `zimbra-tag-list` · `zimbra-tag-items` · `zimbra-tag-create` · `zimbra-tag-delete` · `zimbra-tag-apply` · `zimbra-tag-remove` |
| Escape hatch | `raw` (dedicated IMAP connection) |

## 🧩 2. Session flow

```bash
mail-proxy admin auth login    # smart HITL form: type → email → password/OAuth2 (writes accounts.json + .env)
mail-proxy admin status        # accounts, auth, permissions, probes, issues (JSON)
mail-proxy do inbox-digest     # the whole morning in one call: unread + flagged + today
```

That worked — until I needed to answer *as* the right identity. Every send carries a mandatory `account_id`, and `from_address` overrides the From header for Gmail send-as aliases:

```bash
mail-proxy do message-send '{"to":["x@y.fr"],"subject":"Rendez-vous","body_text":"Dispo demain 15h ?","account_id":"poly"}'
# → {"meta":{"status":"ok",...},"data":{"smtp_accepted":true,"message_id":"<…>","saved_to_sent":true}}
```

## 🧩 3. Safety ladder (deletes are a staircase, not a cliff)

```
message-trash    recoverable delete (prefer this)      HITL + verified
message-spam     report + move to Spam/Junk            HITL + verified
message-delete   PERMANENT expunge                     HITL required
    │   UIDs pre-read (absent targets fail BEFORE approval)
    │   UIDs locked in the review
    └── deletion confirmed by polling until every UID is gone
```

And the Gmail-star lesson, baked into `message-mark --help` so nobody re-learns it: `\Flagged` is verified against IMAP (`UID FETCH FLAGS`), *not* the Gmail Web UI — the web page can keep showing a yellow star while IMAP says unstarred. `verification.ok:true` proves IMAP state only.

## 🔀 4. Before / after

```
BEFORE                              AFTER (mail-proxy)
──────                              ───────────────────
5 browser logins per morning,       inbox-digest across accounts,
no scriptable search,               server-side IMAP + client regex search,
send-as confusion                   mandatory account_id, bounce probes,
                                    Zimbra tags over SOAP
```

## 📚 References

- Full catalog: `mail-proxy do --help` · per-action docs: `mail-proxy do <action> --help`
- Spec: `~/KpihX-Labs/Proxies/mail-proxy/CONTRACT.md` · skill: `k-mail` (sole transport)
