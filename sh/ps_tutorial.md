# 🔬 ps — The Art of Inspecting Processes

## 🎯 The Concrete Problem

Your computer or server runs dozens — sometimes hundreds — of programs simultaneously. A running program is called a **process**.

Now imagine these situations:

- Your server suddenly slows to a crawl. Is a process consuming 100% CPU?
- You launched a script in the background. Is it still running?
- The web server isn't responding. Is the `nginx` or `apache2` process even started?
- You see a suspicious command and want to know who launched it and when.

To answer these questions, you need a way to list active processes and get information about them. That's exactly the role of `ps` (short for **P**rocess **S**tatus).

### ps vs top/htop: The key difference

```
ps      → snapshot  📷  Takes a photograph of process state at a given moment T
top     → live feed 🎥  Films process activity continuously and refreshes in real time
htop    → live feed 🎥  Same as top, but with a better user interface
```

---

## 💡 Part 1: The "Why" — ps's Philosophy (and its Confusion)

`ps`'s mission is straightforward: display information about active processes. However, over the history of UNIX systems, different versions of `ps` emerged — each with its own syntax. That's the source of the confusion.

There are three main **option styles** for `ps`:

1. **UNIX style (System V):** Options are preceded by a dash `-`.
   - Example: `ps -ef`

2. **BSD style:** Options have **no** dash.
   - Example: `ps aux`

3. **GNU long style:** Options are preceded by two dashes `--`.
   - Example: `ps --forest`

The good news: on most modern Linux systems, these styles can coexist. The bad news: it can make reading other people's commands tricky. The best approach is to memorize the two most common invocations and understand what each does.

---

## 🔨 Part 2: The Two Commands You Absolutely Must Know

Forget the myriad options for now. 99% of the time, you'll use one of these two.

### 1. `ps aux` (BSD style)

Probably the most popular `ps` command.

- `a` — show processes of **all** users, not just yours
- `u` — display detailed info in a **u**ser-oriented format
- `x` — also show processes not attached to a terminal (the background daemons / services)

**Typical `ps aux` output:**
```
USER         PID  %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1   0.0  0.1 169324 11332 ?        Ss   Jan09   0:02 /sbin/init
www-data     850   0.2  0.5 834572 45120 ?        Sl   Jan09   5:30 nginx: worker process
kpihx       2150   0.0  0.2 229892 21888 pts/0    Ss   10:30   0:00 -zsh
```

**The most important columns:**

| Column | Meaning |
|--------|---------|
| `USER` | The user who launched the process |
| `PID` | **Process ID** — the unique identifier. You'll use this to kill the process. |
| `%CPU` | CPU percentage consumed — **ideal for finding resource hogs** |
| `%MEM` | RAM percentage consumed |
| `COMMAND` | The command that was executed to start the process |

### 2. `ps -ef` (UNIX style)

The other classic — often preferred by "old-school" sysadmins.

- `-e` — show **every** process (equivalent to `a` + `x`)
- `-f` — show in "full" format, which has the advantage of revealing parent-child relationships

**Typical `ps -ef` output:**
```
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 Jan09 ?        00:00:02 /sbin/init
root         849       1  0 Jan09 ?        00:00:00 nginx: master process /usr/sbin/nginx
www-data     850     849  0 Jan09 ?        00:05:30 nginx: worker process
kpihx       2150    2149  0 10:30 pts/0    00:00:00 -zsh
```

**Key columns here:**

| Column | Meaning |
|--------|---------|
| `UID` | The user |
| `PID` | The process ID (same as before) |
| `PPID` | **Parent Process ID** — the ID of the process that spawned this one |
| `CMD` | The command |

`PPID` is the big advantage of this view. You can see that `nginx: master` (PID 849) spawned the `worker` (PID 850).

```
                  PID 849  nginx: master
                      │
                      └──► PID 850  nginx: worker  (PPID = 849)
```

### Which one to use?

```
ps aux   → resource view  — who is consuming what?
ps -ef   → system view    — understand process hierarchy and dependencies
```

---

## 🎛️ Part 3: Customizing Output — The `-o` Option

You're not limited to the default formats. The `-o` option (or `o` in BSD style) lets you choose exactly which columns you want:

```bash
# Show only PID, user, and command for all processes
ps -eo pid,user,comm

# The ultimate example: find the 5 most memory-hungry processes
ps -eo pid,user,%mem,comm --sort=-%mem | head -n 6
```

Breaking this down:
- `-eo pid,user,%mem,comm` — request 4 specific columns
- `--sort=-%mem` — sort by the `%mem` column; the `-` means descending (largest first)
- `| head -n 6` — pipe to `head` to keep only 6 lines (1 header + 5 processes)

---

## 🔗 Part 4: ps and grep — The Indispensable Duo (and its Pitfall)

The most common task: find the PID of a specific process. The classic method is combining `ps` with `grep`.

```bash
ps aux | grep "nginx"
# Output:
# www-data     850     0.2  0.5 834572 45120 ?  Sl  Jan09  5:30 nginx: worker process
# kpihx       2300     0.0  0.0  12345   880 pts/0  S+ 11:00  0:00 grep "nginx"
```

**The trap:** `grep` shows itself in the list! The command `grep "nginx"` contains the word "nginx".

**The classic workaround:**
```bash
ps aux | grep "[n]ginx"
```

The regex `[n]ginx` matches "nginx", but the `grep` command itself is now `grep "[n]ginx"`, which no longer self-matches.

**The modern, cleaner approach: `pgrep`**

`pgrep` was built exactly for this:
```bash
# Find the PID of nginx
pgrep nginx

# Show PID and full command line
pgrep -a nginx
```

Prefer `pgrep` when you just want to find a process.

---

## 🚀 Part 5: Practical and Advanced Examples

### 1. See the process tree

To understand who launched what, the tree view is fantastic:
```bash
ps axf
# or, with the -ef view:
ps -ef --forest
```

Output (showing hierarchy):
```
/sbin/init
  └── nginx: master process
        └── nginx: worker process
  └── -zsh
        └── ps -ef --forest
```

### 2. The classic kill workflow

Find it, then kill it:
```bash
# 1. Find the PID of the problematic script
ps aux | grep "runaway_script.py"

# 2. Kill the process
kill 12345  # Replace 12345 with the actual PID

# Or better yet, with pkill (combines pgrep + kill)
pkill -f "runaway_script.py"
```

---

## ✅ Conclusion

`ps` is your window into the soul of your system. It tells you what's running, who launched it, and how it's behaving.

- Remember the two key commands: `ps aux` (resource view) and `ps -ef` (hierarchical view)
- Never forget its **snapshot** nature — it's a photo, not a live feed like `top`
- Learn to combine it with `grep`, `sort`, `head`, and `kill` for effective system administration
- For simple process lookups, prefer modern tools like `pgrep` and `pkill` — they're more direct and less error-prone

With `ps` in your arsenal, a slow system or a hung process is no longer a black box — it's a problem you can diagnose and solve.
