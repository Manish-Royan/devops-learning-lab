# 🔗 Exit Codes & Process Operators — Chaining Commands

> **Goal:** Understand exit codes (0-255), the three process operators (`&&`, `||`, `;`), and subcommands (`$()`) — the foundation of every shell script, CI/CD pipeline, and automation you'll ever write.

---

## 📍 Why This Matters for DevOps

Every command you run in Bash returns an **exit code** when it finishes. Every shell script, every `Dockerfile` line, every GitHub Actions step, every Ansible task — they all depend on exit codes to decide "did this work? should I continue?"

If you've ever written:
```yaml
# GitHub Actions
- run: ./build.sh && ./deploy.sh
```
...you've used process operators. Understanding exit codes is understanding *why* that `&&` works — and *when* it might silently fail.

---

## 1️⃣ Exit Codes — How Commands Report Success or Failure

Every time a program finishes, it returns an **exit code** — a number that tells you **how the process finished**.

### 🔢 Checking the Exit Code: `$?`

Bash stores the last command's exit code in the special variable **`$?`**. Check it immediately after running a command:

```bash
$ date
Tue Nov 10 14:30:00 UTC 2026

$ echo $?
0                              # ← date finished successfully, returned 0
```

```bash
$ yes
y
y
y
... (Ctrl+C to stop)

$ echo $?
130                            # ← you stopped it with Ctrl+C, returned 130 (non-zero = didn't finish naturally)
```

```bash
$ false
$ echo $?
1                              # ← false's entire existence is to return non-zero

$ true
$ echo $?
0                              # ← true always returns 0
```

### 📊 The Exit Code Reference Table

Exit codes range from **0 to 255** (technically 0-256, but 256 wraps to 0). Here are the common ones you'll encounter:

