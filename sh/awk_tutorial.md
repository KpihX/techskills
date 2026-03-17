# 🔧 awk — The Column-Aware Text Processor

## 🎯 The Concrete Problem

I had an `access.log` file where every line looked like this:

```
192.168.1.10 - - [10/Jan/2026:10:05:15 +0000] "GET /page.html HTTP/1.1" 200 1542
```

I needed three things:
1. Show only the IP address and the requested page for each line.
2. Calculate the total bytes transferred (the second-to-last field) across all requests.
3. List every request that returned a 404 error.

With `sed` or `grep`, I was stuck. `grep` could find lines containing "404" — but how do you extract and *calculate* a sum from them? `sed` can extract text using complex regular expressions, but it has no concept of "column" or "field", and it can't do arithmetic.

That's when `awk` stepped in. It was designed from the ground up to understand and process **column-structured data**.

---

## 💡 Part 1: The "Why" — awk's Philosophy

The name `awk` comes from its creators: **A**ho, **W**einberger, and **K**ernighan (the 'K' from "K&R C"). Its philosophy is radically different from `grep` or `sed`.

`awk` reads a file line by line — just like the others — but for each line it performs one extra, crucial step: it **splits the line into fields**.

**The core model of `awk` is: `PATTERN { ACTION }`**

- **PATTERN:** A condition. "Does this line match what I'm looking for?" It's optional — if you omit the pattern, the action applies to every line.
- **ACTION:** What to do if the line matches. It's a code block. If you omit the action, the default is to print the entire line (`{ print }`).

**How awk processes data:**

```
┌─────────────────────────────────────────────────┐
│  For each line in the file:                     │
│    1. Split the line into fields ($1, $2, ...)  │
│    2. Does the line match PATTERN?              │
│         YES → execute ACTION                    │
│         NO  → skip                             │
│  After last line: run END block (if any)        │
└─────────────────────────────────────────────────┘
```

It's a proper little programming language dedicated to text processing.

---

## ⚙️ Part 2: The "How" — Syntax, Fields, and Basic Actions

General structure:

```bash
awk 'script' [file...]
```

### The Fields: The Magic of `$`

`awk` stores the split fields in special numbered variables prefixed with `$`:

- `$0` — the entire line (the full record)
- `$1` — the first field
- `$2` — the second field
- `$N` — the Nth field

By default, fields are separated by one or more spaces or tabs.

**Example 1: Print specific columns**

Take the output of `ls -l`:

```
-rw-r--r-- 1 user group  432 Jan 10 10:20 report.txt
drwxr-xr-x 2 user group 4096 Jan  9 09:15 backups
```

To display the size (5th field) and the name (9th field):

```bash
ls -l | awk '{ print $5, $9 }'
# Output:
# 432 report.txt
# 4096 backups
```

The comma in `print` adds the default output separator (a space).

---

## 🎯 Part 3: The Power of PATTERN — Filtering Lines

### 1. Filtering with a regular expression

Like `grep`, you can filter lines that match a regex:

```bash
# Show size and name only for directories (lines starting with 'd')
ls -l | awk '/^d/ { print $5, $9 }'
# Output:
# 4096 backups
```

### 2. Filtering with field comparisons

This is where `awk` leaves every other tool behind. You can use comparison operators (`==`, `!=`, `>`, `<`, `>=`, `<=`) on any field:

```bash
# Show files larger than 1000 bytes
ls -l | awk '$5 > 1000 { print $5, $9 }'

# Solve our log problem: find all 404 errors
# HTTP status code is the 9th field in our log format
awk '$9 == 404 { print $0 }' access.log
```

### 3. Combining patterns

Use `&&` (AND) and `||` (OR) for complex filters:

```bash
# 404 errors coming from a specific IP
awk '$1 == "192.168.1.50" && $9 == 404 { print $7 }' access.log
```

---

## 🧮 Part 4: The Power of ACTION — Variables, Calculations, and Special Blocks

### 1. Variables and arithmetic

`awk` can declare variables and perform arithmetic operations.

**Example: Calculate total disk space used**

```bash
ls -l | awk '{ total += $5 } END { print "Total:", total, "bytes" }'
```

Breaking this down:
- `{ total += $5 }` — for every line, add the 5th field value (the size) to a variable called `total`. `awk` initializes it to 0 automatically.
- `END { ... }` — a **special block** that runs **once**, after all lines are read. Perfect for printing a final result.

### 2. `BEGIN` and `END` blocks

```
Timeline of awk execution:
───────────────────────────────────────────────────
 BEGIN { }     ← runs ONCE before reading any line
   │
   ▼
 PATTERN { ACTION }   ← runs for EACH line
   │
   ▼
 END { }       ← runs ONCE after the last line
───────────────────────────────────────────────────
```

```bash
awk 'BEGIN { print "IP\tPAGE" } { print $1, $7 }' access.log
```

### 3. Built-in variables

`awk` provides very useful built-in variables:

| Variable | Meaning |
|----------|---------|
| `NR` | Number of Records — the current line number (across all files) |
| `NF` | Number of Fields — count of fields in the current line |
| `$NF` | The **last field** — extremely handy |
| `FS` | Field Separator (default: whitespace) |
| `OFS` | Output Field Separator (default: space) |

**Example: Processing a CSV file**

CSV files use a comma as separator. Tell `awk` with the `-F` option:

File `users.csv`:
```
1,alice,France
2,bob,USA
3,charlie,France
```

```bash
# Display the names of French users
awk -F',' '$3 == "France" { print $2 }' users.csv
# Output:
# alice
# charlie
```

### 4. Associative arrays: the ultimate feature

`awk` supports associative arrays (like Python dictionaries), where indices can be strings. Perfect for counting things.

**Example: Count requests per HTTP status code**

```bash
awk '{ counts[$9]++ } END { for (code in counts) print code, counts[code] }' access.log
```

What happens:
- `{ counts[$9]++ }` — for each line, use the status code (`$9`) as an array key and increment its value. If the key doesn't exist yet, `awk` creates it.
- `END { ... }` — after reading everything, iterate through `counts` and print each key (status code) with its value (occurrence count).

---

## 📜 Part 5: awk as a Script Language

For complex logic, write your script in a file and run it with the `-f` option.

File `analyze.awk`:
```awk
# This is a comment
BEGIN {
  FS = ","
  print "Analyzing users..."
}

$3 == "France" {
  french_count++
}

END {
  printf "There are %d French users out of %d total lines.\n", french_count, NR
}
```

Run it:
```bash
awk -f analyze.awk users.csv
```

---

## ✅ Conclusion: When to use awk?

`awk` is the tool to reach for when your data has structure — even a simple one.

```
┌─────────┬──────────────────────────────────────────────┐
│  grep   │ finds lines                                  │
│  sed    │ edits lines                                  │
│  awk    │ extracts, filters, manipulates, computes     │
│         │ data from FIELDS within those lines          │
└─────────┴──────────────────────────────────────────────┘
```

`awk` cleanly replaces Python/Perl/Ruby scripts for a huge range of tasks: log analysis, CSV processing, data reformatting. Its syntax is dense but incredibly expressive. Once you internalize `PATTERN { ACTION }` and the power of fields, `awk` becomes one of the sharpest tools in your command-line arsenal.
