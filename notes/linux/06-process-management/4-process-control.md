# 🎛️ Process Control — `jobs`, `bg`, `fg`, `nice`, `nohup`

> **Goal:** Master the five commands that let you **control** processes — not just view them. Background and foreground jobs with `jobs`/`bg`/`fg`, prioritize with `nice`, and keep processes alive after you log out with `nohup`.

---

## 📍 Viewing vs Controlling

The previous notes covered **viewing** processes (`ps` for snapshots, `top` for live monitoring). This note covers **controlling** them:

- **Background/foreground** a process (`jobs`, `bg`, `fg`)
- **Prioritize** a process (`nice`)
- **Persist** a process across terminal disconnects (`nohup`)

These five commands are the difference between *watching* a process and *managing* it.

---

## 1️⃣ Foreground vs Background — The Core Concept

When you execute a task in your terminal, it runs either in the **foreground** or the **background**:

```
   ┌──────────────────────────────────────────────────────────────┐
   │  FOREGROUND                           BACKGROUND             │
   │  ┌──────────────────────┐            ┌──────────────────────┐│
   │  │ Takes over the       │            │ Runs behind the      ││
   │  │ terminal — you can't │            │ scenes — your prompt ││
   │  │ type anything else   │            │ is free for other    ││
   │  │ until it finishes    │            │ work                 ││
   │  └──────────────────────┘            └──────────────────────┘│
   │                                                              │
   │  $ ./long-script.sh                 $ ./long-script.sh &     │
   │  (terminal blocked)                 [1] 12345                │
   │                                     $ (prompt free!)         │
   └──────────────────────────────────────────────────────────────┘
```

- **Foreground** (default) — the process takes over your terminal. You can't type anything else until it finishes (or you interrupt it).
- **Background** — the process runs behind the scenes. Your terminal prompt is immediately free for other commands.

Three commands manage this dance: `jobs`, `bg`, and `fg`.

---

## 2️⃣ `jobs` — List Active Terminal Jobs

Running **`jobs`** lists all **active or stopped jobs** associated with your current terminal session.

### 📋 Example Workflow

```bash
$ sleep 100
^Z                            # ← Ctrl+Z suspends/stops the process
[1]+  Stopped                 sleep 100

$ jobs
[1]+  Stopped                 sleep 100
```

### 🔍 What Just Happened?

1. You ran `sleep 100` — a command that does nothing for 100 seconds (foreground)
2. You pressed **`Ctrl+Z`** — this **suspends/stops** the process and places it into the background as a **stopped job**
3. You typed `jobs` — it shows the job ID (`[1]`) and status (`Stopped`)

### 📊 Job Status Values

| Status | Meaning |
|---|---|
| **Running** | The job is actively executing in the background |
| **Stopped** | The job is suspended (paused) — not consuming CPU |
| **Done** | The job finished successfully (shows briefly) |
| **Terminated** | The job was killed |

The `[1]` is the **job ID** (different from the PID). You use it with `bg` and `fg` to control the job.

> 💡 **Job ID vs PID:** Job IDs (`[1]`, `[2]`) are specific to your terminal session. PIDs (1234, 5678) are system-wide. You use `%N` syntax with job IDs: `bg %1`, `fg %2`.

---

## 3️⃣ `bg` — Resume a Suspended Job in the Background

**`bg`** resumes a suspended job and runs it in the **background**, freeing up your terminal prompt for other work.

### 📋 Syntax

```bash
bg %[job_id]
```

### 📋 Example

```bash
$ sleep 100
^Z
[1]+  Stopped                 sleep 100

$ jobs
[1]+  Stopped                 sleep 100

$ bg %1
[1]+ sleep 100 &              # ← job is now RUNNING in the background

$ jobs
[1]+  Running                 sleep 100    # ← status changed to Running

$                             # ← prompt is free! you can do other things
```

The job that was **Stopped** is now **Running** in the background — and your terminal is free.

> 🧭 **DevOps use case:** You started a long-running import script in the foreground, but forgot to add `&`. Instead of killing it and restarting, press `Ctrl+Z` then `bg %1` — it continues running in the background without losing progress.

---

## 4️⃣ `fg` — Bring a Job to the Foreground

**`fg`** brings a background or stopped job back to the **foreground** — so it takes over your terminal again.

### 📋 Syntax

```bash
fg %[job_id]
```

### 📋 Example

