# Edge agent — visible profile + MCP (KπX)

## One-time setup (KπX)

1. Open a terminal, run: `edge-agent`
2. In the **new Edge window**, sign in once to sites you need (Gmail, Lebara, etc.)
3. Leave that window open while agents work (or restart with `edge-agent` before each session)

## Daily use

```bash
edge-agent          # start visible Edge (CDP port 9222)
edge-agent-status   # check CDP is up
```

Before agents use **browser-mcp-stdio** or **playwright-mcp-stdio** in Cursor: Edge must be running (`edge-agent-status` → OK).

## Profile path

`$HOME/.config/kpihx-edge-automation` — not your Default Edge profile (Chrome/Edge 136+ policy).

## Cursor MCP

Configured in `$HOME/.cursor/mcp.json`:

- `browser-mcp-stdio` — attach via CDP (semantic / Aria snapshots)
- `playwright-mcp-stdio` — attach via CDP (Playwright tools)

Reload Cursor MCP after config changes.

## Profiles — no mix-up (FAQ)

| Path | What it is |
|------|------------|
| `$HOME/.config/microsoft-edge/` | Your **normal** daily Edge |
| `$HOME/.config/kpihx-edge-automation/` | **Agent Edge only** (separate files) |

Inside `kpihx-edge-automation/` there is a subfolder named `Default/` — that is **not** your main profile; Chromium always uses a `Default` profile directory *inside* each user-data-dir.

Signing into your Microsoft account in the **agent** window only syncs data **into** `kpihx-edge-automation`. It does **not** modify `microsoft-edge/` on disk.

You may run **both** Edges at once (different PIDs). Only the window started with `edge-agent` uses CDP port 9222.

## Bitwarden extension (self-hosted) — web OK, popup fails

### Root cause (confirmed 2026-05-25 via Playwright + CDP)

Bitwarden extension **2026.4.x** calls:

`POST https://vault.kpihx-labs.com/identity/accounts/prelogin/password`

Vaultwarden **before 1.36.0** returns **404** on that route. Legacy `POST …/identity/accounts/prelogin` still returns 200. The extension then shows the generic red banner *"An unexpected error has occurred"* (even with a wrong password — not a master-password typo).

