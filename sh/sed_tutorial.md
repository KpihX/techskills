# ✂️ sed — The Stream Text Editor

## 🎯 The Concrete Problem

Picture this: you're on a remote server with no GUI. You need to modify a 5,000-line configuration file. Your tasks:
- Comment out every line containing the word `beta`
- Replace every occurrence of `old_database_ip` with `new_database_ip`
- Delete the empty debug lines

How would you do it?

1. **The manual approach:** Open the file in `vim` or `nano`, hunt down each occurrence, make the edit, save. Slow, error-prone, and if you need to do the same thing on 10 servers — a nightmare.
2. **The scripted approach:** Write a Python or Perl script to read the file line by line, apply the changes, write a new file. A solid solution, but heavy for simple tasks and requires knowing a programming language.

That's exactly the gap `sed` was built to fill.

---

## 💡 Part 1: The "Why" — sed's Philosophy

`sed` stands for **S**tream **ED**itor. The name contains the entire philosophy:

**Stream:** `sed` doesn't edit the file directly (by default). It reads the file — or any input, like the output of another command — line by line, like a data stream flowing through it. For each line, it applies your rules. Then it prints the result to standard output (your terminal).

```
┌─────────────────────────────────────────────────────┐
│  File / stdin                                       │
│       │                                             │
│       ▼                                             │
│  ┌─────────┐   line by line                         │
│  │  sed    │ ──────────────► Apply script rules     │
│  └─────────┘                       │               │
│                                    ▼               │
│                              stdout (terminal)     │
│                              or redirected file    │
└─────────────────────────────────────────────────────┘
```

> **Assembly line analogy:** Lines from the file are parts on a conveyor belt (the stream). `sed` is a robotic arm that performs an operation on each part (the editing). The modified part continues down the line to the output.

**Editor:** It's designed to perform edit operations: text substitution, deletion, insertion, and more.

### Why this approach wins

- **Non-destructive by default:** `sed` writes to stdout, so your original file stays intact. You can review the result before deciding to overwrite.
- **Pipe-able:** `sed` integrates perfectly into the UNIX philosophy. You can pipe output from one command into `sed`, and pipe `sed`'s output into another command.
  ```bash
  # List processes, filter with grep, then reformat with sed
  ps aux | grep 'nginx' | sed 's/\/usr\/sbin\/nginx/NGINX_PROCESS/'
  ```
- **Scriptable:** Every operation is a text command — save them in a file, reuse them, automate them, integrate them in shell scripts.

---

## ⚙️ Part 2: The "How" — Syntax and Core Commands

General structure:

```bash
sed [OPTIONS] 'script' [file...]
```

- `[OPTIONS]` — change the behavior (e.g., `-i` to edit in place)
- `'script'` — one or more editing commands, in single quotes to prevent shell interpretation
- `[file...]` — file(s) to process; if none provided, `sed` reads stdin

### The Pattern Space

For each line read, `sed` places it in a working memory area called the **pattern space**. All commands in the script operate on this copy of the line. Once all commands have run, `sed` (by default) prints the content of the pattern space.

```
┌──────────────────────────────────────────────────┐
│  Input line                                      │
│       │                                          │
│       ▼                                          │
│  [ Pattern Space ]  ← sed commands operate here │
│       │                                          │
│       ▼                                          │
│  Output (stdout) — unless suppressed with -n    │
└──────────────────────────────────────────────────┘
```

### The Most Important Command: `s` (Substitution)

This is 90% of what you'll do with `sed`. It replaces text.

**Syntax:** `s/pattern/replacement/flags`

- `s` — the substitution command
- `/` — the delimiter (you can use almost any character: `#`, `|`, `:`, etc. — very useful when your text contains `/` like file paths or URLs)
- `pattern` — a **regular expression** to find the text to replace
- `replacement` — the text to substitute in
- `flags` (optional) — modify the behavior

**Example 1: Replace the first occurrence**

File `example.txt`:
```
Hello world, this is a test. The world is beautiful.
```

```bash
sed 's/world/planet/' example.txt
# Output:
# Hello planet, this is a test. The world is beautiful.
```

Only the first "world" was replaced.

### Substitution flags

- `g` (global) — replace **all** occurrences on the line, not just the first:
  ```bash
  sed 's/world/planet/g' example.txt
  # Output:
  # Hello planet, this is a test. The planet is beautiful.
  ```

- `i` (ignore-case) — case-insensitive matching (GNU extension, may not work on non-Linux systems like macOS by default):
  ```bash
  echo "Hello" > test.txt
  sed 's/hello/Hi/i' test.txt
  # Output: Hi
  ```

- **A number** — replace only the Nth occurrence:
  ```bash
  sed 's/world/planet/2' example.txt
  # Output:
  # Hello world, this is a test. The planet is beautiful.
  ```

### Changing the delimiter

```bash
# Replace a file path — ugly with backslash escapes:
# echo "/usr/local/bin" | sed 's/\/usr\/local/\/opt/'
# Much cleaner with a different delimiter:
echo "/usr/local/bin" | sed 's#/usr/local#/opt#'
```

---

## 🎯 Part 3: Targeting Lines — Addressing

