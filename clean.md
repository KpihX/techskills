# 🧹 clean — The Unified Cleanup Suite

> **Machine:** KpihX-Ubuntu (Ubuntu 25.10)
> **Lived on:** 2026-03 · **Status:** Production-stable
> **Source:** `~/Work/sh/clean/`

---

Ubuntu accumulates debris silently. Snap keeps old revisions you'll never boot
into. `uv` and `npm` cache packages from months-old installs. Docker leaves
dangling images after every failed build. HuggingFace fills gigabytes with
model weights you downloaded once, tried, and forgot.

None of this breaks anything — until your disk hits 95% and you start hunting
the cause. By then you're guessing: `du -sh *` on every subdirectory, deleting
things you're not sure about.

The cleanup suite solves this with a **modular, safe-by-default** approach.
One orchestrator script, five specialized modules, four flag levels from
read-only analysis to deep purge.

---

## 🏗️ Architecture

```
clean_all.sh  ←  orchestrator (entry point)
    │
    ├── clean_snap.sh    ←  Snap revisions & caches
    ├── clean_cache.sh   ←  uv, npm, pnpm, browsers, trash
    ├── clean_work.sh    ←  ~/Work/ project artifacts
    ├── clean_docker.sh  ←  images, volumes, networks
    └── clean_ai.sh      ←  Ollama models, HuggingFace cache
```

All modules share the same flag API — you can call `clean_all.sh` to run
everything, or a single module for targeted cleanup.

---

## 🚦 The Four Modes

Every script accepts the same four flags, applied in increasing order of impact:

```
--infos / -i    Read-only. Show sizes, stats, what would be affected.
                Never touches anything. Always safe to run.

--safe / -s     Remove clearly dead weight: disabled snap revisions,
                trash, temp files. Low risk, reversible in most cases.

--purge / -p    Deep clean: caches, old images, unused Docker volumes,
                HuggingFace downloads. Frees the most space. Confirm prompts
                before each destructive step.

--system / -y   Requires sudo. APT cache, systemd journal, root npm,
                system-level snap snapshots. Needs explicit elevation.
```

```
Risk level:
  --infos ─────────────────────────────── 0 risk (read-only)
  --safe  ───────────────────────────── low (reversible)
  --purge ─────────────────────────── medium (confirm prompts)
  --system ──────────────────────── high (needs sudo, system-wide)
```

---

## 📦 Module Breakdown

### clean_snap — Snap Revisions & Caches

Snap keeps every installed revision on disk by default — even disabled ones.
On a machine that's been running for months, this adds up fast.

```bash
./clean_snap.sh --infos    # Show revision count, disabled sizes, ~/snap caches
./clean_snap.sh --safe     # Remove disabled revisions (snap set system refresh.retain=2)
./clean_snap.sh --purge    # Also purge ~/snap/*/common/Cache* for specific apps
./clean_snap.sh --system   # Delete all Snap snapshots (sudo snap saved --delete)
```

### clean_cache — Package & Browser Caches

Covers the most common disk hogs: `uv` pip caches, `npm`/`pnpm` global caches,
browser data (Edge code cache, static files), trash, and diagnostic logs.

```bash
./clean_cache.sh --infos   # Show all cache sizes (uv, npm, pnpm, HuggingFace, browsers)
./clean_cache.sh --safe    # Empty trash, Edge static, home diagnostic logs
./clean_cache.sh --purge   # uv pip cache, npm global cache, pnpm store, HF cache, Edge Code Cache
./clean_cache.sh --system  # APT cache, systemd journal (sudo), root npm cache
```

### clean_docker — Images, Volumes, Networks

Docker's `system prune` is blunt. These modules are surgical — dangling images
first (safe), unused everything second (purge).

```bash
./clean_docker.sh --infos   # Docker system df breakdown
./clean_docker.sh --safe    # docker image prune, docker container prune
./clean_docker.sh --purge   # docker system prune --volumes (unused images + volumes)
```

### clean_ai — Ollama & HuggingFace

AI caches are the fastest-growing disk consumers. Ollama keeps models in
`/usr/share/ollama`, HuggingFace in `~/.cache/huggingface/`.

```bash
./clean_ai.sh --infos   # Show Ollama model list + sizes, HuggingFace cache breakdown
./clean_ai.sh --safe    # Remove HuggingFace temp logs and incomplete downloads
./clean_ai.sh --purge   # Remove phantom Ollama models + full HuggingFace cache
```

### clean_work — ~/Work/ Artifacts

Project debris: `__pycache__`, `.pytest_cache`, `node_modules`, `.venv` in
abandoned projects. This module is project-aware and scoped to `~/Work/`.

```bash
./clean_work.sh --infos   # Show artifact sizes by project
./clean_work.sh --safe    # Remove Python caches (__pycache__, .pytest_cache)
./clean_work.sh --purge   # Also remove node_modules in inactive projects
```

---

## 🎯 Typical Workflow

**Weekly check — see what's piling up:**

```bash
~/Work/sh/clean/clean_all.sh --infos
```

**Monthly safe pass — remove obvious dead weight:**

```bash
~/Work/sh/clean/clean_all.sh --safe
```

**Quarterly deep clean — free serious space:**

```bash
~/Work/sh/clean/clean_all.sh --purge
~/Work/sh/clean/clean_all.sh --system   # then again with sudo for system-level
```

**Targeted: just Docker after a heavy build session:**

```bash
~/Work/sh/clean/clean_docker.sh --purge
```

---

## ⏰ Automation

The suite runs automatically every **Saturday at 10:00 AM** via systemd:

```bash
# Check the timer
systemctl status clean-kpihx.timer

# View recent logs
cat ~/Work/sh/clean/logs/clean_$(date +%Y-%m-%d).log
```

The automated run uses `--safe` — conservative by default, no destructive
operations without manual trigger. The `--purge` and `--system` modes are
always manual, always prompted.

---

## 🚀 Quick Reference

```bash
# Full analysis (safe to run anytime)
~/Work/sh/clean/clean_all.sh --infos

# Safe weekly cleanup (all modules)
~/Work/sh/clean/clean_all.sh --safe

# Deep purge (prompted)
~/Work/sh/clean/clean_all.sh --purge

# System-level (requires sudo)
sudo ~/Work/sh/clean/clean_all.sh --system

# Single module
~/Work/sh/clean/clean_docker.sh --purge
~/Work/sh/clean/clean_ai.sh --infos

# View logs
ls ~/Work/sh/clean/logs/
```