```bash
$ sleep 100 &
[1] 12345

$ jobs
[1]+  Running                 sleep 100

$ fg %1                       # ← bring job 1 to the foreground
sleep 100                     # ← now it's in the foreground, terminal blocked
```

### 🔄 The Full bg/fg Dance

```
   ┌──────────────────────────────────────────────────────────────┐
   │  THE bg/fg WORKFLOW                                          │
   │                                                              │
   │  1. Start a job in the foreground:                           │
   │     $ ./long-script.sh                                       │
   │     (terminal blocked)                                       │
   │          │                                                   │
   │          ▼  Ctrl+Z                                           │
   │                                                              │
   │  2. Job is now STOPPED in the background:                    │
   │     [1]+  Stopped    ./long-script.sh                        │
   │          │                                                   │
   │          ├── bg %1  ──▶  job RUNNING in background          │
   │          │                   (prompt free)                   │
   │          │                                                   │
   │          └── fg %1  ──▶  job back in FOREGROUND             │
   │                              (terminal blocked again)        │
   │                                                              │
   │  3. To run a job in the background from the start:           │
   │     $ ./long-script.sh &                                     │
   │     [1] 12345                                                │
   │     $ (prompt immediately free)                              │
   └──────────────────────────────────────────────────────────────┘
```

---

## 5️⃣ `nice` — Adjust Process Priority

Every Linux process has a **Nice value** that controls its scheduling priority.

### 📊 The Nice Scale

```
   ┌──────────────────────────────────────────────────────────────┐
   │                   THE NICE VALUE SCALE                       │
   │                                                              │
   │   -20 ─────────── 0 ─────────── 19                           │
   │    │              │              │                           │
   │    ▼              ▼              ▼                           │
   │  HIGHEST        DEFAULT       LOWEST                         │
   │  PRIORITY       PRIORITY      PRIORITY                       │
   │  (VIP ticket)  (normal)       (yields CPU)                   │
   │                                                              │
   │   ← more CPU attention ─────── yields CPU to others →        │
   │                                                              │
   │   Negative values (−20 to −1): require root/sudo             │
   │   Positive values (0 to 19): any user can set                │
   └──────────────────────────────────────────────────────────────┘
```

| Value | Priority |
|---|---|
| **`-20`** | **Highest priority** (most favorable — gets the most CPU) |
| **`0`** | Default (normal) |
| **`19`** | **Lowest priority** (least favorable — yields CPU to others) |

### 🔍 Checking a Process's Nice Value

You can check a process's nice value using `ps -l` (the **NI** column) or by inspecting its PID:

```bash
$ ps -l
F S   UID   PID  PPID  C  PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
0 S  1000  1337  1336  0  80   0 -  5401 -      pts/0    00:00:00 bash
                                          ↑
                                       NI = 0 (default nice value)
```

### 🚀 Running a Process with a Custom Nice Value

To give a script or application higher or lower priority **upon launch**, use the `nice` command:

```bash
nice -n <nice_value> <command>
```

```bash
# Launch script.sh with nice value of -10 (HIGH priority — needs sudo)
$ sudo nice -n -10 ./script.sh

# Launch a backup job with nice value of 15 (LOW priority — won't interfere)
$ nice -n 15 ./backup.sh
```

### ⚠️ Permissions

| Nice value | Who can set it |
|---|---|
| **Negative** (-20 to -1) | **Only root/sudo** — you can't give yourself VIP priority |
| **0** (default) | Any user |
| **Positive** (1 to 19) | Any user (you can deprioritize your own processes) |

> 🧭 **DevOps use case:** Run backups and log compression at **nice 15-19** so they don't compete with your production app for CPU. Run critical batch jobs at **nice -10** (with sudo) so they get priority. Use `renice` to change a running process's priority (or press `r` inside `top`).

### 🔄 `renice` — Change Priority of a Running Process

```bash
# Change running process PID 12345 to nice value 10
$ sudo renice -n 10 -p 12345

# Or interactively inside top:
# Press r, enter PID, enter new nice value
```

---

## 6️⃣ `nohup` — Survive Terminal Disconnection

This is one of the most important commands for DevOps engineers running long-lived processes on remote servers.

### ⚠️ The Problem

Normally, if you run a script or process in a terminal and **close that terminal window** (or lose connection — SSH drops, VPN dies, laptop sleeps), **Linux automatically terminates all child processes tied to that terminal**.

```
   WITHOUT nohup:
   ┌───────────────────────────────────────────────┐
   │  SSH session ──▶ bash ──▶ ./long-script.sh  │
   │                       │                       │
   │  SSH disconnects ───▶ SIGHUP sent ──▶ script│
   │                       (hangup)        DIES    │
   └───────────────────────────────────────────────┘
```

