# 🔀 tee — The Stream Splitter

## 🎯 The Concrete Problem

I was running a long compilation — 20 minutes of output streaming past. I needed two things at once:

1. Watch the output **live** to catch any errors as they happened.
2. Save everything to a log file to share with a colleague later.

```bash
# Attempt 1: just redirect to a file
./build.sh > build.log
```

Problem: I see nothing on screen. I had to wait 20 minutes in the dark, or open a second terminal and `tail -f build.log`. Frustrating.

```bash
# Attempt 2: copy-paste from terminal scrollback
# → limited by terminal buffer size, impossible to automate
```

That's when `tee` came to the rescue.

---

## 💡 Part 1: The "Why" — tee's Philosophy

The command `tee` takes its name from a plumbing fitting: the **T-shaped pipe connector**. Its philosophy is simple but essential: **bifurcation**.

```
Without tee:
─────────────────────────────────────────────
source_command ──────────────► stdout (terminal)
                               OR
source_command ──────────────► file
                               (but not both!)

With tee:
─────────────────────────────────────────────
                          ┌──► stdout (terminal)  ← you see it live
source_command ──► tee ──┤
                          └──► file(s)            ← saved for later
─────────────────────────────────────────────
```

> **Courtroom analogy:** Imagine a court clerk listening to the proceedings (the stream). While the speakers continue talking (sound still reaches the audience), the clerk simultaneously writes everything down in the record (the file). Two destinations, one source.

### Why this matters

- **Visibility + Persistence:** You keep an eye on the execution while guaranteeing a written trace.
- **Pipeline link:** Since `tee` sends what it receives to stdout, it can be inserted **anywhere in the middle** of a pipeline without breaking it.
- **Privilege elevation:** One of the safest ways to write to a root-protected file while staying inside a user pipeline.

---

## ⚙️ Part 2: The "How" — Syntax and Basic Usage

General structure:

```bash
command | tee [OPTIONS] [FILE...]
```

- `command |` — `tee` almost always reads from a pipe
- `[OPTIONS]` — modify write behavior (e.g., append instead of overwrite)
- `[FILE...]` — the file(s) where output should be saved

### The simplest usage

```bash
ls -l | tee listing.txt
```

The file list displays in your terminal AND gets saved to `listing.txt`. If `listing.txt` already existed, it gets **overwritten**.

### The essential option: `-a` (append)

By default, `tee` behaves like `>` — it overwrites the file. To behave like `>>` (append to end), use `-a`:

```bash
echo "New log entry" | tee -a journal.log
```

### Write to multiple files at once

`tee` can fan out to multiple targets simultaneously:

```bash
echo "System Alert" | tee log1.txt log2.txt log3.txt
```

---

## 🔑 Part 3: The Famous sudo Use Case

This is the most clever application of `tee`. I wanted to add an entry to a system-protected file:

```bash
# THIS WILL NOT WORK:
sudo echo "127.0.0.1 mysite.local" >> /etc/hosts
```

Why not? Because `sudo` applies to `echo`, but the `>>` redirection is handled by your current shell — which doesn't have root privileges.

```
What actually happens:
──────────────────────────────────────────────────────────
sudo echo "..."   → runs as root ✓
        │
        ▼
     >> /etc/hosts  ← handled by YOUR shell, no root → DENIED ✗
──────────────────────────────────────────────────────────
```

**The solution with `tee`:**
```bash
echo "127.0.0.1 mysite.local" | sudo tee -a /etc/hosts > /dev/null
```

```
How it works:
──────────────────────────────────────────────────────────
echo "..."        → runs with your user rights
        │
        ▼ pipe
sudo tee -a /etc/hosts   → tee runs as ROOT ✓ → can write to /etc/hosts
        │
        ▼
  > /dev/null    → suppress terminal output (optional, keeps it clean)
──────────────────────────────────────────────────────────
```

`echo` runs with your rights, but `tee` runs under `sudo` and has permission to write to `/etc/hosts`. The `> /dev/null` at the end just suppresses the terminal echo if you want to stay quiet.

---

## 🚀 Part 4: Advanced Tricks

### Ignore interruptions with `-i`

If you don't want `tee` to stop when you press `Ctrl+C` (SIGINT), use `-i` (**ignore interrupts**). Useful to ensure logs are fully written even if you interrupt the display:

```bash
./long_task.sh | tee -i record.log
```

### Using with process substitution

You can use `tee` to send data to two different commands simultaneously:

```bash
cat data.csv | tee >(process_accounting) >(process_marketing) > /dev/null
```

---

## 🧪 Part 5: Concrete Examples

**Case 1: Benchmark and log simultaneously**

```bash
./benchmark.sh phi3.5 | tee results_phi3.5.log
```

You see the scores live and keep the file for your records.

**Case 2: Debug a complex pipeline**

If a pipeline isn't producing the expected result, insert `tee` in the middle to inspect the data state at that exact step:

```bash
cat data.txt | grep "error" | tee /tmp/step1.txt | awk '{print $3}' | sort
```

You can then inspect `/tmp/step1.txt` to see if `grep` did its job correctly before `awk` processed it.

**Case 3: Save an installation log with date stamp**

```bash
sudo apt upgrade -y | tee upgrade_$(date +%F).log
```

---

## ✅ Conclusion

`tee` is a small tool with an outsized impact in the world of automation.

```
┌─────────────────────────────────────────────────────┐
│  tee                                                │
│  → doubles the information stream                   │
│  → solves privilege escalation with sudo            │
│  → acts as a mirror inside your pipelines           │
│  → lets you debug complex pipelines step by step    │
└─────────────────────────────────────────────────────┘
```

The next time you launch a command where you want to keep a trace without sacrificing live output, think of `tee`. It's the small T-connector that changes everything.
