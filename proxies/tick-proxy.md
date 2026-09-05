# ✅ tick-proxy — TickTick Without the Browser (v2.2.0)

> **Machine:** KpihX-Ubuntu · **Install:** `uv tool install tick-proxy` · **Auth:** TickTick V1 + V2, session token
> **Repos:** [GitHub](https://github.com/kpihx-labs/tick-proxy) · [GitLab](https://gitlab.com/kpihx-labs/proxies/tick-proxy)

I was constantly context-switching: asking an agent to organize my tasks, then manually replaying every change in the TickTick web app. `tick-mcp` (71 MCP tools) closed the gap inside MCP hosts — but the moment I left the agent, I was back to clicking. So I refactored the whole MCP into `tick-proxy`: 52 actions, same coverage, callable from anywhere.

That worked — until TickTick's API started lying to me. Twice. `parentId` is *silently ignored* at task creation. `groupId` is *silently ignored* at project creation. No error, just a task with no parent and a project in no folder. The proxy doesn't just document these traps — it routes around them with verified composite actions.

## 🧩 1. Action map (52 actions, 12 domains)

| Domain | Actions |
|--------|---------|
| Tasks | `task-create` · `task-update` · `task-complete` · `task-reopen` · `task-delete` · `task-info` · `task-list` · `project-tasks` · `inbox-list` |
| Batch | `task-batch-create` · `task-batch-update` · `task-batch-delete` · `task-move` · `task-parent-set` · `subtask-create` |
| Projects | `project-list` · `project-info` · `project-create` · `project-update` · `project-delete` |
| Folders / Columns | `folder-list` · `folder-manage` · `column-list` · `column-manage` |
| Tags | `tag-list` · `tag-create` · `tag-update` · `tag-merge` · `tag-delete` |
| Habits | `habit-list` · `habit-section-list` · `habit-create` · `habit-update` · `habit-delete` · `habit-checkin` · `habit-records` |
| Query | `workspace-map` · `query-projects` · `query-folders` · `query-tasks` · `query-agenda` |
| Views | `view-today` · `view-week` · `view-week-overview` · `view-upcoming` · `view-overdue` |
| History / Stats / Sync | `history-query` · `focus-stats` · `user-status` · `user-stats` · `sync-full` |
| Escape hatch | `raw` |

## 🧩 2. The two traps (and the composites that defuse them)

**Trap 1 — subtasks.** Passing `parentId` at creation does nothing. The safe path is the composite:

```bash
# ❌ silently creates a parentless task:
tick-proxy do task-create '{"title":"Won’t link"}'   # → exit 1: title_ops required (also: raw titles rejected!)

# ✅ one verified composite (create + link + read-back proof):
tick-proxy do subtask-create '{"parent_id":"<id>","title_ops":[{"op":"insert","insert_lines":[0],"insert_text":"Buy bread"}]}'
```

⚠️ Note the second surprise in that example: raw `title`/`content`/`desc` are *rejected at the CLI* — you must emit `title_ops` document operations, which the HITL review then shows as diffs. Agents write ops, humans review diffs. That design fell out of watching agents mangle task bodies.

**Trap 2 — moves.** The V2 API never cascades: moving a parent strands its children. `task-move` fetches `childIds` and moves them in the same batch, then reads everything back (`@require_verification` — a silent partial move becomes `verification.ok=false`, exit 1).

**Trap 3 — folders.** V1 returns `groupId: null` always and ignores it on create. `project-create` creates via V1, applies the folder via V2 `batch/project`, then reads the truth back. When you just need the real mapping: `workspace-map` or `sync-full`.

## 🧩 3. Session flow

```bash
tick-proxy admin setup              # HITL web form → ~/.config/tick-proxy/.env
tick-proxy admin status             # masked tokens, expiry, V1/V2 probes (JSON)
tick-proxy do view-today            # what needs me right now
tick-proxy do habit-checkin '{"habit_id":"<id>"}'
tick-proxy admin session-refresh    # V2 token expired? password collected transiently, never stored
```

## 🔀 4. Before / after

```
BEFORE (tick-mcp)                         AFTER (tick-proxy)
─────────────────                         ──────────────────
71 MCP tools, MCP host required            52 actions, any shell/script/cron
silent parentId / groupId drops            verified composites (subtask-create,
raw titles accepted, bodies mangled         task-move, project-create)
```

## 📚 References

- Full catalog: `tick-proxy do --help` · per-action docs: `tick-proxy do <action> --help`
- Spec (auth levels V1/V2, HITL matrix): `~/KpihX-Labs/Proxies/tick-proxy/CONTRACT.md`
