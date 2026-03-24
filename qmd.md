# 🧠 qmd - Local Semantic Search for Your Notes

> **Machine:** KpihX-Ubuntu | **Date:** 2026-03-24

I have a large amount of Markdown notes in `~/Work/tutos_live` and `~/Work/Homelab/presentation`. Using `grep` works, but it's limited to exact keyword matching. I wanted something smarter, capable of understanding the intent behind my searches. That's when I discovered `qmd`.

`qmd` is a local search engine that uses AI models to "understand" the meaning of your notes and provide much more relevant results. Here's how I installed and configured it to become an essential building block of my "external memory."

## Installation and Configuration

Installation is done via `bun` (my JS/TS package manager of choice).

```bash
bun install -g @tobilu/qmd
```

Once installed, the first step is to tell it which folders to index. I have two primary knowledge sources:

1.  `~/Work/tutos_live`
2.  `~/Work/Homelab/presentation`

So I created two `qmd` "collections":

```bash
# Create the "tutos_live" collection and specify files to include
qmd collection add ~/Work/tutos_live/ -n tutos_live --mask "**/*.{md,yml,yaml,json,py,toml,sh}"

# Create the "presentation" collection for Homelab docs
qmd collection add ~/Work/Homelab/presentation/ -n presentation --mask "**/*.{md,yml,yaml,json,py,toml,sh}"
```
The `--mask` flag is crucial to tell `qmd` to index not only `.md` files, but also configuration files and scripts.

Finally, you need to initiate the semantic indexing. This command will download AI models (if it's the first time) and transform your documents into vectors.
```bash
qmd embed
```
Your knowledge base is now ready to be queried.

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

To ensure the `qmd` index is always up-to-date, I've implemented a `post-commit` Git hook in each repository. This script automatically runs `qmd update` after each commit that modifies relevant files.

**File:** `.git/hooks/post-commit` (in each repository)
```bash
#!/bin/sh
# Post-commit hook to update the qmd index.

echo "--- [QMD Hook] Running post-commit checks ---"

if ! command -v qmd > /dev/null; then
    echo "--- [QMD Hook] Error: 'qmd' command not found. Skipping update."
    exit 0
fi

# Check if relevant files were modified
if git diff --name-only HEAD~1 HEAD | grep -qE "\.(md|yml|yaml|json|py|toml|sh)$"; then
    echo "--- [QMD Hook] Relevant files were modified. Updating QMD index..."
    qmd update
    echo "--- [QMD Hook] QMD index update complete. ---"
else
    echo "--- [QMD Hook] No relevant files modified. Skipping update."
fi

exit 0
```
Remember to make it executable: `chmod +x .git/hooks/post-commit`.

With this configuration, `qmd` has become a powerful search tool perfectly integrated into my workflow, allowing me to access my own knowledge base intelligently and instantly.
