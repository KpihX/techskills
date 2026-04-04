# 🧠 qmd - Local Semantic Search for Your Notes

> **Machine:** KpihX-Ubuntu | **Date:** 2026-03-24

I have a large amount of Markdown notes in `$HOME/Work/tutos_live` and `$HOME/Work/Homelab/presentation`. Using `grep` works, but it's limited to exact keyword matching. I wanted something smarter, capable of understanding the intent behind my searches. That's when I discovered `qmd`.

`qmd` is a local search engine that uses AI models to "understand" the meaning of your notes and provide much more relevant results. Here's how I installed and configured it to become an essential building block of my "external memory."

## Installation and Configuration

Installation is done via `bun` (my JS/TS package manager of choice).

```bash
bun install -g @tobilu/qmd
```

Once installed, the first step is to tell it which folders to index. Today the canonical source of truth is:

- `$HOME/.config/qmd/index.yml`

The live state can then be checked with:

```bash
qmd status
qmd collection list
```

Historically, I started from two primary knowledge sources:

1.  `$HOME/Work/tutos_live`
2.  `$HOME/Work/Homelab/presentation`

So I created two `qmd` "collections":

```bash
# Historical bootstrap
qmd collection add $HOME/Work/tutos_live/ -n tutos_live --mask "**/*.{md,yml,yaml,toml}"
qmd collection add $HOME/Work/Homelab/presentation/ -n presentation --mask "**/*.{md,yml,yaml,toml}"
```

The important point today is different: keep the index **hot-only**.

- prefer `md`, `yml`, `yaml`, `toml`
- avoid raw chat histories, large `json/jsonl` dumps, and volatile scripts by default
- inspect runtime histories directly from disk instead of forcing them into QMD

Finally, you need to initiate the semantic indexing. This command will download AI models (if it's the first time) and transform your documents into vectors.
```bash
qmd embed
```
Your knowledge base is now ready to be queried.

If embeddings seem CPU-bound, check the local Vulkan toolchain first:

```bash
which glslc
qmd status
```

On this machine, missing `glslc` was the blocker that prevented the local Vulkan backend from building correctly for `node-llama-cpp`.

## Usage: From Simple to Structured Search

Here's how I use `qmd` daily.

### Hybrid Search (`qmd query`)

This is the default command. It's perfect for most cases.

```bash
# Search how I secured Traefik
qmd query "traefik security middleware" -c presentation
```

### Structured Search (Expert Mode)

When I need more precision, I combine keyword search (`lex:`), conceptual search (`vec:`), and hypothetical document (`hyde:`).

```bash
# Problem: Find the guide that explains how to configure a systemd service for a Python script.
qmd query $'hyde: A tutorial that explains how to create a systemd service file to run a Python script as a background daemon on Ubuntu.' -c tutos_live
```

## Automation with Git Hooks

To ensure the `qmd` index stays fresh without blocking commits, I use a shared `post-commit` Git hook template. The current source of truth is:

- `$HOME/.agents/skills/k-qmd/references/post-commit-hook.sh`

Repositories that match an active QMD collection can symlink `.git/hooks/post-commit` directly to that file.

The current behavior is intentionally conservative:

- derive the collection from the repo root name unless overridden
- run `qmd update <collection>` synchronously
- default to `update-only`
- optional debounced background `qmd embed <collection>`
- skip cleanly if the repo has no active QMD collection

For important searches, I now prefer:

```bash
qmdr query "intel arc vulkan" -c tutos_live
```

`qmdr` is an explicit wrapper that:

1. runs `qmd status`
2. checks whether `Pending > 0`
3. runs `qmd embed` first if needed
4. only then executes `qmd query`, `qmd search`, or `qmd vsearch`

This preserves the lean post-commit hook model while still giving me a deterministic "fresh before search" entrypoint.

**File:** `.git/hooks/post-commit` (symlink or copy from the shared template)
```bash
#!/bin/sh
REPO_ROOT="$(git rev-parse --show-toplevel 2>/dev/null || pwd)"
REPO_NAME="$(basename "$REPO_ROOT")"
COLLECTION="${COLLECTION:-$REPO_NAME}"
PATTERN="${PATTERN:-\\.(md|yml|yaml|toml)$}"
QMD="${QMD:-$HOME/.npm-global/bin/qmd}"
EMBED_MODE="${EMBED_MODE:-update-only}"

echo "--- [QMD Hook] Running post-commit checks ---"

if ! command -v "$QMD" > /dev/null 2>&1 && ! command -v qmd > /dev/null 2>&1; then
    echo "--- [QMD Hook] 'qmd' not found. Skipping update."
    exit 0
fi

QMD_BIN="$QMD"
if ! command -v "$QMD_BIN" > /dev/null 2>&1; then
    QMD_BIN="qmd"
fi

if ! "$QMD_BIN" collection show "$COLLECTION" > /dev/null 2>&1; then
    echo "--- [QMD Hook] Collection '$COLLECTION' not found. Skipping."
    exit 0
fi

if git diff --name-only HEAD~1 HEAD 2>/dev/null | grep -qE "$PATTERN"; then
    echo "--- [QMD Hook] Relevant files modified. Updating QMD index..."
    if "$QMD_BIN" update "$COLLECTION"; then
        if [ "$EMBED_MODE" = "update-only" ]; then
            echo "--- [QMD Hook] Update-only mode. Skipping embed."
        fi
        echo "--- [QMD Hook] QMD index update complete."
    fi
else
    echo "--- [QMD Hook] No relevant files modified. Skipping update."
fi

exit 0
```
Remember to make it executable: `chmod +x .git/hooks/post-commit`.

One practical nuance: during the first local Vulkan rebuild, `qmd embed` can still print a temporary `no GPU acceleration` warning while `node-llama-cpp` rebuilds its native backend. The steady-state truth is `qmd status`, not that transient warning.

With this configuration, `qmd` has become a powerful search tool perfectly integrated into my workflow, allowing me to access my own knowledge base intelligently and instantly.
