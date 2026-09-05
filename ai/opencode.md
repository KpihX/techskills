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

I learned this the hard way: I once tuned permissions directly in `~/.config/opencode/opencode.jsonc`, then a kernel sync overwrote the file and my evening of careful zones vanished. Now the ritual after any config change is fixed — verify before trusting:

```bash
opencode debug config     # does the running process see what I wrote?
opencode mcp list         # are the servers actually registered?
opencode providers list   # lanes intact?
```

That worked — until a hook I had clearly registered never fired. Which brings us to the plugin tree.

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

What the hooks actually do for me, day to day: `startup-context` injects my active TODOs and session log into every Colab/Build session, so the agent wakes up knowing what I was doing. `tmux-notify` arms a timer on any pane I send work to and reports back completion — no polling. `suggestions-check` enforces the ≥3 proactive suggestions footer. `raw-exec-guard` throws on raw `sudo`/`sleep` before the call even runs, rerouting me to `tmux send-keys`. They feel like guardrails until you work a week without them — then they feel like memory.

Voice deserves its own paragraph because it broke the cloud habit: `@kpihx/opencode-voice` (a sovereign fork, French siwis voice) does STT+TTS 100% locally through `whisper-cli` and `piper`. No audio leaves the machine. And when I'm truly stuck, `help_open` spawns a peer session for a second opinion — the technical escape route, paired with simply asking me via the `question` tool.

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

The `ollama` lane stays fresh by itself — a shell script re-syncs local models into `opencode.jsonc` with surgical sub-block replacement (other providers untouched). That worked — until I edited the sync script and the old bug kept reproducing. The culprit was my own running shell: it still held the *old function body in memory*. Fixing the file fixes nothing for already-open terminals. Now I run `kr` (reload `.kshrc` cleanly) in that exact terminal, or just open a fresh one. Stale-shell gotchas are the kind of bug you only believe after it bites you twice.

## 🧩 4. Permissions (Confiance Totale, 3 tiers)

```
global config → agent deltas → session runtime   (findLast wins)
🟢 read-only   🟡 low-risk   🟠 trusted   🔴 explicit ask
+ .env/*auth.json/*credentials.json always protected
+ raw sudo/sleep technically BLOCKED (raw-exec-guard throws — route via tmux send-keys)
```

The zones read as bureaucracy until the first time an agent proposes `rm -rf` on a directory you care about — then the explicit-ask tier earns its keep. Secrets follow the same paranoia: local MCPs read `{env:VAR}` placeholders resolved from the shell at startup, never hardcoded strings in JSONC. And live TUI permission tests always run with pane cwd in `/tmp`, so `tmp/…` permission patterns match and no `notify-*` test debris ever lands in a project workdir.

## 🧩 5. Web stack (3 services, not 2)

```
tailnet :2443 → shim :3000 → lens :30001 → web :40977   (day-to-day, voice UI injected)
tailnet :1443 → shim :4097 → web :40977                 (direct, clean mode)
```

`opencode-lens` is the entry point I actually use (CSP stripped, voice mic injected). The shim exists for a real upstream PTY bug (#5844) — middleware, not optional. History lives in SQLite (`~/.local/share/opencode/opencode.db`, WAL), not JSON.

On the phone the same care applies: a long model name once overlapped the Send button and made it untappable. The `web-model-line` plugin forces the model selector onto its own full-width line and lets the bottom bar grow instead of clipping — small fix, daily relief. I use Lens mode interactively and drop to Clean mode (direct backend, no injection) whenever I suspect a plugin of misbehaving. Two URLs, one backend, zero ambiguity about which layer is guilty.

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
