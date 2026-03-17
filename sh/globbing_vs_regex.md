# 🌟 Globbing vs Regex — The Complete Guide

## 🎯 The Concrete Problem

I was running `grep "ca.*t" *.txt` and something felt off. The `*` in `ca.*t` and the `*` in `*.txt` — are those the same thing? They look identical. They're absolutely not.

This is the #1 source of confusion under Linux, and it comes from two completely different systems that happen to share some symbols.

---

## 💡 Part 1: The Philosophy — Ingredient vs Recipe

### Where the words come from

- **Wildcard:** From poker. A card that can substitute for any other — the symbol itself (`*`, `?`).
- **Globbing:** From an old Unix program (`/etc/glob`). It's the **action** the shell performs to expand `*.txt` into an actual list of files (`report.txt`, `notes.txt`, `data.txt`).
- **Regex:** A full language for describing text patterns, evaluated by a program (not the shell).

### The mental model

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  GLOBBING                       REGEX                       │
│  ─────────────────────          ──────────────────────      │
│  Target: FILES on disk          Target: TEXT content        │
│  Interpreter: THE SHELL         Interpreter: THE PROGRAM    │
│  (bash, zsh expand it           (grep, sed, awk, python     │
│   BEFORE running the cmd)        evaluate it at runtime)    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## ⚖️ Part 2: The Fundamental Difference

| Concept | Globbing (Wildcards) | Regular Expressions (Regex) |
|:--------|:---------------------|:----------------------------|
| **Target** | **Files** on disk | **Text content** (strings) |
| **Interpreter** | The **Shell** (bash, zsh) | The **Program** (grep, sed, python) |
| **Symbol `*`** | "Any sequence of characters" | "Repeat the previous element 0+ times" |
| **Match everything** | `*` | `.*` |
| **Match one character** | `?` | `.` |

### The critical nuance around `*`

```
Globbing:   *.txt
            │
            └── Shell expands this to: report.txt notes.txt data.txt
                BEFORE the command even runs
                * means "any sequence of characters (including none)"

Regex:      ca*t
            │
            └── The program reads this as:
                "c" + "a repeated 0 or more times" + "t"
                Matches: "ct", "cat", "caat", "caaaat"
                * does NOT mean "any sequence" — it quantifies the preceding element
```

---

## 🗺️ Part 3: Survival Guide — Which Tool Uses What?

### Group A: File Tools (Globbing)

These manipulate "boxes" (files) — not their contents.

```bash
ls *.txt          # Shell expands *.txt before ls runs
rm *.bak          # Shell expands *.bak before rm runs
cp *.jpg /backup  # Shell expands *.jpg before cp runs
dpkg -i *.deb     # Shell expands *.deb before dpkg runs
```

### Group B: Text Tools (Regex)

These read and analyze the text *inside* files.

```bash
grep "pattern" file    # Regex interpreted by grep
sed 's/old/new/g'      # Regex interpreted by sed
awk '/pattern/'        # Regex interpreted by awk
vim (search mode)      # Regex interpreted by vim
```

### The Trap: `grep` uses BOTH

```bash
grep "ca.*t" *.txt
#    ^^^^^^  ^^^^^
#    Regex   Globbing (expanded by the Shell BEFORE grep starts)
```

```
Execution timeline of: grep "ca.*t" *.txt
──────────────────────────────────────────────────────────
1. Shell sees *.txt
2. Shell expands *.txt → report.txt notes.txt data.txt
3. Shell runs:  grep "ca.*t" report.txt notes.txt data.txt
4. grep receives "ca.*t" as a REGEX string
5. grep searches for "cat", "caat", "ct", etc. in each file
──────────────────────────────────────────────────────────
```

The shell handles the filenames. The program handles the pattern.

---

## 🐍 Part 4: Applying This in Python

Python distinguishes these two concepts clearly through two different modules.

### A. Globbing in Python (the `glob` module)

To list files, you don't use regex — you use `glob` (or `pathlib`):

```python
import glob

# Equivalent of 'ls *.py'
# Returns a list: ['script.py', 'test.py']
python_files = glob.glob('*.py')

print(f"Found {len(python_files)} Python scripts.")
```

*With `pathlib` (more modern):*
```python
from pathlib import Path

# Find all jpg files in the current directory
for image in Path('.').glob('*.jpg'):
    print(image.name)
```

### B. Regex in Python (the `re` module)

To analyze text, extract data, or validate a format:

```python
import re

text = "I have a chat, a cat, and a caaaat."

# Pattern: "c" followed by "a" (0 or more times), followed by "t"
# Note: ca*t is DIFFERENT from *.txt globbing — here * quantifies 'a'
pattern = r"ca*t"

# Find all occurrences
results = re.findall(pattern, text)

print(results)
# Output: ['chat', 'cat', 'caaaat']
# Note: 'h' in 'chat' is skipped because the pattern is strict: c + a(s) + t
# To also match 'chat', you'd need: r"c[ha]*t"
```

### Python summary

```
I want FILES?   → import glob
                  glob.glob('*.py')
                  Path('.').glob('*.jpg')

I want TEXT?    → import re
                  re.findall(r"ca*t", text)
                  re.sub(r"\s+", "_", string)
```

---

## ✅ Conclusion

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  When you type *.txt in the terminal:                   │
│    → the SHELL does the work (globbing)                 │
│    → the command receives a list of filenames            │
│                                                          │
│  When you pass "ca.*t" to grep, sed, awk, or re:        │
│    → the PROGRAM does the work (regex engine)           │
│    → the pattern is evaluated against text content       │
│                                                          │
│  They share symbols (* and ?) but are different systems. │
│  Same face. Different soul.                              │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

Once this distinction clicks, the command line becomes a lot more predictable — and a lot less mysterious.
