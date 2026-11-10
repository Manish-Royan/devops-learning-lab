# 📊 The `top` Command — Live Process Monitoring

> **Goal:** Master `top` — the real-time, live process monitor. Understand the two-section layout (system summary + process table), every column's meaning, and the interactive keys that let you kill, filter, sort, and customize on the fly.

---

## 📍 `ps` vs `top` — Snapshot vs Live

| | `ps` | `top` |
|---|---|---|
| **What it gives** | A **static snapshot** (photograph) | **Real-time, live monitoring** (video) |
| **Refreshes?** | ❌ No — runs once, prints, exits | ✅ Yes — refreshes every 3 seconds by default |
| **Interactive?** | ❌ No | ✅ Yes — press keys to sort, filter, kill |
| **Best for** | Scripts, piping to `grep`, one-off checks | Watching a system under load, finding the CPU/RAM hog |

**`top`** stands for **T**able **O**f **P**rocesses. If system performance spikes or applications slow down, `top` helps you **instantly identify** which processes or users are consuming the most CPU or RAM.

---

## 1️⃣ The `top` Interface Layout

When you type `top` and hit Enter, the screen splits into **two main sections**:

```
   ┌───────────────────────────────────────────────────────────────────┐
   │  TOP SECTION — SYSTEM SUMMARY                                     │
   │  ┌────────────────────────────────────────────────────────────┐   │
   │  │ Uptime & Load Average:  up 14 days,  3:42,  3 users        │   │
   │  │                       load average: 1.05, 0.85, 0.72       │   │
   │  │ Tasks: 230 total, 1 running, 229 sleeping, 0 stopped, 0 z  │   │
   │  │ %Cpu(s): 12.3 us,  3.1 sy,  0.0 ni, 84.5 id,  0.1 wa       │   │
   │  │ MiB Mem :  15924.0 total,  210.3 free, 10245.7 used, 5468  │   │
   │  │ MiB Swap:   2048.0 total, 2048.0 free,    0.0 used.        │   │
   │  └────────────────────────────────────────────────────────────┘   │
   ├───────────────────────────────────────────────────────────────────┤
   │  BOTTOM SECTION — PROCESS TABLE (auto-refreshing)                 │
   │  ┌──────┬────────┬───┬───┬───────┬───────┬───────┬───┬──────┬───┐ │
   │  │ PID  │USER    │PR │NI │VIRT   │RES    │SHR    │S  │%CPU  │%M │ │
   │  ├──────┼────────┼───┼───┼───────┼───────┼───────┼───┼──────┼───┤ │
   │  │ 2103 │postgres│20 │ 0 │1.4g   │642m   │12m    │S  │ 12.3 │7.5│ │
   │  │ 1337 │paul    │20 │ 0 │21604  │5120   │3360   │S  │  0.0 │0.2│ │
   │  │ ...  │        │   │   │       │       │       │   │      │   │ │
   │  └──────┴────────┴───┴───┴───────┴───────┴───────┴───┴──────┴───┘ │
   │  (columns continue: TIME+, COMMAND)                               │
   └───────────────────────────────────────────────────────────────────┘
```

---

## 2️⃣ The Top Section — System Summary

### ⏱️ Uptime & Load Average

```
top - 14:30:22 up 14 days,  3:42,  3 users,  load average: 1.05, 0.85, 0.72
```

| Part | Meaning |
|---|---|
| `14:30:22` | Current time |
| `up 14 days, 3:42` | System has been running for 14 days, 3 hours, 42 minutes |
| `3 users` | 3 users currently logged in |
| `load average: 1.05, 0.85, 0.72` | Load over **1 minute, 5 minutes, 15 minutes** |

> 💡 **Load average** = average number of processes waiting for CPU. A value equal to your CPU core count means the CPU is fully utilized. Above that = processes are waiting (potential bottleneck).

### 📋 Tasks

```
Tasks: 230 total, 1 running, 229 sleeping, 0 stopped, 0 zombie
```