**Fix (server):** upgrade Vaultwarden on docker-host to **`vaultwarden/server:1.36.0`** or newer (PR [#7156](https://github.com/dani-garcia/vaultwarden/pull/7156)). **Applied 2026-05-25** via Portainer stack `vaultwarden` (re-pull + update); `bw-prelogin-check.sh` → HTTP 200.

**Verify after upgrade:**

```bash
curl -sf -X POST "https://vault.kpihx-labs.com/identity/accounts/prelogin/password" \
  -H "Content-Type: application/json" \
  -d '{"email":"YOUR_EMAIL"}' | head -c 200
# must NOT be HTTP 404
```

**Not a profile mix-up:** daily Edge may still work because the extension keeps an **existing session** and skips the new prelogin/password path on fresh login.

**Known on KπX:** Vaultwarden web vault works in the same Edge window; only the **browser extension** shows *"An unexpected error has occurred"* after master password. Daily Edge (`microsoft-edge`) extension is usually already unlocked; **agent profile** is a fresh extension install.

**Prior KB:** `$HOME/Work/KpihX-Labs/Homelab/presentation/tutos_live/6-souverainete-secrets-certification-dns01.md` (HTTPS prefix + HSTS). No separate edge-agent Bitwarden fix before 2026-05-25.

### Server URL (exact)

`https://vault.kpihx-labs.com` — no trailing `/#/login`, no typo hostnames.

### Fix ladder (try in order)

1. **Custom environment (explicit endpoints)** — In extension → gear → *Self-hosted environment* → expand **Custom environment** and set:
   - **API:** `https://vault.kpihx-labs.com/api`
   - **Identity:** `https://vault.kpihx-labs.com/identity`
   - Leave base as `https://vault.kpihx-labs.com` → **Save** → log in again.
2. **2FA** — If the account uses TOTP, the web vault asks after master password; the extension popup may fail with a generic error if the 2FA step does not appear. After password, wait for a code field or check **Settings → Two-step login** on the web vault.
3. **AdGuard** — Agent profile often has AdGuard (`bgnkhhnn…`). Pause protection or allowlist `vault.kpihx-labs.com` and `*.kpihx-labs.com`, then retry.
4. **HSTS / cache (agent Edge only)** — `edge://net-internals/#hsts` → delete `vault.kpihx-labs.com` → restart Edge agent.
5. **Reset extension storage** — `edge://extensions` → Bitwarden → **Clear storage** → reconfigure self-hosted URL → login.
6. **Desktop bridge (workaround)** — Unlock **Bitwarden Desktop** (same server URL). Extension → Settings → enable integration with desktop app → unlock via desktop instead of typing master password in the popup.
7. **Debug** — `edge://extensions` → Bitwarden → **Service worker** → Inspect → Console, retry login; look for `prelogin`, `KDF`, `401`, `NetworkError`, `validateKdf`.

### API sanity (CLI)

```bash
curl -sf https://vault.kpihx-labs.com/api/config | head -c 200
curl -sf -X POST https://vault.kpihx-labs.com/identity/accounts/prelogin \
  -H 'Content-Type: application/json' \
  -d '{"email":"YOUR_EMAIL"}'
```

Prelogin must return `kdf` / `kdfIterations` (not empty).

## Browser MCPs on Edge agent (CDP :9222)

Both attach to the **same** Edge window (`edge-agent`). They do **not** replace each other — pick by task.

| MCP | Package | Used for (typical) | Strengths | Limits |
|-----|---------|-------------------|-----------|--------|
| **playwright-mcp-stdio** | `@playwright/mcp` | Portainer stack edit, Bitwarden popup, `run_code`, resize | Full Playwright API, snapshots, network/console, tab resize | `browser_tabs` list often shows **only the attached tab** |
| **browser-mcp-stdio** | `@agent-infra/mcp-server-browser` | Indexed clicks, `browser_tab_list` / switch, favorites page | Fast clickable index list, tab list/switch, markdown extract | Some `edge://` URLs fail (`edge://tab-groups/`) |

**cursor-ide-browser** (Cursor built-in) = **separate** Chromium in the IDE — not your Edge agent profile.

**Viewport:** If the site looks like a narrow strip on the left, an agent may have called `browser_resize` with a small width (e.g. 420px for extension popups). Fix: `browser_resize` **1920×1080** (Playwright MCP) or maximize the Edge window / F11.

**See all targets:** `curl -sf http://127.0.0.1:9222/json/list` (pages, iframes, Copilot side panel, etc.) — more complete than either MCP tab list.

## vs cursor-ide-browser

| | Edge agent | Cursor built-in browser |
|--|------------|-------------------------|
| Profile | Your automation Edge (cookies persist) | Isolated Cursor Chromium |
| Visibility | Full Edge window on desktop | Panel in IDE |
| Best for | Logged-in portals, HITL demos | Fast chat flows (e.g. support chat) |

## Edge Workspaces, tab groups, favorites, Copilot (live test runbook)

Validated on Edge **148** via Playwright MCP (`browser_run_code_unsafe`). Use this section to replay UI automation without rediscovering selectors.

### URLs (internal chrome)

| Surface | URL | Notes |
|---------|-----|--------|
| Workspaces hub | `edge://spaceworks/` | Lists workspaces + **New workspace** |
| Tab inventory | `edge://tab-search.top-chrome/` | **Organize tabs**, **Manage workspaces**, expand **Open Tabs** |
| Favorites manager | `edge://favorites/` | **Favorites bar** + **Other favorites** trees |
| Tab groups (invalid) | `edge://tab-groups/` | **ERR_INVALID_URL** on this build — use Tab Search UI instead |
| Copilot web | `https://copilot.microsoft.com/chats/<id>` or `edge://copilot-ntp/` | Cookie banner blocks clicks until dismissed |

### Workspace inventory procedure

1. `edge-agent` running; attach Playwright MCP to `:9222`.
2. `goto edge://spaceworks/` — read workspace rows (e.g. **KpihX-Labs · 14 tabs**, **X · 37 tabs**).
3. Click a workspace row → wait ~3s (restores that workspace’s tab set).
4. `goto edge://tab-search.top-chrome/` → click **Open Tabs** if collapsed (`aria-expanded` ≠ `true`).
5. Scrape `[role="listitem"]` **`aria-label`** strings.

**Aria-label grammar (Tab Search):**

- **Grouped:** `Title, GroupName, domain, <time> ago` (4+ comma segments; middle segment is not a domain).
- **Ungrouped:** `Title, domain, <time> ago` (3 segments).

**Parser sketch (Playwright):** split on `, `; if segment[1] matches `/ago|il y a|minute|hour/i` → ungrouped; else if ≥4 parts and segment[1] is not time-like → `group = segment[1]`.

**Caveat:** Switching workspace does not always isolate Tab Search rows in automation (many tabs from other workspaces can remain in the CDP page list). Treat spaceworks tab counts as authoritative; use Tab Search for group names and titles, not strict per-workspace isolation unless you close other workspaces’ tabs first.

**Example snapshot (current session, Tab Search — mixed workspaces):**

| Tab group | Tabs (title · domain) |
|-----------|------------------------|
| **Optim** (6) | Colab projection_simplexe · colab; Drive accès refusé · drive; Google sign-in · accounts; Adobe Acrobat · documentcloud; AI Studio · aistudio; Moodle APM · moodle |
| **Anal Fonc** (5) | AI Studio; chrome saved-tab-groups-unsupported; Drive; Windsurf; Moodle FMA |
| **ML** (2) | grader.dix login; Moodle CSC TD9 |
| **Revisions** (1) | AI Studio |
| **Ungrouped** (25+) | Gmail drafts, Zimbra, KpihX Labs, Example Domain, httpbin, TickTick, vscode.dev, Vaultwarden, edge:// pages, … |

### Sandbox workspace + tab group (UI, not CDP)

**CDP:** `TabGroups.create` via Playwright CDP session → **`'TabGroups.create' wasn't found`** on Edge 148. Use UI only.

**Sandbox tabs opened (automation):**

- `https://example.com/`
- `https://httpbin.org/get`
- `https://www.wikipedia.org/` (optional third tab outside group)

**Workspace:** click **New workspace** on `edge://spaceworks/`, type a name (e.g. `Agent-Sandbox-2026`) + Enter.

**Group (manual / semi-automated):**

1. Open **Tab Search** → **Organize tabs**.
2. Multi-select tabs (vertical tabs UI or Tab Search checkboxes when available).
3. **Group** / name group (e.g. `KπX Lab Pair`) — verify in Tab Search: grouped labels include the custom name.

**Ungrouped tab:** leave one tab (e.g. Wikipedia) outside the group; confirm aria-label has only `Title, domain, ago`.

### Favorites folder + bookmark

1. `goto edge://favorites/`.
2. Under **Favorites bar** or **Other favorites**, use **Add folder** (context menu or toolbar — labels vary by locale/build).
3. Name folder e.g. `agent-sandbox-bookmarks` (prior test used folder name **`fake`** under Favorites bar).
4. With target tab active (e.g. Example Domain), **Add page** / drag URL into folder, or bookmark star → choose folder.

**Automation note:** `edge://favorites/` body text is often empty in Playwright; rely on **`browser_snapshot`** tree (`heading "Favorites"`, `group` nodes named `fake`, `KpihX-Labs`, etc.).

### Copilot multi-turn chat

1. Navigate to `https://copilot.microsoft.com/` or existing chat URL.
2. **Cookie gate:** dismiss `data-testid="cookie-banner-accept-button"` via **`element.evaluate(el => el.click())`** — normal Playwright click fails (`z-40` overlay intercepts).
3. Composer: `#userInput` / `data-testid="composer-input"` / role `textbox` name **Message Copilot**.
4. Submit: **Enter** on textarea (more reliable than **Submit** button when overlays exist).
5. Wait **15–20s** per turn for streaming; scrape `body.innerText` or conversation nodes (not always `article`).

**Test messages used:** `Agent live test 1` … `Agent live test 5` (five user turns). Capture assistant text from page text after each wait.

### Live test pass (2026-05-25, Playwright MCP)

| Task | Result |
|------|--------|
| **Inventory** | `edge://spaceworks/` → **KpihX-Labs · 14 tabs**, **X · 41 tabs** (was 37 before extra test tabs). Tab Search after workspace click: **75** `listitem` rows — **not workspace-isolated** (same groups in both passes). |
| **Groups (Tab Search)** | **Optim**, **Anal Fonc**, **ML**, **Revisions** (+ misparsed **born out of the terminal. · GitHub** from long Warp title). Ungrouped includes Gmail, Zimbra, TickTick, Example Domain, httpbin, Wikipedia, KpihX Labs, edge:// pages, Vaultwarden, etc. |
| **Sandbox workspace** | **New workspace** `Agent-Sandbox-2026` attempted via **New workspace** button — **not listed** on spaceworks afterward (likely name dialog missed or merged into **X** tab count). |
| **Sandbox tabs** | Opened `example.com`, `httpbin.org/get`, `wikipedia.org` (duplicate instances in CDP). |
| **Tab group UI** | **Organize tabs** menu opens; **KπX Lab Pair** rename via Ctrl+click + group button — **not verified** in aria-labels (example/httpbin still ungrouped in scrape). |
| **Favorites** | `edge://favorites/` — **Add folder** via `getByTitle('Add folder')` works; folder **`fake`** already present from earlier run; new folder **`agent-sandbox-bookmarks`** not confirmed in tree text (shadow DOM — use snapshot refs). |
| **Copilot** | `https://copilot.microsoft.com/chats/9nAfSxjKCnmFe1a3hTSdw` exists; **5 user messages** attempted — landing-page overlay blocks normal `fill` on `#userInput`; use **cookie `evaluate` click** + **Enter**; assistant text capture needs signed-in chat + longer wait or frame-aware scrape. |

### Playwright MCP quirks (this profile)

| Issue | Mitigation |
|-------|------------|
| `browser_click` param | Use **`target`** ref from snapshot, not `ref` |
| `context.pages()` explosion | Workspace switch + many `newPage()` calls → dozens of tabs; prefer reuse one page + `goto` |
| Sensitive URLs in logs | Never paste full `accounts.google.com` OAuth URLs or tokens in docs/chat |
| Narrow viewport | Run `browser_resize` 1920×1080 before portal work |

### Cleanup pass (2026-05-25, manual MCP — no scripts)

**Method:** `browser_tabs` **close** one index at a time (highest first); verify after each batch with **list** + `edge://spaceworks/`.

| Check | Result |
|-------|--------|
| Test URLs removed | No `example.com`, `httpbin`, `wikipedia`, `copilot.microsoft`, `edge://tab-search`, `edge://copilot-ntp` in tab list |
| Workspaces | **KpihX-Labs · 14 tabs**, **X · 37 tabs**; no sandbox workspace |
| Utility tabs closed | `edge://favorites/`, `edge://spaceworks/` closed when opened only for cleanup |
| Academic / mail preserved | Gmail, Moodle, Colab, Zimbra, TickTick, AI Studio, OAuth choosers — **not** closed |
| Favorites debris | ✅ **`fake`** + **`agent-sandbox-bookmarks`** removed by KπX on `edge://favorites/` (Playwright Delete menu not in a11y tree) |

**Production rule:** See **`k-browser/SKILL.md` → Leave no trace** — always run this checklist after authorized tests.

## Files

- Launcher: `$HOME/.agents/skills/k-browser/scripts/edge-agent.sh`
- Shell: `$HOME/.agents/skills/k-browser/assets/edge-agent.ksh`
- Vision: `$HOME/.agents/skills/k-context/references/edge-agent-automation-vision.md`
