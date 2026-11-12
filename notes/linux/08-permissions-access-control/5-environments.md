# 🌍 Environments & Shell Configuration

> **Goal:** Understand environment variables (`PATH`, `HOME`, `USER`), how to view them (`printenv`), where they're configured (`/etc/environment`, `~/.bashrc`, `~/.bash_profile`), and how the shell startup hierarchy works.

---

## 📍 What Is an Environment?

Every process on Linux inherits a set of **key-value pairs** called **environment variables**. These tell programs things like:

- Where to find executables (`PATH`)
- Where your home directory is (`HOME`)
- Your username (`USER`)
- Which shell you're using (`SHELL`)
- Your language/locale (`LANG`)

```
   ┌──────────────────────────────────────────────────────────────┐
   │  ENVIRONMENT VARIABLES — INHERITED BY EVERY PROCESS          │
   │                                                              │
   │  PATH=/usr/local/bin:/usr/bin:/bin                           │
   │  HOME=/home/paul                                             │
   │  USER=paul                                                   │
   │  SHELL=/bin/bash                                             │
   │  LANG=en_US.UTF-8                                            │
   │  PWD=/home/paul/projects                                     │
   │  ...                                                         │
   │                                                              │
   │  When you run a command (e.g., `ls`):                        │
   │  - bash looks through PATH for `ls`                          │
   │  - finds it at /usr/bin/ls                                   │
   │  - runs it with the environment inherited                    │
   └──────────────────────────────────────────────────────────────┘
```

When you type a command, the shell looks through every directory in `PATH` to find the right executable. This elegant mechanism is completely defined by POSIX (recall Day 2).

---

## 1️⃣ Viewing Environment Variables

### `printenv` — Show All Environment Variables

```bash
$ printenv
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOME=/home/paul
USER=paul
LANG=en_US.UTF-8
PWD=/home/paul
TERM=xterm-256color
...
```

### `echo $VAR` — Show a Specific Variable

```bash
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

$ echo $HOME
/home/paul

$ echo $USER
paul
```

### 📋 The Most Common Variables

| Variable | What it tells programs |
|---|---|
| **`PATH`** | Where the shell looks for executables (colon-separated directories) |
| **`HOME`** | Your home directory path |
| **`USER`** | Your username |
| **`SHELL`** | Which shell you're using (`/bin/bash`, `/bin/zsh`) |
| **`LANG`** | Your language/locale (`en_US.UTF-8`) |
| **`PWD`** | Your current working directory |
| **`TERM`** | Your terminal type (`xterm-256color`) |
| **`EDITOR`** | Your preferred text editor (`vim`, `nano`) |

### 📋 Setting Variables

```bash
# Temporary — only for this session
$ export MY_VAR="hello"
$ echo $MY_VAR
hello

# Prepend a directory to PATH
$ export PATH="/my/custom/bin:$PATH"

# Permanent — add to ~/.bashrc (see below)
```

> 🧭 **DevOps tip:** `PATH` is the most important env var. When a command "not found" error appears but the binary exists, it's usually a `PATH` issue — the directory isn't in `PATH`. Fix: `export PATH="/the/missing/dir:$PATH"`.

---

## 2️⃣ Where Env Vars Are Configured — The Hierarchy

Environment variables are configured at **three levels**, applied in a specific order:

```
   ┌──────────────────────────────────────────────────────────────┐
   │              SHELL STARTUP HIERARCHY                         │
   │                                                              │
   │  1. /etc/environment    (system-wide, ALL users)             │
   │     ↓                                                        │
   │  2. /etc/profile        (system-wide, login shells)          │
   │     ↓                                                        │
   │  3. ~/.bash_profile     (user-specific, login shells only)   │
   │     ↓                                                        │
   │  4. ~/.bashrc           (user-specific, EVERY session)       │
   │                                                              │
   │  Order: system-wide → user login → user every-session        │
   │  Later files can override earlier ones                       │
   └──────────────────────────────────────────────────────────────┘
```

### 📋 The Three Key Files

#### 1. `/etc/environment` — System-Wide

Environment variables that should be available to **all users**. Any variables defined here are available to every user when they start a session.

```bash
$ cat /etc/environment
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
JAVA_HOME="/usr/lib/jvm/java-11-openjdk"
```

