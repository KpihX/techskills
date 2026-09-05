# 💼 link-proxy — LinkedIn From the Shell (v0.1.6)

> **Machine:** KpihX-Ubuntu · **Install:** `bun link` in repo (Bun ≥ 1.1) · **Auth:** OAuth 2.0 (`openid profile email w_member_social`), 60-day tokens
> **Repos:** [GitHub](https://github.com/kpihx-labs/link-proxy) · [GitLab](https://gitlab.com/kpihx-labs/proxies/link-proxy)

Posting to LinkedIn from an agent meant one of two bad options: unofficial scraping (fragile, ToS-hostile) or an MCP server nobody could call from a script. `link-proxy` wraps LinkedIn's *official* OAuth surface — 10 flat actions, Bun + TypeScript + Zod — so the same catalog works for agents and for a cron job that publishes my build notes.

## 🧩 1. Action map (10 actions)

| Domain | Actions |
|--------|---------|
| Posts | `post-create` (text) · `post-create-article` (URL + Open Graph card) · `post-create-image` (3-step media pipeline) · `post-create-full` (multi-image/video + custom title/desc, unlimited media) |
| Social | `post-like` · `comment-create` · `comment-reply` (threaded) |
| Profile | `profile-get` (OIDC) · `profile-status` (auth + token summary) |
| Escape hatch | `raw` (unrestricted LinkedIn REST v2 gateway, always HITL) |

Every write requires HITL approval — posts, likes, comments, all of it. Nothing reaches my public feed on a stray keystroke.

## 🧩 2. Session flow

```bash
link-proxy admin auth login    # HITL web form: credentials → OAuth consent
link-proxy admin auth status   # token days remaining, member details
link-proxy do profile-get
link-proxy do post-create '{"text":"Shipped v0.1.6 from the shell.","visibility":"PUBLIC"}'
# → HITL, then published
```

That worked — until a token silently died on day 61. Non-Partner apps get **no refresh token** — 60 days, then re-auth, no exceptions. The lifecycle is now ritual:

```
Day 0   → link-proxy admin auth login
Day 55+ → link-proxy admin auth status     (watch days_left)
Day 60  → token invalid — re-run auth
```

Media uploads are pipelines, not single calls: `post-create-image` runs register → PUT → attach; `post-create-full` uploads each item sequentially through the 2-step register+PUT flow. The proxy hides the choreography; `raw` exposes it when LinkedIn changes something.

## 🔀 3. Before / after

```
BEFORE                              AFTER (link-proxy)
──────                              ───────────────────
scraping or MCP-host-only,          official API, shell-native,
token expiry surprises              60-day ritual + status probe,
media uploads by hand               pipelines hidden, raw hatch open
```

## 📚 References

- Full catalog: `link-proxy do --help` · per-action docs: `link-proxy do <action> --help`
- Token: `~/.config/link-proxy/token.json` (0600) · `admin auth logout` clears it (HITL)
- Spec: `~/KpihX-Labs/Proxies/link-proxy/CONTRACT.md`
