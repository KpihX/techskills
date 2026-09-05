# 🤖 OpenCode — The Agent Runtime (v1.18.16)

> **Machine:** KpihX-Ubuntu · **Install:** `curl -fsSL https://opencode.ai/install | bash` (`~/.opencode/bin/`)
> **Skill:** `k-opencode` (v2.79.1) — config, plugins, providers, permissions, full hub

I used to drive AI agents through their own chat windows — one UI per provider, configs scattered, no shared memory between them. Then I flipped the model: one terminal-native agent runtime, one sovereign kernel (`~/.agents/`) symlinked into it, every provider behind one config file. OpenCode is that runtime — and everything below is the KπX wiring that makes five different agent brands share one brain.

## 🧩 1. One kernel, every agent

```
~/.agents/  (sovereign kernel, git-tracked)
   ├── AGENTS.md ──symlink──▶ ~/.config/opencode/AGENTS.md
   ├── agents/   ──symlink──▶ ~/.config/opencode/agents/   (Colab, Explore, Build, Lite…)
   ├── skills/   ──native──▶ scanned by OpenCode, no symlink needed
   └── skills/k-opencode/assets/opencode.jsonc ──symlink──▶ ~/.config/opencode/opencode.jsonc
```

I edit the asset, never the runtime copy. `opencode.jsonc` holds models, permissions, MCPs, and the plugin array — one file, whole fleet.

## 🧩 2. Full plugin tree (19 active)

```
opencode
├── lifecycle plugins ("plugin" array, file://{env:HOME} — $HOME/~ NEVER expand)
│   ├── hooks (9): raw-exec-guard · prompt-enforcer[·-strict/-lite] · stop-enforcer
│   │              math-enforcer · make-check-guard · git-noverify-guard
│   │              startup-context · suggestions-check · tmux-notify · opencode-notify
│   └── non-hooks (4): opencode-help (peer sessions) · opencode-ollama sync
│                      opencode-go-proxy (:4999 path-rewrite) · + npm ×6
├── TUI plugin: @kpihx/opencode-voice (sovereign fork, local STT/TTS — whisper-cli + piper)
├── slash commands: /context (auto-discovered .opencode/commands/*.md)
└── web UI (opencode-lens proxy injects): web-voice mic · web-model-line fix
```

That worked — until hooks silently failed to load. Lesson, now muscle memory: only `file://{env:HOME}/…` paths load; `$HOME`, `~`, and hardcoded `/home/<user>` fail *silently*. And registering a config entry ≠ runtime readiness — every binary the plugin shells out to (`sox`, `whisper-cli`, `tmux`) must exist per machine.

## 🧩 3. Providers (local-first, three lanes)

| Provider | Backend | Models |
|----------|---------|--------|
| `ollama` | local :11434 (auto-synced by script) | qwen, mistral, … |
| `ollama6` | deep-pc6 via tunnel :11436 | same, remote GPUs |
| `opencode-go-2` | 2nd subscription via :4999 proxy (fixes `modelsDiscovery` path-strip bug) | mimo, deepseek (34 discovered) |

```bash
opencode --version
opencode debug config
opencode mcp list
opencode providers list
opencode models
```

## 🧩 4. Permissions (Confiance Totale, 3 tiers)

```
global config → agent deltas → session runtime   (findLast wins)
🟢 read-only   🟡 low-risk   🟠 trusted   🔴 explicit ask
+ .env/*auth.json/*credentials.json always protected
+ raw sudo/sleep technically BLOCKED (raw-exec-guard throws — route via tmux send-keys)
```

## 🧩 5. Web stack (3 services, not 2)

```
tailnet :2443 → shim :3000 → lens :30001 → web :40977   (day-to-day, voice UI injected)
tailnet :1443 → shim :4097 → web :40977                 (direct, clean mode)
```

`opencode-lens` is the entry point I actually use (CSP stripped, voice mic injected). The shim exists for a real upstream PTY bug (#5844) — middleware, not optional. History lives in SQLite (`~/.local/share/opencode/opencode.db`, WAL), not JSON.

## 🔀 6. Before / after

```
BEFORE                                    AFTER (opencode + kernel)
──────                                    ─────────────────────────
one chat UI per provider,                 one runtime, one config,
configs scattered, agents amnesiac        shared skills/agents/memory,
                                          local models first, web anywhere
```

## 📚 References

- Skill hub: `k-opencode/SKILL.md` + `references/` (permissions, plugins-architecture, providers, voice, sqlite-storage, agent-system)
- Verify after any change: `opencode debug config` + a `/tmp`-cwd live TUI test