The terminal sends **SIGHUP** (Signal Hangup) to its child processes when it closes — and they die.

### ✅ The Solution: `nohup`

**`nohup`** stands for **"No Hang Up"**. It allows a process to **keep running in the background even after you close the terminal or log out**.

```bash
nohup ./script.sh &
```

- `nohup` tells the process to **ignore SIGHUP** signals
- `&` puts it in the background (so your prompt is free)

```
   WITH nohup:
   ┌────────────────────────────────────────────────┐
   │  SSH session ──▶ bash ──▶ nohup ./script.sh &│
   │                       │                        │
   │  SSH disconnects ───▶ SIGHUP sent ──▶ script │
   │                       IGNORED       KEEPS      │
   │                                     RUNNING    │
   └────────────────────────────────────────────────┘
```

### 📄 The `nohup.out` File

When you use `nohup`, Linux automatically creates a file named **`nohup.out`** in the current directory. All **standard output and logs** generated by your script are saved here so you can review them later.

```bash
$ nohup ./import-data.sh &
[1] 12345
nohup: ignoring input and appending output to 'nohup.out'

$ logout
# (script keeps running...)

# Later, log back in:
$ cat nohup.out
Starting import...
Processed 1000 records...
Processed 2000 records...
Import complete!
```

### 🎯 Redirecting Output (Better Practice)

By default, `nohup` writes to `nohup.out` — but for real DevOps work, you'll usually **redirect** output explicitly:

```bash
# Send output to a specific log file instead of nohup.out
nohup ./import-data.sh > /var/log/import.log 2>&1 &
```

This sends both stdout (`>`) and stderr (`2>&1`) to `/var/log/import.log`, which is cleaner than relying on the default `nohup.out`.

> 🧭 **DevOps use case:** You SSH into a production server, start a 4-hour data migration, and your VPN drops. Without `nohup`, the migration dies and you have to start over. With `nohup`, it keeps running — you log back in, check `nohup.out`, and pick up where you left off.

---

## 7️⃣ Putting It All Together — Real-World Scenarios

### 🎯 Scenario 1: "I started a long job and forgot to background it"

```bash
$ ./big-import.sh
# (5 minutes later, realizing this will take hours)
^Z                            # suspend it
[1]+  Stopped    ./big-import.sh

$ bg %1                       # resume in background
[1]+ ./big-import.sh &

$ jobs                        # confirm it's running
[1]+  Running    ./big-import.sh

$                             # terminal is free, keep working
```

### 🎯 Scenario 2: "I need this backup to survive my logout"

```bash
$ nohup ./backup.sh > /var/log/backup.log 2>&1 &
[1] 12345

$ exit
# (backup keeps running even after you disconnect)

# Log back in tomorrow:
$ tail /var/log/backup.log
Backup started at 2026-11-10 14:30:00
... backing up /home ...
... backing up /var ...
Backup completed successfully at 2026-11-10 18:45:32
```

### 🎯 Scenario 3: "My backup job is hogging the CPU"

```bash
# Start the backup with LOW priority so it doesn't compete with production
$ nice -n 19 ./backup.sh

# Or if it's already running, renice it:
$ ps -ef | grep backup.sh
paul      9876  1337  0 14:30 pts/0    00:00:05 ./backup.sh

$ renice -n 19 -p 9876
9876 (process ID) old priority 0, new priority 19

# Or interactively in top:
# Press r, enter 9876, enter 19
```

### 🎯 Scenario 4: "The production API needs priority"

```bash
# Launch the API with HIGH priority (needs sudo)
$ sudo nice -n -10 ./api-server

# It now gets CPU preference over other processes
```

### 🎯 Scenario 5: "The full DevOps workflow"

```bash
# 1. SSH in, check what's running
$ ps -ef | grep python

# 2. Start a critical migration with high priority, surviving logout
$ sudo nice -n -5 nohup ./migrate.sh > /var/log/migrate.log 2>&1 &
[1] 12345

# 3. Check it's running
$ jobs
[1]+  Running    sudo nice -n -5 nohup ./migrate.sh > /var/log/migrate.log 2>&1 &

# 4. Watch it live in top
$ top
# Press u, type root, watch the migration process

# 5. Log out — migration keeps running
$ exit

# 6. Log back in, check progress
$ tail -f /var/log/migrate.log
```

---

