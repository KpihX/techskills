# 🔍 grep & Regular Expressions — The Surgical Search Superpower

## 🎯 The Concrete Problem

I was staring at hundreds of log files and source files. I needed to find a specific function I'd written, a particular error message, a configuration key — buried somewhere in thousands of lines.

1. **The manual approach:** Open each file, use your editor's "Find" feature. Slow, repetitive, impossible to automate.
2. **The `grep` approach:** In a single command, scan through millions of lines across hundreds of files and find exactly what you're looking for.

`grep` (short for **G**lobal search for **R**egular **E**xpression and **P**rint) is the UNIX tool for text search. It reads input (a file or a stream) and prints every line that contains a given search pattern. But its real power comes from the "RE" in its name: **Regular Expressions**.

This tutorial is in two parts: first `grep` as a tool, then a deep dive into the world of regular expressions (regex) — a universal skill that carries over to bash, Python, and most other languages.

---

## 🔨 Part 1: grep in Practice

Basic syntax: `grep [OPTIONS] "pattern" [file...]`

### Most commonly used options

| Option | Meaning |
|--------|---------|
| `-i` | Ignore case — match regardless of uppercase/lowercase |
| `-v` | Invert match — show lines that do **not** match |
| `-c` | Count — don't show lines, just count how many matched |
| `-l` | Files with matches — show only the **filenames** containing the pattern |
| `-n` | Line number — prefix each match with its line number |
| `-r` / `-R` | Recursive — search inside a directory and all subdirectories |
| `-o` | Only matching — print only the matched portion, not the full line |
| `-A N` | After — show N lines **after** each match |
| `-B N` | Before — show N lines **before** each match |
| `-C N` | Context — show N lines **around** each match |

**Example:**
```bash
grep -r -n -i "database_error" /var/log/
```
*Recursively searches `/var/log/` case-insensitively for "database_error", printing the filename and line number for every match.*

---

## 🧠 Part 2: Regular Expressions — The Superpower

A regular expression (or regex) is a **language for describing text patterns**. It's search on steroids. Instead of looking for literal text, you define rules.

### Why are regexes so fast?

When you hand a regex to an engine (like `grep`'s or Python's), it first **compiles** it. The engine transforms your regex string into a highly efficient data structure — most often a **finite automaton**.

```
Your regex string
      │
      ▼  compile
┌─────────────┐
│  Finite     │  ← ultra-optimized flow diagram
│  Automaton  │
└─────────────┘
      │
      ▼  scan text character by character
   Match / No match
```

The engine then reads your text character by character and navigates through this diagram at blazing speed, without complex backtracking (in most cases). That initial compilation is why regex is far faster for complex patterns than a naive search loop.

### The Regex Toolkit

Here are the core concepts — valid almost everywhere.

**1. Literal characters**

The simplest case: the letter `a` matches the character "a". The string `error` matches "error".

**2. Anchors**

They don't match a character — they match a **position**:
- `^` — start of the line. `^error` matches "error" only if it's at the beginning.
- `$` — end of the line. `error$` matches "error" only if it ends the line.

**3. The dot `.` (wildcard)**

- `.` — matches **any single character** (except a newline). `c.t` matches "cat", "cot", "c_t", etc.

**4. Character classes `[]`**

- `[abc]` — matches exactly one character: 'a', 'b', or 'c'. `gr[ae]y` matches "gray" and "grey".
- `[a-z]` — any lowercase letter. `[0-9]` for any digit.
- `[^abc]` — the `^` inside brackets means **negation**. Matches any character that is NOT 'a', 'b', or 'c'.

**5. Quantifiers**

They apply to the character or group immediately before them:

| Quantifier | Meaning | Example |
|------------|---------|---------|
| `*` | 0 or more times | `ca*t` → "ct", "cat", "caat" |
| `+` | 1 or more times | `ca+t` → "cat", "caat", but NOT "ct" |
| `?` | 0 or 1 time | `colou?r` → "color" and "colour" |
| `{n}` | exactly n times | `[0-9]{3}` → a 3-digit number |
| `{n,}` | at least n times | |
| `{n,m}` | between n and m times | |

