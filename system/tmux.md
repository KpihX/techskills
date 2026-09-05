# 🖥️ tmux — The Terminal That Never Dies (v3.6, prefix `Ctrl-a`)

> **Machine:** KpihX-Ubuntu · **Install:** `sudo apt install tmux` · **Config source of truth:** `k-tmux/assets/tmux.conf` (symlinked as `~/.tmux.conf`)
> **Skill:** `k-tmux` (v1.31.0) — sessions, sudo gate, stateful agent integration, full hub

I used to lose work three ways: closed terminals killing long builds, `sudo` prompts freezing agent sessions, and SSH drops nuking remote shells. tmux fixed all three at once — persistent server, named sessions per machine, and the *only* channel where interactive elevation is allowed to happen.

## 🧩 1. Mental model (three levels)

```
server
 └── session   (named workspace — one per equipment: ubuntu, pve, deep-pc6…)
      └── window   (tab: agents, default, tmp)
           └── pane   (split region: nvim | opencode | shell)
```

```bash
tmux ls                                            # list sessions
tmux new -ds <name>                                # new detached session (short names: pve-dns)
tmux new-window -t NAME -n tab                     # new tab in a session
tmux list-panes -a -F '#{session_name}:#{window_index}.#{pane_index} #{pane_current_command}'
tmux source-file ~/.tmux.conf                      # apply after config edits
```

Keys (prefix `Ctrl-a`, release, then key): `%` / `"` split, `h/j/k/l` or arrows navigate, `z` zoom, `[` copy-mode, `Ctrl+A r` kill-and-relaunch the pane process from shell history.

## 🧩 2. The sudo gate (agents read this twice)

Agents **never** run raw `sudo` — a hook technically blocks it. Elevation happens only in a shared pane:

```
agent shell (no sudo ever)          shared ops pane (ubuntu:agents.2)
      │                                        │
      │── tmux send-keys -t … 'sudo …' Enter ─▶│  KπX types the password
      │◀── <tmux-notify> finished ─────────────│  agent reads the result
```

`sui` is deprecated. Same gate covers SSH passphrases, vault unlocks, and any blocking prompt.

## 🧩 3. Run and get notified (never poll)

```bash
TMUX_TIMEOUT=30 TMUX_LINES=50 tmux send-keys -t SESSION:WIN.PANE 'long build' Enter
# → <tmux-armed> immediately; <tmux-notify> when the pane returns to its prompt
# → <tmux-watchdog> with an A/B/C/D decision tree if it runs long
```

I burned evenings `sleep`-polling panes before this existed. Now: arm with a timeout, do something else, the notification arrives. Short timeout (20 s) for anything that might prompt — catch the password request fast. Never `capture-pane` in a loop; one diagnostic capture only if the watcher looks suspect.

## 🧩 4. Survival kit (my settings)

- `mouse on`, wheel → copy-mode scroll, drag-to-copy, `history-limit 50000`
- `tmux-resurrect` + `tmux-continuum`: auto-save every 5 min, manual restore after reboot (boot auto-restore is OFF — run the recovery workflow)
- `focus-events on` (editor integration), `Ctrl+A r` universal "kill it and rerun"

That worked — until a reboot ate a 3-day session. Now resurrect saves continuously and I verify restore *before* trusting a layout.

## 🔀 5. Before / after

```
BEFORE                                    AFTER (tmux 3.6)
──────                                    ─────────────────
closed terminal = dead build,             persistent sessions per machine,
frozen sudo in agents,                    sudo gate + auto-notify,
SSH drop = lost shell                     reboot recovery, mouse + 50k scrollback
```

## 📚 References

- Skill hub: `k-tmux/SKILL.md` + `references/` (session-layout, cli-reference, copy-mode-mouse, stateful-integration, boot-recovery)
- Always identify the pane (`list-panes` + `pane_current_command`) before any `send-keys`
