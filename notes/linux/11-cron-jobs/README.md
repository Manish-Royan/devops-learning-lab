# 🐧 Linux Learning Journey: Cron Jobs — Scheduling Automation

> **Goal:** Understand cron jobs — how the `crond` daemon runs scheduled tasks, the `crontab` syntax (5-field time pattern), common scheduling patterns, special shortcut strings (`@reboot`/`@daily`), output redirection, and troubleshooting.

---

## 📍 Why Cron Matters for DevOps

Everything you've learned so far — shell scripting, text processing, file management — has been **manual**: you run a command, it happens once. But DevOps is about **automation at scale**, and many tasks need to run **on a schedule**:

- Backups every night at 2 AM
- Log rotation every Sunday
- SSL certificate renewal every month
- Health checks every 5 minutes
- Database cleanup on the 1st of every month
- Sending a daily report at 9 AM

You can't sit at a terminal and run these manually. That's what **cron** solves — it's Linux's built-in **task scheduler**, running commands automatically at intervals you define.

> 🧭 **DevOps angle:** If a script is the *what*, cron is the *when*. Every production server runs dozens of cron jobs — system maintenance, application backups, monitoring checks, certificate renewals. Understanding cron is understanding how Linux keeps itself running without human intervention.

---

## 1️⃣ What Is a Cron Job?

A **cron job** is a scheduled task that allows you to run a script or command **automatically at specified intervals** — without manual intervention.

### 🎯 Purpose

Automate repetitive tasks:
- System maintenance
- File backups
- Sending reports
- Restarting services
- Log rotation
- Health checks

### ⚙️ How It Works

Cron relies on a background daemon service called **`crond`** that runs continuously. Every minute, `crond` wakes up, checks the cron tables, and executes any jobs whose schedule matches the current time.

```
   ┌──────────────────────────────────────────────────────────────┐
   │              HOW CRON WORKS:                                 │
   │                                                              │
   │   ┌──────────────┐                                           │
   │   │    crond     │  ← background daemon, always running      │
   │   │   (daemon)   │                                           │
   │   └──────┬───────┘                                           │
   │          │                                                   │
   │          │ Every minute, wakes up and checks:                │
   │          │   "Does any scheduled job match the current       │
   │          │    time?"                                         │
   │          │                                                   │
   │          ▼                                                   │
   │   ┌──────────────┐          ┌──────────────┐                 │
   │   │  crontab     │────────▶│  Run matching│                 │
   │   │  (user's     │  yes?    │  jobs        │                 │
   │   │   schedule)  │────────▶│              │                 │
   │   └──────────────┘          └──────────────┘                 │
   │                                                              │
   │   ⚠️ Requirement: The server/machine must be up and running  │
   │   for cron jobs to execute. If the machine is off at the     │
   │   scheduled time, the job is simply SKIPPED (not run later). │
   └──────────────────────────────────────────────────────────────┘
```

### ⚠️ Critical Requirement

