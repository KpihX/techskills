# Google Workspace CLI (`gws`)

> Terminal access to Gmail, Drive, Calendar, Sheets, Docs, and other Workspace APIs — Discovery-based, JSON output, agent-friendly.

Package: [`googleworkspace/cli`](https://github.com/googleworkspace/cli) · binary: **`gws`**

Skill hub: `$HOME/.agents/skills/k-google/SKILL.md`

---

## Install

```bash
npm install -g @googleworkspace/cli
gws --version
```

Uses the npm global prefix (`$HOME/.npm-global/bin/gws`).

---

## Architecture (content vs container)

| CLI | Role |
|-----|------|
| **`gws`** | Workspace **content** (mail, drive files via API, calendar, docs) |
| **`gcloud`** | GCP **container** (projects, VMs, buckets, IAM) — see Google Cloud docs |

Do not use `gcloud` for Gmail or personal Drive file workflows.

---

## Auth (one-time, user-owned)

```bash
gws auth setup
# or
gws auth login
```

- Requires a Google Cloud project + OAuth client (Desktop app).
- Store client id/secret in Vaultwarden; inject via `bw-env` when scripting.
- Agents must **not** complete OAuth interactively without KπX at the keyboard — same policy as `m365 login`.

Check status after KπX authenticates:

```bash
gws auth status
```

---

## Usage patterns

### Discovery-style (full API surface)

```bash
gws gmail users messages list --params '{"userId":"me","maxResults":5}'
gws drive files list --params '{"pageSize":10}'
gws calendar events list --params '{"calendarId":"primary","maxResults":5}'
```

### Helpers (when available)

```bash
gws gmail +send --help
gws drive +upload --help
gws sheets +read --help
```

Prefer read-only Discovery calls for agent triage until send/upload is explicitly approved.

---

## Scope limits (unverified apps)

Testing-mode OAuth apps may hit Google scope caps (~25 scopes). Prefer targeted login:

```bash
gws auth login --scopes drive,gmail,sheets
```

See upstream docs if consent screen blocks personal `@gmail.com` accounts.

---

## Related KπX tools

| Need | Tool |
|------|------|
| Mounted Drive trees (`$HOME/Gdrive*`) | `rclone` — `$HOME/Work/techskills/` + `k-rclone` |
| Folder map / matricule paths | `k-drive` |
| Poly Zimbra mail | mail-mcp — `k-mail` (not `gws`) |

---

## Changelog

| Date | Note |
|------|------|
| 2026-05-25 | Initial doc — `gws` 0.22.5 installed on KπX workstation |
