# 🌐 browser-proxy — A Visible Edge You Drive From the Shell (v0.8.1)

> **Machine:** KpihX-Ubuntu · **Install:** `uv tool install browser-proxy` · **Auth:** local daemon + paired Edge extension
> **Repos:** [GitHub](https://github.com/kpihx-labs/browser-proxy) · [GitLab](https://gitlab.com/kpihx-labs/proxies/browser-proxy)

Headless browser automation kept betraying me: scripts clicked things I couldn't see, sessions died invisibly, and debugging meant re-running blind. `browser-proxy` takes the opposite stance — **one daemon, one real visible Edge window per profile, no headless mode exists**. I watch everything the agent does, always. (An earlier headless design was scrapped as a 100% Transparency violation.)

## 🧩 1. The stack (three pieces, one CLI)

```
browser-proxy (CLI)
      │
      ├── daemon (on-demand, local-first)
      │     └── systemd Edge units — browser-proxy-edge@<profile>.service
      │           ONE real visible window per profile, private CDP endpoint
      └── extension bridge (KπX-owned Edge extension, paired per profile)
            tab groups · bookmarks · overlays · captchas · comboboxes
```

Admin manages the lifecycle; `do` drives pages:

```bash
browser-proxy admin status     # services, files, symlinks, token, permissions
browser-proxy admin profile    # systemd-templated profile instances
browser-proxy admin extension  # extension pairing
browser-proxy do profile-list '{}'
browser-proxy do profile-start '{"profile":"research"}'
```

## 🧩 2. Action map (71 actions — the largest catalog)

| Domain | Actions |
|--------|---------|
| Profiles | `profile-list` · `profile-start` · `profile-remove` (trash, never permanent delete) |
| Windows | `window-list` · `window-create` · `window-close` · `window-sync` · `window-save` · `window-restore` · `window-saved-list` · `window-saved-remove` |
| Tabs | `tab-list` · `tab-get` · `tab-create` · `tab-close` · `tab-activate` · `tab-update` |
| Groups | `group-list` · `group-create` · `group-update` · `group-move` · `group-add-tabs` · `group-remove-tabs` |
| Bookmarks | `bookmark-list` · `bookmark-get` · `bookmark-create` · `bookmark-remove` · `bookmark-update` |
| Extensions | `extension-list` · `extension-get` · `extension-enable` · `extension-disable` · `extension-reload` · `extension-search` |
| Page control | `page-navigate` · `page-reload` · `page-back` · `page-forward` · `page-click` · `page-click-eval` · `page-click-coordinates` · `page-hover` · `page-press` · `page-type` · `page-fill-form` · `page-select-option` · `page-scroll` · `page-evaluate` · `page-snapshot` · `page-screenshot` · `page-query` · `page-console-list` · `page-network-list` · `page-dialog-policy` |
| Cookies / Storage | `cookie-list` · `cookie-set` · `cookie-remove` · `storage-local-get` · `storage-local-set` · `storage-local-remove` · `storage-local-clear` |
| Human layer | `browser-ask-user` · `browser-dismiss-overlays` · `browser-solve-captcha` · `browser-set-date` · `browser-set-combobox` · `browser-drop-file` · `browser-get-new-tab` · `clipboard-read` · `clipboard-write` |
| Escape hatch | `raw` (raw CDP: `Target.getTargets`, …) |

## 🧩 3. A real session

I needed to research across tabs without losing the layout. Windows save and restore as named, batched snapshots:

```bash
browser-proxy do window-create '{"profile":"research","url":"https://example.com"}'
browser-proxy do page-navigate '{"profile":"research","target_id":"T1","url":"https://example.com/docs"}'
browser-proxy do page-snapshot '{"profile":"research","target_id":"T1"}'
browser-proxy do window-save '{"profile":"research","name":"docs-session"}'
# …later, on a fresh boot:
browser-proxy do window-restore '{"profile":"research","name":"docs-session"}'
```

That worked — until a cookie banner ate every click. Now `browser-dismiss-overlays` runs before any click chain, and `browser-ask-user` hands control back to me mid-flow instead of the agent guessing through a captcha (`browser-solve-captcha` covers the easy ones).

## 🔀 4. Before / after

```
BEFORE (headless MCPs)                    AFTER (browser-proxy)
──────────────────────                    ─────────────────────
invisible clicks, blind reruns,           visible window always,
per-agent browser processes,              one daemon + saved layouts,
sessions die silently                     extension bridge for what CDP can't
```

## 📚 References

- Full catalog: `browser-proxy do --help` · per-action payloads + 3 examples each: `browser-proxy do <action> --help`
- Spec: `~/KpihX-Labs/Proxies/browser-proxy/CONTRACT.md`