## 8️⃣ Modern Alternatives (Worth Knowing)

While `nohup` + `&` + `jobs` works, modern DevOps often uses better tools for long-running processes:

| Tool | What it does | When to use |
|---|---|---|
| **`screen`** / **`tmux`** | Terminal multiplexers — keep a full shell session alive across disconnects | Interactive work where you need to come back to the same terminal |
| **`systemd` services** | Run processes as managed services with auto-restart | Production daemons (nginx, postgres, your app) |
| **`docker`** | Run processes in containers with restart policies | Containerized applications |
| **`nohup`** | Quick-and-dirty "survive my logout" | One-off scripts, quick jobs |

> 🧭 **DevOps best practice:** For **production daemons**, use `systemd` services (auto-restart, logging, dependency management). For **one-off long jobs** (data migrations, backups), `nohup` is perfectly fine. For **interactive sessions** across disconnects, use `tmux` or `screen`.

---

## 9️⃣ Quick Reference Cheat Sheet

| Command | What it does | Example |
|---|---|---|
| **`jobs`** | List active/stopped jobs in current terminal | `jobs` |
| **`bg %N`** | Resume job N in the background | `bg %1` |
| **`fg %N`** | Bring job N to the foreground | `fg %2` |
| **`Ctrl+Z`** | Suspend the current foreground process | (keyboard shortcut) |
| **`&`** | Run a command in the background from the start | `./script.sh &` |
| **`nice -n <val> <cmd>`** | Launch a process with custom priority (-20 to 19) | `sudo nice -n -10 ./api.sh` |
| **`renice -n <val> -p <PID>`** | Change priority of a running process | `renice -n 19 -p 12345` |
| **`nohup <cmd> &`** | Run a process that survives terminal disconnect | `nohup ./backup.sh &` |
| **`nohup.out`** | Default file where nohup saves output | (auto-created) |

### 🧠 Memory Hooks

- **`jobs`** = "what jobs do I have?" (list them)
- **`bg`** = "**b**ack**g**round" (resume suspended, in background)
- **`fg`** = "**f**ore**g**round" (bring back to front)
- **`nice`** = "be **nice** to other processes" (lower priority = nicer; higher priority = less nice)
- **`nohup`** = "**no h**ang**up**" (ignore the hangup signal, survive logout)

### 📊 Nice Value Quick Reference

```
   -20 ← HIGHEST priority (VIP, needs root)
     0 ← DEFAULT
    19 ← LOWEST priority (yields CPU, anyone can set)
```

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **Foreground** | Process takes over the terminal — you can't type until it finishes. |
| **Background** | Process runs behind the scenes — prompt is free. |
| **`&`** | Append to a command to run it in the background from the start. |
| **`Ctrl+Z`** | Suspend the current foreground process (stops it, puts in background). |
| **`jobs`** | List all active/stopped jobs in your current terminal session. Shows job ID + status. |
| **`bg %N`** | Resume suspended job N **in the background** (prompt free). |
| **`fg %N`** | Bring job N back to the **foreground** (terminal blocked). |
| **Job ID vs PID** | Job ID (`[1]`, `[2]`) is terminal-specific. PID is system-wide. Use `%N` for job IDs. |
| **Nice value** | Process priority: -20 (highest) to 19 (lowest). Default 0. |
| **Negative nice** | Higher priority — requires root/sudo. Like a VIP ticket for CPU. |
| **Positive nice** | Lower priority — any user can set. Yields CPU to others. |
| **`nice -n <val> <cmd>`** | Launch a process with a custom nice value. |
| **`renice -n <val> -p <PID>`** | Change the nice value of a running process. |
| **`top` + `r` key** | Renice interactively inside `top`. |
| **SIGHUP** | Signal sent to processes when their terminal closes — they die. |
| **`nohup`** | "No Hang Up" — process ignores SIGHUP, survives terminal disconnect. |
| **`nohup.out`** | Default file where `nohup` saves stdout/stderr. |
| **Better logging** | `nohup ./script.sh > /var/log/script.log 2>&1 &` — redirect explicitly. |
| **Modern alternatives** | `tmux`/`screen` for interactive sessions, `systemd` for daemons, `docker` for containers. |
| **DevOps relevance** | `nohup` for one-off long jobs, `nice` for prioritizing backups vs production, `jobs`/`bg`/`fg` for terminal workflow management. |

---

← Back to [Process Management](./README.md) | Previous: [Exit Codes & Operators](./3-exit-codes-operators.md) |