**Cron jobs will only execute if the server/machine is up and running.** If your server is down at 2 AM when a backup was scheduled, that backup simply **doesn't run** — cron doesn't catch up on missed jobs. (For catch-up scheduling, you'd use `anacron`, but that's beyond today's scope.)

### 🔍 Checking if `crond` Is Running

```bash
$ systemctl status crond
● crond.service - Command Scheduler
   Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled)
   Active: active (running) since...
```

If it's not running, start it:

```bash
$ sudo systemctl start crond
$ sudo systemctl enable crond    # start on boot
```

> 💡 On Debian/Ubuntu, the service is called `cron` (not `crond`). On RHEL/Fedora/CentOS, it's `crond`.

---

## 2️⃣ Managing Cron Jobs (`crontab` Commands)

Cron jobs are configured inside a **crontab (cron table)** file. **Each user on the system has their own independent crontab file** — they are user-specific.

### 📋 The Three Essential Commands

| Command | What it does |
|---|---|
| **`crontab -e`** | **E**dit — opens an editor to add or modify scheduled tasks |
| **`crontab -l`** | **L**ist — shows all active cron entries for the current user |
| **`crontab -r`** | **R**emove — deletes ALL scheduled jobs for your user (⚠️ no confirmation!) |

```bash
# View your current cron jobs
$ crontab -l
no crontab for manish

# Edit/add cron jobs (opens your default editor, usually vim or nano)
$ crontab -e

# List after adding
$ crontab -l
* * * * * /home/manish/backup.sh

# Remove ALL jobs (careful!)
$ crontab -r
```

> ⚠️ **`crontab -r` is destructive.** It deletes all your cron jobs immediately without asking for confirmation. If you only want to remove one job, use `crontab -e` and delete that specific line instead.

### 📋 Other Useful Variations

| Command | What it does |
|---|---|
| **`crontab -e -u alice`** | Edit another user's crontab (needs `sudo`) |
| **`crontab -l -u alice`** | List another user's cron jobs |
| **`sudo crontab -e`** | Edit root's crontab (for system-level jobs) |

> 🧭 **DevOps tip:** Use `sudo crontab -e` for system-wide jobs (backups, log rotation) and regular `crontab -e` for user-specific jobs. Root's crontab runs with full privileges.

---

## 3️⃣ The Crontab Syntax Format

Every line in a crontab file follows a strict **5-part time pattern** followed by the command or script path to execute:

```
   * * * * * /path/to/script.sh
   │ │ │ │ │
   │ │ │ │ │
   │ │ │ │ └──── Day of the week  (0 - 6, where 0 = Sunday)
   │ │ │ └────── Month            (1 - 12, where 1 = January)
   │ │ └──────── Day of the month (1 - 31)
   │ └────────── Hour             (0 - 23, 24-hour format)
   └──────────── Minute           (0 - 59)
```

### 📊 The 5 Fields

| Field | Position | Range | What it controls |
|---|---|---|---|
| **Minute** | 1st | `0 - 59` | Which minute of the hour |
| **Hour** | 2nd | `0 - 23` | Which hour (24-hour format) |
| **Day of month** | 3rd | `1 - 31` | Which day of the month |
| **Month** | 4th | `1 - 12` | Which month (1 = January) |
| **Day of week** | 5th | `0 - 6` | Which day (0 = Sunday, 6 = Saturday) |

### 🎯 The `*` Means "Every"

An asterisk (`*`) in any field means **"every"** — every minute, every hour, every day, every month, every day of the week.

```
   * * * * * /script.sh
   ─ ─ ─ ─ ─
   │ │ │ │ │
   │ │ │ │ └── every day of the week
   │ │ │ └──── every month
   │ │ └────── every day of the month
   │ └──────── every hour
   └────────── every minute
   
   → Runs EVERY MINUTE of EVERY DAY (1440 times per day!)
```

### 📊 Decoding Examples

```
   30 2 * * * /backup.sh
   │  │
   │  └── at 2 AM (hour 2)
   └───── at minute 30
   
   → Runs at 2:30 AM every day

   0 9 * * 1-5 /report.sh
   │ │       │
   │ │       └── Monday through Friday (days 1-5)
   │ └── at 9 AM
   └── at minute 0
   
   → Runs at 9:00 AM, Monday through Friday
```

> 💡 **Tip:** If the syntax is confusing, use an online helper tool like **[Crontab Guru](https://crontab.guru)** to decode or build your schedules. It translates the 5-field syntax into plain English.

---

## 4️⃣ Common Scheduling Examples

Here are the schedules you'll use most often:

| Schedule | Description |
|---|---|
| `* * * * *` | Run **every single minute** |
| `*/15 * * * *` | Run **every 15 minutes** |
| `*/30 * * * *` | Run **every 30 minutes** |
| `0 * * * *` | Run **every hour** (at minute 0) |
| `0 12 * * *` | Run **every day at 12:00 PM (noon)** |
| `0 0 * * *` | Run **every day at midnight** |
| `0 0 * * 0` | Run **every Sunday at midnight** |
| `0 0 1 * *` | Run on the **1st day of every month** |
| `30 2 10,20,30 * *` | Run on the **10th, 20th, and 30th** of every month at **2:30 AM** |
| `0 9 * * 1-5` | Run **Monday through Friday at 9:00 AM** |
| `0 18 * * 1-5` | Run **Monday through Friday at 6:00 PM** |
| `0 0 * * 6` | Run **every Saturday at midnight** |

### 📋 Syntax Patterns

| Pattern | Meaning | Example |
|---|---|---|
| **`*`** | Every value | `* * * * *` = every minute |
| **`*/N`** | Every Nth value | `*/15 * * * *` = every 15 minutes |
| **`N`** | Specific value | `0 12 * * *` = at 12:00 |
| **`N,M,K`** | List of values | `10,20,30` = on the 10th, 20th, and 30th |
| **`N-M`** | Range of values | `1-5` = Monday through Friday |
| **`N-M/K`** | Range with step | `0-23/2` = every 2 hours |

### 📊 Decoded Real-World Examples

```bash
# Every minute (testing/debugging)
* * * * * /home/manish/test.sh

# Every 15 minutes (health check)
*/15 * * * * /opt/scripts/health-check.sh

# Every day at 2 AM (nightly backup)
0 2 * * * /opt/scripts/backup.sh

# Every Sunday at midnight (weekly cleanup)
0 0 * * 0 /opt/scripts/cleanup.sh

# 1st of every month at midnight (monthly report)
0 0 1 * * /opt/scripts/monthly-report.sh

# Weekdays at 9 AM (daily standup reminder)
0 9 * * 1-5 /opt/scripts/standup-reminder.sh

# On the 10th, 20th, and 30th at 2:30 AM (patch deployment)
30 2 10,20,30 * * /opt/scripts/deploy-patches.sh
```

> 🧭 **DevOps tip:** When testing a cron job, set it to `* * * * *` (every minute) first to confirm it runs. Once verified, change to the actual production schedule. Never leave a job running every minute in production unless that's the intended frequency.

---

## 5️⃣ Special Strings (Quick Shortcuts)

Instead of using the 5-star syntax, you can use built-in **special keywords** for common schedules:

| String | Equivalent | When it runs |
|---|---|---|
| **`@reboot`** | (no equivalent) | Runs automatically **every time the system boots up** |
| **`@yearly`** or **`@annually`** | `0 0 1 1 *` | Once a year (January 1 at midnight) |
| **`@monthly`** | `0 0 1 * *` | Once a month (1st at midnight) |
| **`@weekly`** | `0 0 * * 0` | Once a week (Sunday at midnight) |
| **`@daily`** or **`@midnight`** | `0 0 * * *` | Once a day at midnight |
| **`@hourly`** | `0 * * * *` | Once every hour (at minute 0) |

### 📋 Examples

```bash
# Run a script every time the system boots
@reboot /home/user/startup-script.sh

# Run a daily backup at midnight
@daily /opt/scripts/backup.sh

# Run a weekly cleanup
@weekly /opt/scripts/cleanup.sh

# Run an hourly health check
@hourly /opt/scripts/health-check.sh
```

> 💡 **`@reboot` is special** — it's the only one that doesn't follow the 5-field pattern. It runs once when the system starts, which is perfect for starting services, mounting drives, or running initialization scripts.

---

## 6️⃣ Redirecting Output & Logging

**Cron jobs run in a minimal background environment** and do not show output on your terminal. If your script generates text or errors, that output goes nowhere by default — or worse, gets emailed to the system user.

### 📋 Redirect Output to a File

```bash
# Save output to a file (overwrites each run)
* * * * * /bin/ls > /tmp/ls_output.txt

# Append output to a file (preserves history)
* * * * * /bin/ls >> /tmp/ls_output.txt
```

| Operator | Behavior |
|---|---|
| **`>`** | Overwrites the file each run |
| **`>>`** | Appends new output to the end of the file |

### 📋 Capture Both stdout AND stderr

```bash
# Capture errors too (essential for debugging)
* * * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1

# Discard all output (silent)
* * * * * /opt/scripts/cleanup.sh > /dev/null 2>&1
```

Recall from Day 4: `2>&1` merges stderr into stdout. `> /dev/null` discards everything.

### 📋 The Full Pattern for Production

```bash
# Production-ready cron job with logging
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1

# What this does:
# - Runs at 2:00 AM every day
# - Appends stdout AND stderr to /var/log/backup.log
# - You can check the log later with: tail -f /var/log/backup.log
```

> 🧭 **DevOps best practice:** Always redirect cron output to a log file. If a job fails silently, you'll have no idea why. The log file is your only debugging tool when something goes wrong at 3 AM.

---

## 7️⃣ Troubleshooting & Cron Logs

If your cron job isn't running as expected, there are two places to investigate:

### 📋 1. Check the System Cron Log

The system logs every cron job execution. Check it with `sudo`:

```bash
# Watch the cron log in real time
$ sudo tail -f /var/log/cron

# Or on Debian/Ubuntu, it might be in syslog:
$ sudo tail -f /var/log/syslog | grep CRON
```

You'll see entries like:
```
Nov 14 02:00:01 server CROND[12345]: (root) CMD (/opt/scripts/backup.sh >> /var/log/backup.log 2>&1)
Nov 14 02:00:01 server CROND[12345]: (root) MAILTO (mailed 102 bytes of output but got status 0x000b)
```

### 📋 2. Check Your Script's Log File

If you redirected output (as you should!), check that log:

```bash
$ tail -f /var/log/backup.log
Starting backup at 2026-11-14 02:00:01
Backing up /var/www/html...
Backup complete. Size: 2.3 GB
```

### 📋 Common Cron Problems

| Problem | Likely Cause | Fix |
|---|---|---|
| **Job doesn't run at all** | `crond` not running | `sudo systemctl start crond` |
| **Job doesn't run at expected time** | Wrong timezone | Check `date` and `/etc/timezone` |
| **Script works manually but not in cron** | Missing environment variables | Cron has minimal PATH — use **absolute paths** to commands |
| **Script fails — "command not found"** | PATH issue | Use `/usr/bin/python3` not `python3` |
| **Permission denied** | File not executable | `chmod +x script.sh` |
| **No output captured** | No redirection | Add `>> /var/log/job.log 2>&1` |
| **Job ran but output is empty** | Script depends on TTY | Cron has no terminal — scripts must not require user input |

### 🎯 The #1 Cron Debugging Rule

> **Always use absolute paths.** Cron runs with a minimal environment — `PATH` is often just `/usr/bin:/bin`. Your script might call `python3`, but cron doesn't know where `python3` is. Use `/usr/bin/python3` instead. Same for every command inside your script.

```bash
# ❌ WRONG — may fail in cron (PATH issue)
* * * * * script.sh

# ✅ CORRECT — absolute paths everywhere
* * * * * /home/manish/scripts/script.sh
```

And inside `script.sh`:
```bash
#!/bin/bash
# ❌ WRONG
python3 /home/manish/app.py

# ✅ CORRECT
/usr/bin/python3 /home/manish/app.py
```

---

## 8️⃣ Real DevOps Scenarios

| Scenario | Crontab Entry | Why |
|---|---|---|
| **Nightly database backup** | `0 2 * * * /opt/scripts/db-backup.sh >> /var/log/db-backup.log 2>&1` | Every day at 2 AM, backup the DB and log the result |
| **Log rotation** | `0 0 * * 0 /usr/sbin/logrotate /etc/logrotate.conf` | Every Sunday at midnight, rotate/compress old logs |
| **SSL certificate renewal** | `0 12 * * * /usr/bin/certbot renew --quiet` | Check daily at noon if certs need renewal (Let's Encrypt) |
| **Health check every 5 min** | `*/5 * * * * /opt/scripts/health-check.sh >> /var/log/health.log 2>&1` | Monitor service health and alert if down |
| **Cleanup temp files** | `0 4 * * * find /tmp -type f -mtime +7 -delete` | Every day at 4 AM, delete files older than 7 days |
| **Monthly report** | `0 0 1 * * /opt/scripts/monthly-report.sh` | 1st of every month, generate and send a report |
| **Sync to backup server** | `0 3 * * * rsync -av /data/ backup@remote:/backup/` | Every night at 3 AM, sync data to a remote backup server |
| **Restart stuck service** | `*/10 * * * * /opt/scripts/check-nginx.sh` | Every 10 minutes, check if nginx is up and restart if not |
| **On reboot — start app** | `@reboot /opt/scripts/start-app.sh` | Every time the system boots, start the application |

> 🧭 **DevOps reality:** Every production server has a crontab. Backups, monitoring, log rotation, certificate renewals, health checks — all scheduled via cron. When you inherit a server, `sudo crontab -l` (root's jobs) and `crontab -l` (your jobs) are the first commands to understand what's automated.

---

## 9️⃣ Quick Reference Cheat Sheet

### 📋 Commands

| Command | What it does |
|---|---|
| **`systemctl status crond`** | Check if cron daemon is running |
| **`crontab -e`** | Edit your cron jobs (opens editor) |
| **`crontab -l`** | List your cron jobs |
| **`crontab -r`** | Remove ALL your cron jobs (⚠️ no confirmation) |
| **`sudo crontab -e`** | Edit root's cron jobs |
| **`sudo crontab -l -u alice`** | List another user's cron jobs |
| **`sudo tail -f /var/log/cron`** | Watch cron execution log |

### 📋 Syntax

| Field | Range | Meaning |
|---|---|---|
| **Minute** | `0-59` | Minute of the hour |
| **Hour** | `0-23` | Hour (24-hour format) |
| **Day of month** | `1-31` | Day of the month |
| **Month** | `1-12` | Month (1 = January) |
| **Day of week** | `0-6` | Day (0 = Sunday) |

### 📋 Common Schedules

| Pattern | When it runs |
|---|---|
| `* * * * *` | Every minute |
| `*/15 * * * *` | Every 15 minutes |
| `0 * * * *` | Every hour |
| `0 12 * * *` | Every day at noon |
| `0 0 * * *` | Every day at midnight |
| `0 0 * * 0` | Every Sunday at midnight |
| `0 0 1 * *` | 1st of every month at midnight |
| `0 9 * * 1-5` | Weekdays at 9 AM |

### 📋 Special Strings

| String | When it runs |
|---|---|
| `@reboot` | On system boot |
| `@hourly` | Every hour (`0 * * * *`) |
| `@daily` | Every day at midnight (`0 0 * * *`) |
| `@weekly` | Every Sunday at midnight (`0 0 * * 0`) |
| `@monthly` | 1st of every month (`0 0 1 * *`) |
| `@yearly` | January 1st (`0 0 1 1 *`) |

### 🧠 Memory Hooks

- **`crontab -e`** = **e**dit
- **`crontab -l`** = **l**ist
- **`crontab -r`** = **r**emove (careful!)
- **`*`** = every
- **`*/N`** = every Nth
- **`@reboot`** = on boot
- **5 fields**: Minute, Hour, Day-of-month, Month, Day-of-week
- **Always use absolute paths** in cron jobs

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **Cron job** | A scheduled task that runs automatically at specified intervals. |
| **`crond`** | The background daemon that executes cron jobs. Must be running. |
| **`systemctl status crond`** | Check if the cron daemon is active. (On Debian/Ubuntu: `cron`) |
| **`crontab -e`** | Edit your cron jobs (opens editor). |
| **`crontab -l`** | List your cron jobs. |
| **`crontab -r`** | Remove ALL your cron jobs (⚠️ no confirmation — destructive!) |
| **`sudo crontab -e`** | Edit root's crontab (system-level jobs). |
| **User-specific** | Each user has their own independent crontab. |
| **5-field syntax** | `minute hour day-of-month month day-of-week command` |
| **Minute** | `0-59` |
| **Hour** | `0-23` (24-hour format) |
| **Day of month** | `1-31` |
| **Month** | `1-12` |
| **Day of week** | `0-6` (0 = Sunday) |
| **`*`** | Every (every minute, every hour, etc.) |
| **`*/N`** | Every Nth (e.g., `*/15` = every 15 minutes) |
| **`N,M,K`** | List (e.g., `10,20,30` = on those specific days) |
| **`N-M`** | Range (e.g., `1-5` = Monday through Friday) |
| **`@reboot`** | Run on system boot. |
| **`@hourly`** | Run every hour (`0 * * * *`). |
| **`@daily`** | Run every day at midnight (`0 0 * * *`). |
| **`@weekly`** | Run every Sunday at midnight (`0 0 * * 0`). |
| **`@monthly`** | Run on the 1st of every month (`0 0 1 * *`). |
| **`@yearly`** | Run once a year (`0 0 1 1 *`). |
| **Output redirection** | `>> /var/log/job.log 2>&1` — capture stdout AND stderr. |
| **`>`** | Overwrite file. **`>>`** | Append to file. |
| **`> /dev/null 2>&1`** | Discard all output (silent). |
| **Cron log** | `sudo tail -f /var/log/cron` (or `/var/log/syslog | grep CRON`). |
| **Minimal environment** | Cron has minimal PATH — always use absolute paths to commands. |
| **No terminal** | Cron jobs run without a TTY — scripts must not require user input. |
| **Missed jobs** | If the machine is off at the scheduled time, the job is skipped (not run later). |
| **Production pattern** | `0 2 * * * /opt/scripts/job.sh >> /var/log/job.log 2>&1` |
| **DevOps relevance** | Cron = the scheduler of Linux. Backups, monitoring, log rotation, cert renewal — all run via cron. Every production server depends on it. |

---