The real power of `sed` is applying a command **only to certain lines**. This is called addressing. The address goes right before the command.

**Syntax:** `[address]command`

### 1. Address by line number

- Apply to a specific line: `3s/a/b/` (substitutes only on line 3)
- Apply to a range: `5,10d` (deletes lines 5 through 10)
- From a line to the end: `15,$s/old/new/` (`$` means the last line)

**Example** — File `lines.txt`:
```
line 1
line 2 (change me)
line 3
line 4
line 5 (change me)
```

```bash
# Change only line 2
sed '2s/change me/modified/' lines.txt

# Change lines 2 and 5 — two ways:
sed -e '2s/change me/modified/' -e '5s/change me/modified/' lines.txt
sed '2s/change me/modified/; 5s/change me/modified/' lines.txt
```

### 2. Address by pattern (regular expression)

Even more powerful. The command only applies to lines matching the pattern:

- `/[Mm]essage/d` — delete all lines containing "Message" or "message"
- `/^#/d` — delete all lines starting with `#` (comments)
- `/^$/d` — delete all blank lines

**Example** — File `config.txt`:
```
# Enable cache
enable_cache=true

# Disable debug
# debug_mode=true
debug_mode=false
```

```bash
# Comment out the line that enables the cache
sed '/enable_cache=true/s/^/#/' config.txt
# Output:
# # Enable cache
##enable_cache=true
# ...
```

### 3. Address by pattern range

Apply a command from one pattern to another:

```bash
sed '/START/,/END/d' server.log
# Deletes the line containing START, the line containing END,
# and everything in between.
```

### 4. Negating an address with `!`

Apply a command to **all lines EXCEPT** those matching the address:

```bash
sed '/debug/!d' server.log
# Deletes all lines that do NOT contain 'debug'.
# Keeps only the debug lines.
```

---

## 🔨 Part 4: Other Useful Commands

- `d` (delete) — removes the entire line from the pattern space; it won't be printed:
  ```bash
  # Strip comments and blank lines
  sed '/^#/d; /^$/d' config.txt
  ```

- `p` (print) — prints the content of the pattern space. Seems useless since `sed` does that by default — but it reveals its power with `-n`.

  **The `-n` + `p` duo: a `grep` replacement**

  The `-n` option tells `sed` to **NOT** print the pattern space at the end of each cycle. Combined with `p`, you can print **only** the lines you choose:

  ```bash
  # Simulate `grep 'debug'`
  sed -n '/debug/p' config.txt
  # Only lines containing "debug" will be shown.
  ```

- `i` (insert) and `a` (append) — add text:
  - `i \text to insert` — inserts text **before** the current line
  - `a \text to append` — appends text **after** the current line

  ```bash
  # Add a line after line 2
  sed '2a \## NEW LINE ADDED ##' config.txt
  ```

- `y` (transliterate) — replaces characters one-for-one. Useful for case conversion:
  ```bash
  # Convert to uppercase
  echo "hello world" | sed 'y/abcdefghijklmnopqrstuvwxyz/ABCDEFGHIJKLMNOPQRSTUVWXYZ/'
  ```

---

## ⚠️ Part 5: In-Place Editing and Best Practices

Until now, we haven't modified any files. To do that, use the `-i` option.

**WARNING: THE `-i` OPTION IS DESTRUCTIVE.**

```bash
# This command MODIFIES config.txt directly. No going back.
sed -i '/^$/d' config.txt
```

### Best practice: create a backup

Most versions of `sed` let you create an automatic backup by adding a suffix to `-i`:

```bash
# Modifies config.txt and creates a backup at config.txt.bak with the original content
sed -i.bak 's/old/new/g' config.txt
```

**This is very strongly recommended whenever using `-i`.**

---

## 🚀 Part 6: Advanced Concepts — Going Further

`sed` has a second memory area called the **hold space**. Think of it as an internal clipboard.

```
┌────────────────┐         ┌────────────────┐
│  Pattern Space │         │   Hold Space   │
│  (current line)│         │  (clipboard)   │
└────────────────┘         └────────────────┘
     h → copy to hold            g → copy to pattern
     H → append to hold          G → append to pattern
     x → swap the two spaces
```

- `h` — copy pattern space **to** hold space (overwrites)
- `H` — append pattern space **to** hold space
- `g` — copy hold space **to** pattern space (overwrites)
- `G` — append hold space **to** pattern space
- `x` — exchange the two spaces

This enables complex operations like reversing a file's line order, or moving blocks of text.

**Example: Duplicate every line**
```bash
sed 'h;G' example.txt
# For each line:
# 1. `h` copies it to the hold space.
# 2. `G` appends the hold space (which contains the same line) to the pattern space.
# Result: every line appears twice.
```

---

## ✅ Conclusion

`sed` is not an obscure command — it's a powerful, efficient stream text processor.

- **Safe by default** — your original files are never touched unless you say so
- **Fast and lightweight** — no interpreter overhead
- **Universal** — available on virtually every UNIX-like system
- **Ideal for automation** — scriptable, pipe-able, repeatable

The key to mastering `sed` is practice. Start with simple substitutions, then add addressing, and when you hit a complex problem, remember that the hold space exists to save you.