| Code | Meaning |
|---|---|
| **0** | ✅ **Success** — the command completed without errors |
| **1** | ⚠️ General catch-all "there was an error" |
| **2** | Bash internal error — you used bash incorrectly |
| **126** | Permission denied or file isn't executable |
| **127** | **Command not found** — you typed something that doesn't exist |
| **128** | The `exit` command itself had a problem (e.g., non-integer exit code) |
| **130** | Program ended with **`Ctrl+C`** (128 + SIGINT signal #2) |
| **137** | Program ended with **SIGKILL** (128 + SIGKILL signal #9) |
| **255** | Out of bounds — you tried to exit with a code larger than 255 |

```
   ┌──────────────────────────────────────────────────────────────┐
   │                   EXIT CODE SPECTRUM                         │
   │                                                              │
   │   0 ────────────────────────────────────────────────── 255   │
   │   ✅                                                   ⚠️    │
   │  SUCCESS                                       OUT OF BOUNDS │
   │                                                              │
   │   0    = success                                             │
   │   1    = general error                                       │
   │   2    = bash misuse                                         │
   │   126  = permission denied / not executable                  │
   │   127  = command not found                                   │
   │   128  = bad exit command                                    │
   │   130  = Ctrl+C (SIGINT)                                     │
   │   137  = SIGKILL (kill -9)                                   │
   │   255  = out of bounds                                       │
   │                                                              │
   │   RULE: 0 = success, anything else = failure                 │
   │   (Most of the time, you only care about 0 vs non-0)         │
   └──────────────────────────────────────────────────────────────┘
```

> 💡 **The 128 + N pattern:** Codes 128-143 mean "killed by signal N." `130 = 128 + 2` (SIGINT is signal 2). `137 = 128 + 9` (SIGKILL is signal 9). Recall the signals from Day 2.

### 🧭 The Practical Rule

Most of the time, **you only need to worry about whether it's `0` or not `0`**:
- `0` → success → continue
- non-zero → failure → stop, debug, or run a fallback

Some programs use codes 5-100 to signify different failure types — you can usually ignore that. Just check: **zero or not-zero?**

---

## 2️⃣ Process Operators — Chaining Commands

Exit codes become powerful when you use them to **chain commands**. Bash gives you three operators that decide whether the next command runs based on the previous one's exit code.

### 📋 The Three Operators

| Operator | Name | When does the next command run? |
|---|---|---|
| **`&&`** | AND | Only if the **first succeeded** (exit 0) |
| **`\|\|`** | OR | Only if the **first failed** (non-zero) |
| **`;`** | Semicolon | **Always** — no matter what the first returned |

```
   ┌──────────────────────────────────────────────────────────────┐
   │                   PROCESS OPERATOR LOGIC                     │
   │                                                              │
   │   A && B     "run B only if A succeeded"                     │
   │   ┌─────┐    success(0)    ┌─────┐                           │
   │   │  A  │ ──────────────▶ │  B  │  ✅ B runs                │
   │   └─────┘                  └─────┘                           │
   │   ┌─────┐    fail(non-0)   ┌─────┐                           │
   │   │  A  │ ────✗──────────  │ B   │  ❌ B skipped             │
   │   └─────┘                  └─────┘                           │
   │                                                              │
   │   A || B     "run B only if A failed"                        │
   │   ┌─────┐    success(0)    ┌─────┐                           │
   │   │  A  │ ────✓────────── │  B  │  ❌ B skipped              │
   │   └─────┘                 └─────┘                            │
   │   ┌─────┐    fail(non-0)  ┌─────┐                            │
   │   │  A  │ ──────────────▶ │  B  │  ✅ B runs                │
   │   └─────┘                 └─────┘                            │
   │                                                              │
   │   A ; B      "run B no matter what"                          │
   │   ┌─────┐    any code      ┌─────┐                           │
   │   │  A  │ ──────────────▶ │  B  │  ✅ B always runs         │
   │   └─────┘                 └─────┘                            │
   └──────────────────────────────────────────────────────────────┘
```

### ✅ `&&` — Run if First Succeeds (AND)

The **`&&`** operator runs the second command **only if the first command completes successfully** (returns exit code 0). If the first fails, the second **will not execute**.

```bash
# Create a file, then append the date, then append uptime
# Each step only runs if the previous one succeeded
$ touch status.txt && date >> status.txt && uptime >> status.txt
```

If `touch status.txt` fails (e.g., permission denied), the `date` and `uptime` commands **won't run** — preventing garbage from being appended to a non-existent file.

```bash
$ true && echo hi
hi                             # ← true succeeded, so echo runs

$ false && echo hi
$                              # ← false failed, so echo skipped (no output)
```

> 🧭 **DevOps use case:** This is how CI/CD pipelines chain steps. `./build.sh && ./test.sh && ./deploy.sh` — if the build fails, tests and deploy are automatically skipped. No need for complex `if` statements.

### 🔀 `||` — Run if First Fails (OR)

The **`||`** operator runs the second command **only if the first command fails** (non-zero exit). If the first succeeds, the second is skipped.

```bash
$ false || echo hi
hi                             # ← false failed, so echo runs as fallback

$ true || echo hi
$                              # ← true succeeded, so echo skipped
```

> 🧭 **DevOps use case:** Fallbacks. `mkdir /opt/myapp || echo "Directory already exists, continuing..."` — try to create the directory, and if it fails (already exists), print a message instead of crashing the script.

### 🔄 `;` — Run No Matter What (Semicolon)

The **`;`** operator runs commands **sequentially, regardless of exit codes**. Whether the first succeeds or fails, the second runs.

```bash
$ true ; false ; echo hello
hello                          # ← all three run, regardless of exit codes
```

Note: `true` runs (returns 0), `false` runs (returns 1), `echo hello` runs (returns 0). The semicolon doesn't care about any of it.

> 🧭 **DevOps use case:** When you want commands to run in sequence but don't care about dependencies. `cd /tmp ; ls` — list `/tmp` even if `cd` somehow fails (though it shouldn't). Use sparingly — `&&` is usually safer because it stops on failure.

---

## 3️⃣ Combining Operators — Real-World Patterns

### 🎯 Pattern 1: "Try this, or else do that"

```bash
# Try to start nginx; if it fails, show the logs
systemctl start nginx || journalctl -u nginx -n 50
```

### 🎯 Pattern 2: "Do A, then B if A worked, otherwise C"

```bash
# Build, then deploy if build succeeds, otherwise notify
./build.sh && ./deploy.sh || ./notify-failure.sh
```

> ⚠️ **Careful with this pattern!** `A && B || C` has a subtle trap: if `A` succeeds but `B` fails, `C` runs too (because `B`'s failure triggers `||`). This isn't always what you want. For complex logic, use `if` statements.

### 🎯 Pattern 3: "Setup, then run, then cleanup — always cleanup"

```bash
# Run setup, then the main job, then cleanup (no matter what)
./setup.sh && ./main-job.sh ; ./cleanup.sh
```

Here `cleanup.sh` runs whether `main-job.sh` succeeded or failed — because `;` doesn't care about exit codes.

### 🎯 Pattern 4: The Deployment Chain (CI/CD classic)

```bash
# Every step depends on the previous one succeeding
./lint.sh && ./test.sh && ./build.sh && ./deploy.sh && ./notify-success.sh
```

If any step fails, the entire chain stops — and you'll know exactly where it broke by checking the logs.

---

## 4️⃣ Subcommands — `$()` (Command Substitution)

A **subcommand** uses the syntax **`$(command)`** to execute a separate command and **capture its standard output**. The returned value is then **inserted into the parent command** at that position.

### 📝 Basic Examples

```bash
$ echo "I think $(whoami) is a really cool user"
I think paul is a really cool user

$ echo "The current date is $(date)"
The current date is Tue Nov 10 14:30:00 UTC 2026
```

How it works:
1. Bash sees `$(whoami)` inside the `echo` command
2. It runs `whoami` in a subshell
3. `whoami` outputs `paul`
4. Bash substitutes `paul` back into the original command
5. The final command becomes: `echo "I think paul is a really cool user"`

```
   ┌──────────────────────────────────────────────────────────────┐
   │  echo "I think $(whoami) is a really cool user"              │
   │                  │                                           │
   │                  └── Bash spots $(...)                       │
   │                      │                                       │
   │                      ▼                                       │
   │                  Run `whoami` in a subshell                  │
   │                  Output: "paul"                              │
   │                      │                                       │
   │                      ▼                                       │
   │                  Substitute output back in                   │
   │                      │                                       │
   │                      ▼                                       │
   │  echo "I think paul is a really cool user"   ← final command │
   │  Output: "I think paul is a really cool user"                │
   └──────────────────────────────────────────────────────────────┘
```

### 📋 Practical Example — Building a Log Entry

Let's say you have a job that runs every day, checks the date and uptime, and appends that to a file:

```bash
$ echo "$(date +%x) - $(uptime -p)" >> log.txt

$ cat log.txt
11/10/2026 - up 14 days, 3 hours, 42 minutes
```

- `$(date +%x)` → outputs the date in `MM/DD/YYYY` format
- `$(uptime -p)` → outputs how long the system has been up, prettified
- Both outputs get concatenated into the `echo`, then appended to `log.txt`

### 📋 Reading From a File via Subcommand

```bash
$ echo "$(cat < log.txt)"
11/10/2026 - up 14 days, 3 hours, 42 minutes
```

This runs `cat < log.txt`, captures its output, and passes it to `echo`. (A more idiomatic way is just `cat log.txt` directly, but this shows the substitution pattern.)

### 🕰️ The Old Way: Backticks `` `command` ``

```bash
$ echo "hi `whoami`, how are you"
hi paul, how are you
```

Backticks are the **old, legacy syntax** for command substitution. They still work for backward compatibility, but **they're not recommended in modern DevOps practice** because:

| | `$()` (modern) | `` ` ` `` (old backticks) |
|---|---|---|
| **Nesting** | ✅ Easy: `$(echo $(date))` | ❌ Hard: requires escaping `` \`echo \`date\`\` `` |
| **Readability** | ✅ Clear | ❌ Easy to confuse with single quotes |
| **Modern best practice** | ✅ Yes | ❌ Avoid — use `$()` instead |

> 💡 **Always use `$()` in modern scripts.** Backticks are a relic — they'll work, but they'll make your code harder to read and impossible to nest cleanly.

---

## 5️⃣ Putting It All Together — A Real Automation Script

Here's a small script that uses everything from this note — exit codes, all three operators, and subcommands:

```bash
#!/bin/bash
# Daily health check script

# 1. Create a log file with today's date in the filename
LOGFILE="health-$(date +%Y-%m-%d).txt"

# 2. Try to create the file; if it fails, exit with error
touch "$LOGFILE" || { echo "Cannot create $LOGFILE"; exit 1; }

# 3. Append system info (subcommands capture date + uptime)
echo "=== Health Check: $(date) ===" >> "$LOGFILE"
echo "Uptime: $(uptime -p)" >> "$LOGFILE"
echo "User: $(whoami)" >> "$LOGFILE"

# 4. Check if nginx is running; report status
systemctl is-active nginx && echo "nginx: ✅ running" >> "$LOGFILE" || echo "nginx: ❌ not running" >> "$LOGFILE"

# 5. Cleanup old logs, no matter what (semicolon)
find /tmp -name "health-*.txt" -mtime +7 -delete ; echo "Old logs cleaned up"

# 6. Check the exit code of the whole script
echo "Script finished with exit code: $?"
```

Every line uses a concept from this note:
- **Line 3** → subcommand `$(date +%Y-%m-%d)`
- **Line 6** → `||` fallback with `{ }` grouping
- **Lines 9-11** → subcommands `$(date)`, `$(uptime -p)`, `$(whoami)`
- **Line 14** → `&&` and `||` for conditional success/failure reporting
- **Line 17** → `;` to run cleanup regardless of prior exit codes
- **Line 20** → `$?` to check the final exit code

---

## 6️⃣ Quick Reference Cheat Sheet

### 🔢 Exit Codes

| Code | Meaning |
|---|---|
| `0` | Success |
| `1` | General error |
| `2` | Bash misuse |
| `126` | Not executable / permission denied |
| `127` | Command not found |
| `130` | Ctrl+C (SIGINT) |
| `137` | SIGKILL (kill -9) |
| `255` | Out of bounds |

### 🔗 Operators

| Operator | Name | Runs next command if... |
|---|---|---|
| `&&` | AND | first **succeeded** (exit 0) |
| `\|\|` | OR | first **failed** (non-zero) |
| `;` | Semicolon | **always** (ignores exit code) |
| `$()` | Subcommand | captures output, inserts it into parent command |

### 🧠 Memory Hooks

- **`$?`** = "how did the last command do?" (0 = good, non-zero = bad)
- **`&&`** = "do this AND that" (only if the first worked)
- **`||`** = "do this OR that" (the second is a fallback)
- **`;`** = "do this, then that" (no matter what)
- **`$()`** = "run this, capture the output, paste it here"

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **Exit code** | Every command returns 0-255 when it finishes. Stored in `$?`. |
| **0 = success** | Zero means the command completed successfully. |
| **Non-zero = failure** | Anything other than 0 means failure. The specific number tells you why. |
| **`echo $?`** | Check the exit code of the last command. |
| **Common codes** | 0=success, 1=error, 127=not found, 130=Ctrl+C, 137=SIGKILL, 255=out of bounds. |
| **128+N pattern** | Codes 128+ mean "killed by signal N." 130=128+2 (SIGINT), 137=128+9 (SIGKILL). |
| **`&&` (AND)** | Run second command **only if first succeeded** (exit 0). |
| **`\|\|` (OR)** | Run second command **only if first failed** (non-zero). |
| **`;` (semicolon)** | Run commands **always**, regardless of exit codes. |
| **`$()` (subcommand)** | Run a command, capture its output, insert it into the parent command. |
| **Backticks `` ` ` ``** | Old legacy syntax for subcommands. Avoid — use `$()` instead (nestable, readable). |
| **CI/CD chaining** | `build && test && deploy` — if any step fails, the chain stops. |
| **Fallback pattern** | `A \|\| B` — try A, if it fails run B. |
| **Cleanup pattern** | `A ; cleanup` — run cleanup no matter what A returned. |
| **DevOps relevance** | Exit codes + operators = the foundation of every shell script, Dockerfile, CI/CD pipeline, and Ansible task. |

---

← Back to [Process Management](./README.md) | Previous: [`top` Command](./2-top-command.md) | Next: [Process Control](./4-process-control.md) →
