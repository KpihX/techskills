# ✅ tick-mcp — TickTick MCP Server

> 71 MCP tools for full TickTick control from any agent — tasks, projects, habits, focus, queries, and intent-first views.

I was constantly context-switching between Claude and the TickTick web app — asking Claude to help me organize tasks, then manually executing the changes in the browser. `tick-mcp` collapses that gap: the agent can read, create, move, organize, and query my entire TickTick workspace directly, with no copy-paste in between.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Agent (Claude)                            │
└──────────────────────────────┬──────────────────────────────────┘
                               │  71 MCP tools
                    ┌──────────┴──────────┐
                    │                     │
               stdio (local)        HTTP (homelab)
                    │                     │
          ~/.local/bin/tick-mcp   tick.kpihx-labs.com/mcp
                    │                     │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │      tick-mcp       │
                    │                     │
                    │  V1 (official API)  │  oauth2 tokens
                    │  V2 (unofficial)    │  from bw-env
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │   TickTick API      │
                    │  api.ticktick.com   │
                    └─────────────────────┘
```

Two transports, same tools:
- **stdio** — local binary, launch by Claude Code directly
- **streamable-HTTP** — homelab deployment behind Traefik, reachable from any agent (Claude.ai, etc.)

---

## 📦 Install (local)

```bash
cd ~/Work/AI/MCPs/tick_mcp
uv tool install --editable .
tick-mcp serve    # MCP server (stdio)
tick-admin        # Rich+Typer admin CLI
```

---

## 🔌 Claude Code config

Two entries — homelab HTTP (primary) with local stdio fallback:

```json
// In ~/.claude.json → projects["/home/kpihx"]["mcpServers"]
"tick-mcp": {
  "type": "http",
  "url": "https://tick.kpihx-labs.com/mcp"
},
"tick-mcp--fallback": {
  "command": "zsh",
  "args": ["-l", "-c", "/home/kpihx/.local/bin/tick-mcp serve"]
}
```

**Why the login shell wrapper on the fallback?** `tick-mcp` needs TickTick OAuth tokens from `bw-env`. Claude Code doesn't load a login shell, so env vars from `~/.kshrc` are invisible. `zsh -l -c` sources `.zprofile` → `.kshrc` → secrets. See [zsh_env.md](system/zsh_env.md).

---

## 🛠️ Tool categories

| Category | Examples |
|---|---|
| **Tasks** | `create_task`, `update_task`, `complete_task`, `batch_create_tasks` |
| **Projects** | `create_project`, `list_projects`, `get_project_detail` |
| **Intent views** | `tasks_of_today`, `week_agenda`, `priority_dashboard`, `overdue_tasks` |
| **Query/Search** | `query_tasks`, `query_projects`, `search_nodes`, `workspace_map` |
| **Verified actions** | `create_subtask`, `verified_move_tasks`, `verified_set_subtask_parent` |
| **Habits** | `create_habit`, `habit_checkin`, `get_habit_records` |
| **Focus** | `get_focus_stats`, `get_productivity_stats` |
| **Kanban** | `list_columns`, `manage_columns` |
| **Tags** | `create_tag`, `merge_tags`, `list_tags` |
| **Utilities** | `ticktick_guide`, `build_recurrence_rule`, `full_sync` |

### Start here: `ticktick_guide`

Always call `ticktick_guide` first in a new session — it orients the agent on the current workspace structure and recommends the right tool flow for your intent.

---

## ⚠️ Critical API gotchas

These are silent bugs in the TickTick API that trip up agents:

```
parentId silently ignored at creation (V1 + V2 batch)
→ Always use: create_task → set_subtask_parent

groupId silently ignored by V1 project create/update
→ Always follow up with V2 batch/project call

PATCH /habits/batch is a FULL REPLACEMENT (not partial)
→ Always read-modify-write before updating habits

move_tasks does NOT cascade to children
→ Fetch childIds first → move atomically in single batch
```

---

## 🗂️ Workspace structure (KpihX)

| Folder | Projects |
|---|---|
| 🎓 X | Notes, Todos, Agenda, Extra Scolaire |
| 💼 Careers | Todos, Notes, Agenda |
| 🏠 Persos | Courses, Proches, Administratif, Santé, Todos, Notes, Agenda |
| 🛠️ Tech & Dev | Notes, Prompts, Windows, Ubuntu, Todos, Projets, Agent OS |
| 🌊 Flow | Ideas, Tasks, Pense Bêtes, Research, À voir |
| 🖥️ Homelab | Todos, Ideas, Notes, Boost |

---

## 🗺️ Roadmap

- Saved query presets for recurring weekly views
- Intent-first dashboards (daily briefing, sprint planning)
- Telegram admin bridge (in progress)

**Repos:** [GitHub](https://github.com/KpihX/tick-mcp) · [GitLab](https://gitlab.com/kpihx-labs/tick-mcp)
