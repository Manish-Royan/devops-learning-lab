# 📋 The `ps` Command — Process Snapshot

> **Goal:** Master the `ps` (Process Status) command — from the basic form to `ps -ef`, `ps aux`, `ps -u`/`-g` filtering, and `ps -ejH` tree view — so you can instantly inspect any running process on a Linux server.

---

## 📍 What Is a Process?

Whenever you run an application or service on a Linux server, the system creates a **background process** for it, assigning it a **Process ID (PID)** — a unique number used to identify and manage that application.

The **`ps`** command stands for **P**rocess **S**tatus. Its purpose: **display all running processes** in the Linux system so you can monitor, track, and get detailed information about them.

> 🧭 **DevOps angle:** `ps` is your **static snapshot** tool — it shows you what's running at the exact moment you hit Enter. For live/real-time monitoring, use `top` (covered in the next note). Think of `ps` as a photograph and `top` as a video.

---

## 1️⃣ The Basic `ps` Command

If you run just `ps` in your terminal, it displays information about the running processes associated with your **current shell session only** — not the whole system.

```bash
$ ps
  PID TTY          TIME CMD
 1234 pts/0    00:00:00 bash
 1567 pts/0    00:00:00 ps
```

### 📊 What Each Column Means

| Column | Meaning |
|---|---|
| **PID** | Unique **P**rocess **ID** — the number the kernel uses to identify this process |
| **TTY** | **T**erminal **TY**pe the user is logged into (e.g., `pts/0`) |
| **TIME** | Cumulative **CPU time** (in minutes:seconds) that the process has been running |
| **CMD** | The **command** associated with the process (e.g., your `bash` shell and `ps` itself) |

```
   ┌──────────────────────────────────────────────────────┐
   │  $ ps                                                │
   │    PID TTY          TIME CMD                         │
   │   1234 pts/0    00:00:00 bash    ← your shell        │
   │   1567 pts/0    00:00:00 ps      ← the ps command    │
   │                                                      │
   │   Only shows processes from YOUR current shell       │
   │   (not the whole system)                             │
   └──────────────────────────────────────────────────────┘
```

---

## 2️⃣ Viewing All Processes: `ps -e` or `ps -A`

To see **all** processes running across the entire system (not just your current shell), use the `-e` or `-A` flag.

```bash
$ ps -e
  PID TTY          TIME CMD
    1 ?        00:00:02 systemd
    2 ?        00:00:00 kthreadd
    3 ?        00:00:00 rcu_gp
  ...
  (hundreds of lines)
```

> 💡 Since this generates a lot of output, pipe it into `more` for page-by-page viewing:
> ```bash
> ps -e | more
> ```

---

## 3️⃣ Full Format Details: `ps -ef` (The DevOps Workhorse)

In practical system administration, the most widely used variation is **`ps -ef`**:
- **`e`** = *every* process
- **`f`** = *full format*

```bash
$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 10:30 ?        00:00:02 /usr/lib/systemd/systemd
root         2     0  0 10:30 ?        00:00:00 [kthreadd]
root       742     1  0 10:30 ?        00:00:00 /usr/sbin/sshd -D
paul      1337   742  0 11:00 pts/0   00:00:00 -bash
paul      1401  1337  0 11:05 pts/0   00:00:00 ps -ef
```

### 📊 Extra Information `ps -ef` Provides

| Column | Meaning |
|---|---|
| **UID** | **U**ser **ID** — which user is running/attached to the service |
| **PPID** | **P**arent **P**rocess **ID** — what process spawned this one |
| **C** | CPU utilization (deprecated, often 0) |
| **STIME** | **ST**art **TIME** — when the process was started |
| **TTY, TIME, CMD** | (same as basic `ps`) |

> 🧭 **DevOps use case:** `ps -ef` helps you quickly check **which user launched a specific process or service**. When debugging "who started this runaway process?", this is your first move.

---

## 4️⃣ BSD-Style Detailed View: `ps aux`

Another classic format derived from Unix software distribution is **`ps aux`** — notice **no hyphen** is used here (this is the BSD style, vs. the System V style of `ps -ef`).

```bash
$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1 169372 13492 ?        Ss   10:30   0:02 /usr/lib/systemd/systemd
paul      1337  0.0  0.2  21604  5120 pts/0    Ss   11:00   0:00 -bash
postgres  2103  2.3 15.4 1428840 642100 ?      Sl   11:02   0:15 /usr/lib/postgresql/bin/postgres
```