| State | Meaning |
|---|---|
| **total** | All processes on the system |
| **running** | Actively using CPU right now |
| **sleeping** | Waiting for something (I/O, timer, event) — most processes are here |
| **stopped** | Suspended (e.g., via `Ctrl+Z`) |
| **zombie** | Finished but parent hasn't reaped them yet (recall Day 2 processes note) |

### 🖥️ CPU States

```
%Cpu(s): 12.3 us,  3.1 sy,  0.0 ni, 84.5 id,  0.1 wa,  0.0 hi,  0.0 si,  0.0 st
```

| Code | Meaning | What it tells you |
|---|---|---|
| **`us`** | **U**ser processes | CPU time spent running your apps (nginx, postgres, etc.) |
| **`sy`** | **S**ystem (kernel) | CPU time spent in the kernel |
| **`ni`** | **N**ice processes | CPU time spent on niced (low-priority) processes |
| **`id`** | **I**dle | CPU doing nothing — **higher is better** (means headroom) |
| **`wa`** | **W**ait (I/O) | CPU waiting for disk/network — **high = disk bottleneck** |
| **`hi`** | Hardware interrupts | |
| **`si`** | Software interrupts | |
| **`st`** | **St**eal time | CPU stolen by the hypervisor (cloud VMs) — **high = noisy neighbor** |

> 🧭 **DevOps angle:** On cloud VMs, watch **`st`** (steal). If it's consistently high, your cloud provider is overcommitting the host — another tenant is stealing your CPU. Time to migrate or resize.

### 💾 Memory Usage

```
MiB Mem :  15924.0 total,  210.3 free, 10245.7 used, 5468.0 buff/cache
MiB Swap:   2048.0 total, 2048.0 free,    0.0 used.
```

Same information as `free -h` (recall the memory note): total, free, used, buff/cache. Watch **buff/cache** — it's reclaimable. And **Swap used** — if it's climbing, you're under memory pressure.

---

## 3️⃣ The Bottom Section — Process Table

A dynamic, auto-refreshing table listing active processes:

| Column | Meaning |
|---|---|
| **PID** | Process ID |
| **USER** | The user who owns the process |
| **PR** | **Pr**iority — kernel scheduling priority |
| **NI** | **N**ice value (-20 to 19; lower = higher priority) |
| **VIRT** | **Virt**ual memory size (everything the process can address) |
| **RES** | **Res**ident set size — physical RAM currently used |
| **SHR** | **Shr**ared memory |
| **S** | **S**tate (`R`=running, `S`=sleeping, `Z`=zombie, `T`=stopped) |
| **%CPU** | Percentage of CPU currently consumed |
| **%MEM** | Percentage of RAM currently consumed |
| **TIME+** | Cumulative CPU time (with centisecond precision) |
| **COMMAND** | The command/application name |

### 🧠 VIRT vs RES vs SHR — Don't Confuse Them

```
   ┌─────────────────────────────────────────────────────────────┐
   │  PROCESS MEMORY (as shown by top)                           │
   │                                                             │
   │  VIRT ────────────────────────────  total virtual memory    │
   │  │  (everything the process CAN address — includes          │
   │  │   swapped-out, mapped files, shared libs)                │
   │  │                                                          │
   │  │   RES ──────────────  physical RAM actually used         │
   │  │   │  (what's actually in RAM right now)                  │
   │  │   │                                                      │
   │  │   │   SHR ────  memory shared with other processes       │
   │  │   │   (e.g., shared libraries)                           │
   │  │   │                                                      │
   │  │   └─ (RES includes SHR)                                  │
   │  └─ (VIRT includes RES + swapped + mapped files)            │
   └─────────────────────────────────────────────────────────────┘
```

- **VIRT** = huge number, includes everything (shared libs, mapped files, swap space) — usually not what you care about
- **RES** = the number to watch — **actual physical RAM used**
- **SHR** = subset of RES — memory shared with other processes

> 💡 When debugging "what's eating my RAM?", look at **RES**, not VIRT. VIRT can be enormous for processes that mmap large files but barely touch them.

---

## 4️⃣ Interactive Commands — Controlling `top` While It Runs

While `top` is running, press these keys to sort, filter, or control processes dynamically. **You don't need to quit `top` to use them — just press the key.**

### ⌨️ The Interactive Key Reference