**6. Alternation (OR `|`)**

- `|` — the OR operator. `cat|dog` matches "cat" or "dog".

**7. Grouping and Capturing `()`**

Parentheses serve two roles:
- **Grouping:** Apply a quantifier to a set. `(un)?cat` matches "cat" and "uncat".
- **Capturing:** Memorize the portion of text that matched the group. Used heavily in programming to extract data.

**8. Special character classes (shortcuts)**

| Shortcut | Equivalent | Meaning |
|----------|-----------|---------|
| `\d` | `[0-9]` | A digit (`\D` for non-digit) |
| `\w` | `[a-zA-Z0-9_]` | A word character (`\W` for non-word) |
| `\s` | whitespace | Space, tab, newline... (`\S` for non-whitespace) |

**9. Word boundaries `\b`**

This is crucial. `\b` is an anchor that matches the position between a word character (`\w`) and a non-word character (`\W`), or a string start/end.

- `\bcat\b` will match "cat" in "the cat is here", but **not** in "concatenate".

---

## 🛠️ Part 3: Regex in grep and bash

`grep` comes in several regex "flavors":

```
BRE (Basic RE)     → default mode
                     + and ? and | and () need to be escaped: \+, \?, \|, \(\)
                     grep "ca\+t"

ERE (Extended RE)  → use grep -E (or egrep)
                     special characters work natively — much cleaner
                     grep -E "ca+t"

PCRE               → use grep -P
                     adds advanced features like lookarounds
                     grep -P '(?<=user:)\w+'
```

**Example with PCRE lookahead:** `grep -P '(?<=user:)\w+'` would find `kpihx` in `user:kpihx` without including `user:` in the result.

> **Tip: Always use `grep -E` for modern, intuitive regex syntax.**

---

## 🐍 Part 4: Regex in Python

In Python, the standard module for regex is `re`.

**Important:** Always use **raw strings** `r"..."` for your regex in Python. This prevents Python from interpreting backslashes as escape characters before the regex engine sees them.

### Essential functions in the `re` module

**`re.search(r"pattern", "string")`** — finds the pattern anywhere in the string. Returns a match object if found, `None` otherwise:

```python
import re
match = re.search(r"\d{3}", "my code is 123 and 456")
if match:
    print(f"Found: {match.group(0)}")  # Prints "Found: 123"
```

`match.group(0)` returns the full match. `match.group(1)` returns the first capturing group, etc.

**`re.match(r"pattern", "string")`** — like `search`, but only looks at the **very start** of the string.

**`re.findall(r"pattern", "string")`** — finds **all** non-overlapping matches and returns them as a list:

```python
codes = re.findall(r"\d{3}", "my code is 123 and 456")
# codes = ['123', '456']
```

**`re.sub(r"pattern", r"replacement", "string")`** — replaces all occurrences. The Python equivalent of `sed`:

```python
new_string = re.sub(r"\s+", "_", "a sentence with spaces")
# new_string = "a_sentence_with_spaces"
```

**`re.compile(r"pattern")`** — if you use the same regex in a loop, compile it first for better performance:

```python
regex_ip = re.compile(r"\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b")
for line in log_file:
    if regex_ip.search(line):
        print(f"Line with IP found: {line}")
```

---

## ✅ Conclusion

Regular expressions are a language of their own. The learning curve can feel steep, but the investment pays back enormously.

```
┌─────────────────────────────────────────────────────┐
│  grep     → fast, powerful command-line searches    │
│  re (Py)  → analysis, validation, transformation   │
│             of complex text inside your programs   │
└─────────────────────────────────────────────────────┘
```

Master these concepts and you gain a superpower that saves hours — and lets you manipulate text with surgical precision, in any environment.