### 📊 What `ps aux` Adds

| Column | Meaning |
|---|---|
| **`%CPU`** | Exact percentage of CPU the process is consuming |
| **`%MEM`** | Exact percentage of memory the process is consuming |
| **VSZ** | Virtual memory size (KB) |
| **RSS** | Resident set size — physical memory used (KB) |
| **STAT** | Process state (`R`=running, `S`=sleeping, `Z`=zombie, etc.) |
| **START** | When the process was started |

> 💡 This is similar to the Windows Task Manager view — you get **real CPU and memory percentages** per process, which `ps -ef` doesn't show.

### 🆚 `ps -ef` vs `ps aux` — Which to Use?

| | `ps -ef` (System V) | `ps aux` (BSD) |
|---|---|---|
| **Style** | Hyphen flags | No hyphen |
| **Shows UID/PPID?** | ✅ Yes | ❌ No (shows USER instead) |
| **Shows %CPU/%MEM?** | ❌ No | ✅ Yes |
| **Best for** | "Who started this? What's the parent?" | "What's eating my CPU/RAM?" |

Both work on modern Linux (which supports both styles). Pick based on what you need to know.

---

## 5️⃣ Practical Example: Searching for a Running Process

This is the **#1 DevOps pattern** with `ps`. If you want to check whether a specific application or service (like the **Apache Web Server**, `httpd`) is running, combine `ps` with `grep`:

```bash
$ ps -ef | grep httpd
root      1317     1  0 11:35 ?        00:00:14 /usr/sbin/httpd -DFOREGROUND
apache    6162  1317  0 11:36 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache    6164  1317  0 11:36 ?        00:01:00 /usr/sbin/httpd -DFOREGROUND
apache    6165  1317  0 11:36 ?        00:01:00 /usr/sbin/httpd -DFOREGROUND
apache    6166  1317  0 11:36 ?        00:01:02 /usr/sbin/httpd -DFOREGROUND
```

### 🔍 How to Read This Output

```
   root      1317     1  ...  /usr/sbin/httpd    ← MAIN process (PID 1317, parent=1/systemd)
   apache    6162  1317  ...  /usr/sbin/httpd    ← child worker (parent=1317)
   apache    6164  1317  ...  /usr/sbin/httpd    ← child worker
   apache    6165  1317  ...  /usr/sbin/httpd    ← child worker
   apache    6166  1317  ...  /usr/sbin/httpd    ← child worker
        │     │      │
        │     │      └── PPID (parent PID) — all workers share parent 1317
        │     └──────── PID
        └────────────── UID (root owns the main, apache owns the workers)
```

You can see:
- The **main process** owned by `root` (PID 1317)
- Its **sub-processes** (workers) owned by `apache`
- All workers share **PPID 1317** — they're children of the main process

### ✅ Cross-Verify with `systemctl`

You can cross-verify this using `systemctl status httpd`, where the **Main PID matches** the one displayed by `ps`:

```bash
$ systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service)
   Active: active (running) since Sat 2022-07-02 11:35:50
 Main PID: 1317 (httpd)                    ← 🎯 matches ps -ef output!
   Status: "Running, listening on: port 8080"
    Tasks: 213
   Memory: 29.2M
   CGroup: /system.slice/httpd.service
           ├─1317 /usr/sbin/httpd -DFOREGROUND
           ├─6162 /usr/sbin/httpd -DFOREGROUND
           ├─6164 /usr/sbin/httpd -DFOREGROUND
           ├─6165 /usr/sbin/httpd -DFOREGROUND
           └─6166 /usr/sbin/httpd -DFOREGROUND
```

> 🧭 **DevOps common use cases:** Checking for running **Java applications**, **Python scripts**, **web servers** (nginx, httpd), **databases** (postgres, mysql). The `ps -ef | grep <name>` pattern is universal.

---

## 6️⃣ Filtering Processes by User or Group

If you want to view all processes belonging to a specific user or group:

### 👤 By User: `ps -u`

```bash
# All processes owned by root
$ ps -u root

# All processes owned by paul
$ ps -u paul
```

### 👥 By Group: `ps -G` (uppercase)

```bash
# All processes associated with a group
$ ps -G paul
```

> 💡 Note the case: `-u` for user (lowercase), `-G` for group (uppercase). Easy to mix up.

---

## 7️⃣ Viewing the Process Tree: `ps -ejH`