| Key | What it does |
|---|---|
| **`k`** | **K**ill a process — prompts for PID, then signal (15=graceful, 9=force) |
| **`u`** | Filter by **u**ser — type a username (e.g., `root`, `paul`); blank = all |
| **`d`** or **`s`** | Change refresh **d**elay (default 3s; enter `1` for 1-second updates) |
| **`n`** | Limit **n**umber of processes displayed (e.g., `10` for top 10) |
| **`M`** | Sort by **M**emory usage (high to low) — find the RAM hog |
| **`P`** | Sort by **P**ercentage CPU (high to low) — find the CPU hog |
| **`r`** | **R**enice — change a process's nice value (-20 to 19) |
| **`f`** | **F**ield management — add/remove/reorder columns |
| **`c`** | Show **c**ommand's absolute path (not just the name) |
| **`z`** | Toggle **z**color/monochrome display |
| **`b`** | **b**old/reverse video for running processes |
| **`x`** | Highlight the sort field |
| **`y`** | Highlight running tasks |
| **`i`** | Show only **i**dle/non-zero memory processes |
| **`h`** | **H**elp — shows all available commands |
| **`q`** | **Q**uit `top` |

### 🎯 Most-Used Keys (Memorize These)

```
   ┌─────────────────────────────────────────────────────┐
   │  THE FIVE KEYS YOU'LL USE 90% OF THE TIME:          │
   │                                                     │
   │   M  = sort by memory      ← "what's eating RAM?"   │
   │   P  = sort by CPU         ← "what's eating CPU?"   │
   │   k  = kill a process      ← "stop that runaway"    │
   │   u  = filter by user      ← "show only my stuff"   │
   │   q  = quit                ← "I'm done looking"     │
   └─────────────────────────────────────────────────────┘
```

### 📋 Detailed Walkthrough of Key Operations

**`k` — Kill a Process:**
1. Press `k`
2. `top` prompts: `PID to signal/kill [default pid = 1234]`
3. Enter the PID of the runaway/hanging process
4. `top` prompts: `Signal to send [15/sigterm]`
5. Enter `15` for **graceful termination** (SIGTERM) or `9` for **force kill** (SIGKILL)

> 🧭 Recall from Day 2: SIGTERM (15) gives the process time to clean up. SIGKILL (9) is instant death with no cleanup. Always try 15 first; use 9 only as last resort.

**`u` — Filter by User:**
1. Press `u`
2. `top` prompts: `Which user (blank for all)`
3. Type a username (e.g., `root` or `paul`)
4. Only that user's processes appear

**`d` or `s` — Change Refresh Delay:**
1. Press `d` (or `s`)
2. `top` prompts: `Change delay from 3.0 to`
3. Enter a new interval in seconds (e.g., `1` for 1-second updates, `0.5` for half-second)

**`M` — Sort by Memory:**
Just press `M`. The process table instantly re-sorts by `%MEM` (highest first). Press `P` to switch back to CPU sorting.

**`r` — Renice a Process:**
1. Press `r`
2. `top` prompts: `PID to renice`
3. Enter the PID
4. `top` prompts: `Renice PID 1234 to value`
5. Enter a nice value (-20 to 19). Negative values require root.

> 💡 This is the live equivalent of the `nice`/`renice` commands (covered in the process control note). Use it to deprioritize a non-urgent backup job that's hogging CPU, or prioritize a critical query.

---

## 5️⃣ Practical DevOps Scenarios

### 🚨 Scenario 1: "Server is slow"

```bash
$ top
# Press M (sort by memory) — is something eating RAM?
# Press P (sort by CPU) — is something pegging CPU?
# Watch the wa (I/O wait) in the CPU line — is disk the bottleneck?
# Watch the load average — is it way above your core count?
```

### 🎯 Scenario 2: "Kill the runaway process"

```bash
$ top
# Press P (sort by CPU) — the hog is now at the top
# Note its PID
# Press k, enter the PID, enter 9 (force kill)
# Confirm it's gone
```

### 👤 Scenario 3: "What's the postgres user running?"

```bash
$ top
# Press u, type postgres
# Now only postgres processes are visible
# Press M to sort by memory — find the heaviest query
```

