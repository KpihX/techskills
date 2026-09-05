# 🔒 ts-proxy — Tailscale API Without the Dashboard (v1.2.2)

> **Machine:** KpihX-Ubuntu · **Install:** `uv tool install ts-proxy` (PyPI) · **Auth:** Tailscale API token
> **Repos:** [GitHub](https://github.com/kpihx-labs/ts-proxy) · [GitLab](https://gitlab.com/kpihx-labs/proxies/ts-proxy)

I manage a Tailscale tailnet — homelab nodes, phones, ephemeral dev machines — and every routine op (approve a device, rotate a key, check DNS) meant opening the admin console in a browser. For an agent, that console doesn't even exist. `ts-proxy` puts the whole Tailscale API behind 44 flat actions with JSON schemas printed right in the terminal.

## 🧩 1. Action map (44 actions)

| Domain | Actions |
|--------|---------|
| Devices | `list-devices` · `get-device` · `authorize-device` · `delete-device` · `expire-device` · `rename-device` · `update-device` · `set-device-key-expiry` · `set-subnet-routes` |
| Auth keys | `list-authkeys` · `create-authkey` · `delete-authkey` |
| Users | `list-users` · `get-user` · `create-invitation` · `delete-invitation` · `list-invitations` · `get-invitation` · `suspend-user` · `restore-user` · `update-user-role` |
| ACL | `get-acl` · `update-acl` |
| DNS | `get-dns-nameservers` · `update-dns-nameservers` · `get-dns-preferences` · `update-dns-preferences` · `get-search-paths` · `update-search-paths` |
| Settings | `get-settings` · `update-settings` · `get-contacts` · `update-contacts` |
| Posture | `list-posture-checks` · `get-posture-check` · `create-posture-check` · `update-posture-check` · `delete-posture-check` |
| Webhooks | `list-webhooks` · `get-webhook` · `create-webhook` · `delete-webhook` · `rotate-webhook-secret` · `test-webhook` |

Every action prints its own JSON SCHEMA in `do --help` — payloads are validated before anything hits the network.

## 🧩 2. Session flow

I log in once, then everything is read-first:

```bash
ts-proxy admin login     # interactive API login
ts-proxy admin status    # verifies API connectivity
ts-proxy do list-devices | jq '.[].hostname'
ts-proxy do get-device '{"device_id":"12345"}' | jq '.hostname'
```

That worked — until a stale device kept holding a route. Deleting network objects is destructive, so it goes through HITL with an explicit rationale in the payload:

```bash
ts-proxy do delete-device '{"payload":{"device_id":"12345"},"rationale":"stale node, replaced"}'
# → HITL approval, then removal
```

## 🧩 3. Design worth stealing

```
ts-proxy do get-device '{"device_id":"12345"}' ./device_query.json  ← payload OR file
                                    │
                        ┌───────────┴───────────┐
                        │ strict payload model  │  API evolution never breaks
                        │ (no CLI flag drift)   │  the CLI surface
                        └───────────────────────┘
```

All business logic is payload-driven (`{"payload":{…},"rationale":"…"}`); CLI flags never carry business data. New Tailscale API fields flow through the same envelope — no flag plumbing per endpoint. The other Python proxies inherited this shape from `ts-proxy`'s hard-won lesson: **flags rot, schemas compose**.

## 🔀 4. Before / after

```
BEFORE                              AFTER (ts-proxy)
──────                              ─────────────────
admin console in browser,           44 schema-validated actions,
click-ops per device,               HITL on destructive calls,
no agent access                     jq-able JSON everywhere
```

## 📚 References

- Full catalog: `ts-proxy do --help` · per-action docs + schemas: `ts-proxy do <action> --help`
- Spec: `~/Kpihx-Labs/Proxies/ts-proxy/CONTRACT.md`