To view processes in a **hierarchical tree format** — showing parent processes and their respective child processes — use:

```bash
$ ps -ejH
```

The flags break down as:
- **`-e`** = all processes
- **`-j`** = jobs format (shows PGID, SID)
- **`-H`** = **h**ierarchy (tree view via indentation)

### 🌳 What the Tree Shows

```
   PID  PPID  PGID   SID TTY          TIME COMMAND
     1     0     1     1 ?        00:00:02 systemd
   742     1   742   742 ?        00:00:00 sshd
  1337   742  1337  1337 ?        00:00:00   sshd: paul@pts/0
  1338  1337  1337  1337 ?        00:00:00     bash
  1401  1338  1337  1337 ?        00:00:00       ps -ejH
  1317     1  1317  1317 ?        00:00:14 httpd
  6162  1317  1317  1317 ?        00:00:00   httpd
  6164  1317  1317  1317 ?        00:01:00   httpd
  6165  1317  1317  1317 ?        00:01:00   httpd
  6166  1317  1317  1317 ?        00:01:02   httpd
```

The **indentation** shows the parent-child relationship:
- `systemd` (PID 1) is the root
- `sshd` (PID 742) is a child of systemd
- `bash` (PID 1338) is a child of sshd
- `ps -ejH` (PID 1401) is a child of bash
- `httpd` (PID 1317) is a child of systemd
- The four `httpd` workers are children of the main httpd process

### 🧭 DevOps Use Case

This helps you understand **how major applications spawn sub-processes** underneath them. For example, Apache managing multiple worker threads/sub-processes, or PostgreSQL spawning worker processes for each connection. When debugging "why did this child process die?", the tree shows you the parent — which often holds the clue.

> 💡 Another tree-viewing option: `pstree` (a dedicated command that draws an ASCII tree). And `ps -ef --forest` gives a similar forest view with full-format columns.

---

## 8️⃣ Quick Reference Cheat Sheet

| Command | What it shows |
|---|---|
| `ps` | Processes in current shell only (PID, TTY, TIME, CMD) |
| `ps -e` or `ps -A` | All processes on the system |
| `ps -ef` | All processes, full format (adds UID, PPID, STIME) |
| `ps aux` | BSD-style, adds %CPU, %MEM, VSZ, RSS, STAT |
| `ps -u <user>` | Processes owned by a specific user |
| `ps -G <group>` | Processes owned by a specific group |
| `ps -ejH` | Process tree (hierarchical, with indentation) |
| `ps -ef \| grep <name>` | **The DevOps classic** — find a specific running process |
| `ps -l` | Long format — shows nice value (NI), priority (PRI) |

### 🧠 Memory Hook

- **`ps`** = **P**rocess **S**napshot (a photograph of what's running)
- **`-e`** = **e**very process
- **`-f`** = **f**ull format
- **`-u`** = **u**ser
- **`-H`** = **H**ierarchy (tree)

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **`ps`** | Process Status — static snapshot of running processes. Default: current shell only. |
| **Basic `ps` columns** | PID (process ID), TTY (terminal), TIME (CPU time), CMD (command). |
| **`ps -e` / `ps -A`** | Show ALL processes on the system (not just current shell). Pipe to `more` for pagination. |
| **`ps -ef`** | Every process + full format. Adds UID (user), PPID (parent PID), STIME (start time). The DevOps workhorse. |
| **`ps aux`** | BSD-style (no hyphen). Adds `%CPU`, `%MEM`, VSZ, RSS, STAT. Like Windows Task Manager. |
| **`ps -ef` vs `ps aux`** | `-ef` for "who/parent?", `aux` for "CPU/RAM hog?". Both work on modern Linux. |
| **`ps -ef \| grep <name>`** | Find a specific running process. The #1 DevOps pattern. Cross-verify with `systemctl status`. |
| **`ps -u <user>`** | Filter processes by user. |
| **`ps -G <group>`** | Filter processes by group (uppercase G). |
| **`ps -ejH`** | Process tree — shows parent/child hierarchy via indentation. |
| **PPID** | Parent Process ID — what spawned this process. Useful for debugging. |
| **`ps -l`** | Long format — shows NI (nice value), PRI (priority). Useful with `nice` command. |
| **DevOps relevance** | `ps` = your process photograph. Use with `grep` to find services, with `-ef` to see users/parents, with `-ejH` to see hierarchy. |

---

← Back to [Process Management](./README.md) | Next: [The `top` Command](./2-top-command.md) →