### ☁️ Scenario 4: "Cloud VM feels sluggish"

```bash
$ top
# Look at the CPU line — check the st (steal) value
# If st is consistently > 5%, your hypervisor is stealing CPU
# Time to resize the VM or switch providers
```

---

## 6️⃣ Quick Reference Cheat Sheet

### 🚀 Launching `top`

```bash
top                    # default, refreshes every 3s
top -d 1               # refresh every 1 second
top -u paul            # show only paul's processes
top -n 5               # exit after 5 refreshes (useful in scripts)
```

### ⌨️ Inside `top` (press these keys while it's running)

| Key | Action |
|---|---|
| `M` | Sort by memory |
| `P` | Sort by CPU |
| `k` | Kill process (prompts for PID + signal) |
| `u` | Filter by user |
| `r` | Renice process |
| `d`/`s` | Change refresh delay |
| `n` | Limit number of processes shown |
| `f` | Field management (add/remove columns) |
| `c` | Show full command path |
| `z` | Toggle color |
| `b` | Toggle bold |
| `x` | Highlight sort column |
| `y` | Highlight running tasks |
| `i` | Show only active/idle processes |
| `h` | Help |
| `q` | Quit |

### 🧠 Memory Hook

- **`top`** = **T**able **O**f **P**rocesses (live, refreshing)
- **`M`** = **M**emory sort
- **`P`** = **P**ercent CPU sort
- **`k`** = **k**ill
- **`u`** = **u**ser filter
- **`r`** = **r**enice
- **`q`** = **q**uit

> 💡 **Install `htop`** for a friendlier, color-coded version: `sudo apt install htop` (Debian/Ubuntu) or `sudo dnf install htop` (RHEL/Fedora). It's `top` with a much nicer interface — but `top` is on every Linux box by default, so learn it first.

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **`top`** | Table of Processes — real-time, live monitoring. Refreshes every 3s by default. |
| **`ps` vs `top`** | `ps` = static snapshot (photograph). `top` = live video. |
| **Two sections** | Top = system summary (uptime, load, tasks, CPU, memory). Bottom = process table. |
| **Load average** | 1/5/15-minute average of processes waiting for CPU. Equals core count = fully utilized. |
| **Task states** | total, running, sleeping, stopped, zombie. Most processes are sleeping. |
| **CPU `us`** | User processes (your apps). |
| **CPU `sy`** | System/kernel processes. |
| **CPU `id`** | Idle — higher = more headroom. |
| **CPU `wa`** | I/O wait — high = disk bottleneck. |
| **CPU `st`** | Steal time (cloud) — high = noisy neighbor stealing your CPU. |
| **VIRT** | Virtual memory — everything addressable. Usually a huge number. Not what you care about. |
| **RES** | Resident set size — actual physical RAM used. **The number to watch.** |
| **SHR** | Shared memory — subset of RES, shared with other processes. |
| **`M` key** | Sort by memory usage (find RAM hog). |
| **`P` key** | Sort by CPU usage (find CPU hog). |
| **`k` key** | Kill a process — prompts for PID + signal (15=graceful, 9=force). |
| **`u` key** | Filter by user. |
| **`r` key** | Renice a process (-20 to 19). |
| **`d`/`s` key** | Change refresh delay. |
| **`n` key** | Limit number of processes shown. |
| **`f` key** | Field management — add/remove/reorder columns. |
| **`c` key** | Show full command path. |
| **`z`/`b` keys** | Toggle color / bold display. |
| **`q` key** | Quit `top`. |
| **`top -d 1`** | Refresh every 1 second. |
| **`top -u <user>`** | Show only a specific user's processes. |
| **`htop`** | Friendlier color-coded alternative. Install it — but learn `top` first since it's everywhere. |
| **DevOps relevance** | `top` = your live process monitor. Sort by M/P to find hogs, k to kill, u to filter, watch `wa`/`st` for bottlenecks. |

---

← Back to [Process Management](./README.md) | Previous: [`ps` Command](./1-ps-command.md) | Next: [Exit Codes & Operators](./3-exit-codes-operators.md) →