> Format: simple `KEY=value` pairs, no `export` needed.

#### 2. `~/.bashrc` — Every Session

A script that runs at the **beginning of every bash session** for a specific user. Located in `~/.bashrc` (the user's home directory). Used for customizations like:
- Setting environment variables
- Changing the prompt
- Setting colors
- Defining aliases
- Sourcing other scripts

```bash
$ cat ~/.bashrc
# User-specific aliases
alias ll='ls -la'
alias gs='git status'

# Custom PATH
export PATH="$HOME/.local/bin:$PATH"

# Custom prompt
export PS1='\u@\h:\w\$ '

# Colors
export TERM=xterm-256color
```

**Runs every time a bash session starts** — including new terminal windows, `bash` invocations, and subshells.

#### 3. `~/.bash_profile` — Login Shells Only

Runs **only at the first interactive login shell** — typically when you SSH in or log in at a physical terminal. Does NOT run for new terminal windows (those are non-login shells).

```bash
$ cat ~/.bash_profile
# Runs only on login shells (SSH, console)
export JAVA_HOME="/usr/lib/jvm/java-11-openjdk"

# The standard pattern: source .bashrc so it runs too
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi
```

### 📊 `.bashrc` vs `.bash_profile` — The Critical Distinction

| | `.bashrc` | `.bash_profile` |
|---|---|---|
| **When it runs** | **Every** bash session (new terminal, subshell) | **Only** login shells (SSH, console) |
| **What goes in it** | Aliases, prompt, colors, PATH tweaks | Login-once setup (JAVA_HOME, mail check) |
| **Runs for `gnome-terminal`?** | ✅ Yes | ❌ No |
| **Runs for SSH?** | Only if sourced by `.bash_profile` | ✅ Yes |

### 🔁 The Standard Pattern: `.bash_profile` Sources `.bashrc`

Because `.bash_profile` only runs on login shells, and you want your customizations on every session, the standard pattern is:

```bash
# ~/.bash_profile
if [ -f ~/.bashrc ]; then
    . ~/.bashrc        # source .bashrc so it runs too
fi
```

This ensures `.bashrc` runs on every session start — login or non-login.

```
   ┌──────────────────────────────────────────────────────────────┐
   │  THE PATTERN: .bash_profile SOURCES .bashrc                  │
   │                                                              │
   │  Login shell (SSH in):                                       │
   │    1. /etc/environment loads                                 │
   │    2. /etc/profile loads                                     │
   │    3. ~/.bash_profile loads                                  │
   │       └── sources ~/.bashrc ✅ (customizations apply)        │
   │                                                              │
   │  Non-login shell (new terminal window):                      │
   │    1. /etc/environment already set (inherited)               │
   │    2. ~/.bashrc loads directly ✅ (customizations apply)     │
   │                                                              │
   │  Result: customizations apply in BOTH cases                  │
   └──────────────────────────────────────────────────────────────┘
```

> 💡 If you ever make changes to `.bashrc` and they don't take effect immediately, run `source ~/.bashrc` (or `. ~/.bashrc`) to reload it without opening a new terminal.

---

## 3️⃣ Environment Variable Inheritance

When a process starts, it **inherits** the environment of its parent. This is how env vars propagate:

```
   ┌──────────────────────────────────────────────────────────────┐
   │  ENV VAR INHERITANCE:                                        │
   │                                                              │
   │  bash (has PATH, HOME, USER, MY_VAR="hello")                 │
   │    │                                                         │
   │    ├──▶ ls          (inherits all env vars)                 │
   │    ├──▶ python3     (inherits all env vars)                 │
   │    ├──▶ docker      (inherits all env vars)                 │
   │    └──▶ ./my-script.sh                                      │
   │         (inherits all env vars — including MY_VAR)           │
   │                                                              │
   │  If you export a new var in a child, the parent              │
   │  does NOT see it (inheritance is one-directional)            │
   └──────────────────────────────────────────────────────────────┘
```

```bash
# Parent sets a var
$ export API_KEY="secret123"
$ ./deploy.sh
# deploy.sh can read $API_KEY (inherited)

# Child sets a var — parent doesn't see it
$ bash -c 'export CHILD_VAR="hello"'
$ echo $CHILD_VAR
(empty)    # ← parent didn't inherit from child
```

> 🧭 **DevOps use case:** This is how `.env` files and secret injection work in CI/CD. The pipeline sets `export DB_PASSWORD=xxx`, then runs `./deploy.sh` — the script inherits `DB_PASSWORD` without it being hardcoded in the script.

---

## 4️⃣ Practical DevOps Scenarios

### 🎯 Scenario 1: Add a Custom Binary to PATH

```bash
# You installed a tool to ~/.local/bin, but `mytool` isn't found
$ mytool
bash: mytool: command not found

# Add ~/.local/bin to PATH (permanent)
$ echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
$ source ~/.bashrc

$ mytool    # now works!
```

### 🎯 Scenario 2: Set JAVA_HOME for All Users

```bash
# Add to /etc/environment (system-wide)
$ echo 'JAVA_HOME="/usr/lib/jvm/java-11-openjdk"' | sudo tee -a /etc/environment

# Log out and back in for it to take effect
# (or: source /etc/environment manually)
```

### 🎯 Scenario 3: Custom Aliases for Your Workflow

```bash
# Add to ~/.bashrc
$ cat >> ~/.bashrc << 'EOF'
alias ll='ls -la'
alias gs='git status'
alias dc='docker compose'
alias k='kubectl'
alias tf='terraform'
EOF

$ source ~/.bashrc
$ ll    # now works as ls -la
```

### 🎯 Scenario 4: Pass Env Vars to a Script

```bash
# Set for just one command (no export needed)
$ DB_PASSWORD=secret ./migrate.sh

# Or export then run
$ export DB_PASSWORD=secret
$ ./migrate.sh
```

---

## 5️⃣ Quick Reference Cheat Sheet

| Command | What it does |
|---|---|
| **`printenv`** | Display all currently set environment variables |
| **`echo $VAR`** | Show a specific variable's value |
| **`export VAR=value`** | Set an env var for the current session |
| **`VAR=value command`** | Set an env var for ONE command only |
| **`source ~/.bashrc`** (or `. ~/.bashrc`) | Reload `.bashrc` without opening a new terminal |

### 📋 Where to Put What

| File | Scope | When it runs |
|---|---|---|
| **`/etc/environment`** | All users, system-wide | Every login (PAM) |
| **`/etc/profile`** | All users, login shells | Login shells only |
| **`~/.bash_profile`** | One user, login shells | SSH/console login only |
| **`~/.bashrc`** | One user, every session | Every new bash session |

### 🧠 Memory Hooks

- **`printenv`** = **print env**ironment variables
- **`export`** = "export this var to child processes"
- **`.bashrc`** = **rc** = "run commands" (every session)
- **`.bash_profile`** = **profile** = login profile (once)
- **Pattern**: `.bash_profile` sources `.bashrc` → customizations apply everywhere

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **Environment variables** | Key-value pairs inherited by every process (PATH, HOME, USER, etc.). |
| **`printenv`** | Display all currently set environment variables. |
| **`echo $VAR`** | Show a specific variable's value. |
| **`export VAR=value`** | Set a variable for the current session (inherited by children). |
| **`PATH`** | Colon-separated list of directories the shell searches for executables. |
| **`/etc/environment`** | System-wide env vars — available to ALL users. Simple `KEY=value` format. |
| **`~/.bashrc`** | Runs at the start of EVERY bash session. User-specific customizations: aliases, prompt, PATH tweaks. |
| **`~/.bash_profile`** | Runs ONLY at first interactive login shell (SSH, console). Login-once setup. |
| **`.bashrc` vs `.bash_profile`** | `.bashrc` = every session. `.bash_profile` = login shells only. |
| **Sourcing pattern** | `.bash_profile` sources `.bashrc` → customizations apply in both login and non-login shells. |
| **`source ~/.bashrc`** | Reload `.bashrc` without opening a new terminal. Shortcut: `. ~/.bashrc`. |
| **Inheritance** | Child processes inherit parent's env vars. One-directional — children can't push vars back to parent. |
| **`VAR=value command`** | Set an env var for ONE command only (no export needed). |
| **DevOps relevance** | Env vars = how config, secrets, and PATH are managed. `.bashrc` = where aliases and customizations live. |

---

← Back to [Permissions & Access Control](./README.md) | Previous: [Inodes & Links](./4-inodes-links.md) |
