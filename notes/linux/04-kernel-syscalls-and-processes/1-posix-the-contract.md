# 📜 POSIX — The Contract

> **Goal:** Understand what POSIX is, why it exists, and the specific interfaces it standardizes — so you understand *why* a shell script runs on macOS, Linux, and WSL without modification.

---

## 📍 Why POSIX Exists — The Unix Wars

Have you ever wondered why you can write a shell script on your Mac, copy it to a Linux server, and it *just works*? Or why tools like `grep`, `ls`, and `cat` behave almost identically everywhere? That's not magic — that's **POSIX**.

### ⚔️ The Chaos (1970s–1980s)

AT&T created UNIX in the 1970s. It was brilliant — and very quickly, different companies took UNIX and modified it their own way:

- **IBM** → AIX
- **HP** → HP-UX
- **Sun Microsystems** → SunOS / Solaris
- **Berkeley** → BSD

The problem: a program written for one UNIX system would **break on another**. Developers were stuck rewriting the same code over and over just to run it on different machines.

> Imagine being a developer in 1985. You write a fantastic application. It runs perfectly on IBM's AIX. But your client runs HP-UX. You have to rewrite almost everything. Time wasted, money lost, frustration everywhere.

This was the **portability problem** — and it was genuinely slowing down the entire software industry. The **Unix wars** were real.

### 🕊️ The Peace Treaty (1988)

The **IEEE** (Institute of Electrical and Electronics Engineers) sat down and said: *"Enough. Let's define a common standard."* In **1988**, they published the first **POSIX** standard.

**POSIX** = **P**ortable **O**perating **S**ystem **I**nterface. The **X** at the end is a nod to UNIX.

The goal was simple but powerful: **define a common interface so that programs written to the standard could run on any compliant operating system without modification.**

```
   BEFORE POSIX (1980s):                AFTER POSIX (1988+):

   ┌──────┐  ┌──────┐  ┌──────┐       ┌─────────────────────────┐
   │ AIX  │  │ HP-UX│  │SunOS │       │   POSIX CONTRACT        │
   │ code │  │ code │  │ code │       │   (one API to rule      │
   │      │  │      │  │      │       │    them all)            │
   └──┬───┘  └──┬───┘  └──┬───┘       └───────────┬─────────────┘
      │         │         │                        │
      ▼         ▼         ▼            ┌───────────┼───────────┐
   broken!   rewrite!  rewrite!        ▼           ▼           ▼
                                       AIX        HP-UX       Linux
                                    (compliant) (compliant) (follows)
```

---

## 1️⃣ What POSIX Actually Standardizes

Think of POSIX as a **rule book**. It says: "If you call yourself a POSIX-compliant operating system, your OS *must* provide these specific system calls, these behaviors, these tools."

It covers:
- The **file system** interface
- **Processes** (creation, communication, termination)
- **Threads**
- **I/O**
- **Signals**
- **Shell** commands
- And more

Crucially, POSIX is **not about how the OS does things internally** — it's about **the interface the OS exposes to programs**.

> **Analogy — the power outlet standard:** You don't care about the wiring inside the wall, as long as your plug fits. POSIX is the outlet shape; the kernel is the wiring. Different countries (kernels) can wire it differently, but as long as the outlet (interface) matches, your plug (program) works.

---

## 2️⃣ Everything Is a File

One of POSIX's most important concepts: **almost everything is a file.**

| Resource | Treated as |
|---|---|
| Your hard drive | A file |
| A network socket | A file |
| A keyboard | A file |
| A running process | Files in `/proc/` |
| Kernel settings | Files in `/proc/sys/`, `/sys/` |

This **"everything-is-a-file"** philosophy is genius because it means the **same functions** — `open`, `read`, `write`, `close` — work on pretty much anything. A developer doesn't need to learn 20 different APIs for 20 different devices. One consistent interface handles them all.

---

## 3️⃣ File Descriptors — The Ticket Stubs

When you open a file in a POSIX system, the OS gives you back a small integer called a **file descriptor (fd)**. These little numbers are like **ticket stubs** — hand it back to the OS and it knows exactly which file you mean.

```
   ┌─────────────────────────────────────────────────┐
   │           FILE DESCRIPTOR TABLE                 │
   │                                                 │
   │   fd 0  →  stdin   (standard input — keyboard)  │
   │   fd 1  →  stdout  (standard output — screen)   │
   │   fd 2  →  stderr  (standard error — screen)    │
   │   fd 3  →  first file you open                  │
   │   fd 4  →  second file you open                 │
   │   ...   →  and so on                            │
   └─────────────────────────────────────────────────┘
```

The first three are **always reserved**:

| fd | Name | Default | Purpose |
|---|---|---|---|
| **0** | `stdin` | Keyboard | Where a program reads input |
| **1** | `stdout` | Screen | Where a program writes normal output |
| **2** | `stderr` | Screen | Where a program writes error messages |

Everything else you open starts at **3**. This is why `ls -la /proc/$$/fd` shows you exactly what files your shell currently has open.

> 🧭 **DevOps angle:** This is the foundation of **redirection** (`>`, `2>`, `2>&1`) and **pipes** (`|`). When you write `command 2>&1`, you're literally telling the shell: "make fd 2 point to the same place as fd 1." It's file descriptor manipulation.

---

## 4️⃣ Standard Streams — The Three-Channel Model

POSIX defines **three standard streams** that every process inherits:

```
                     ┌─────────────────────┐
       stdin (0) ──▶│                     │
   (keyboard input)  │     YOUR PROGRAM    │
                     │                     │──▶ stdout (1)  → normal output (screen/file)
                     │                     │
                     │                     │──▶ stderr (2)  → error messages (screen/file)
                     └─────────────────────┘
```

This separation is **brilliant**. You can:
- Redirect `stdout` to a file to save output, while errors still appear on screen
- Redirect `stderr` to a log file, while normal output goes to the screen
- Pipe `stdout` to another command, while errors stay visible

```bash
# Save output to file, errors to screen
command > output.txt

# Save errors to file, output to screen
command 2> errors.log

# Save both to the same file
command > all.log 2>&1

# Pipe output to grep, discard errors
command 2>/dev/null | grep "pattern"
```

This clean three-stream model is something every developer eventually falls in love with.

---

## 5️⃣ Pipes — Composing Programs

One of the most powerful ideas in POSIX and the Unix philosophy: a **pipe** connects the **output** of one program directly to the **input** of another.

Remember how file descriptors work? A pipe is just **two file descriptors** — a **write end** and a **read end**. You write data into one end and it pops out the other.

```
   ┌──────────┐    write end      read end    ┌──────────┐
   │ program A│──────[fd 1]═══[fd 0]──────────│ program B│
   │ (ls)     │         ║ pipe ║              │ (grep)   │
   └──────────┘         ║      ║              └──────────┘
                        ║      ║
                   data flows left → right
```

This powers the classic Unix command chaining:

```bash
ls -la | grep ".txt" | sort | head -10
```

- `ls` lists files → output goes into the pipe
- `grep` reads from the pipe, filters for `.txt` → output goes into next pipe
- `sort` reads, sorts alphabetically → output goes into next pipe
- `head` reads, keeps first 10 lines → output goes to screen

**Each program does one thing well. Pipes let you compose them into something powerful.** This is the Unix philosophy made literal — and it's all built on file descriptors.

---

## 6️⃣ Signals — Tiny Notifications

A **signal** is a tiny notification sent to a process. You've used signals without even knowing it:

| Signal | Trigger | Meaning |
|---|---|---|
| **SIGINT** | `Ctrl + C` | "Please interrupt/stop" |
| **SIGSEGV** | Bad memory access | "Segmentation fault — you crashed" |
| **SIGTERM** | `kill <pid>` | "Polite termination — please clean up and exit" |
| **SIGHUP** | Terminal disconnects | "Hangup — your terminal is gone" |
| **SIGKILL** | `kill -9 <pid>` | "Die NOW — no cleanup allowed" |

POSIX defines a whole **dictionary** of these signals. Processes can **catch and handle** most of them however they like — or just let the **default behavior** happen. The exceptions are `SIGKILL` and `SIGSTOP`, which cannot be caught, blocked, or ignored (the kernel enforces these directly).

> 💡 We covered signals in depth in [Day 2 — Getting Hands-On](../02-getting-hands-on-linux/README.md). POSIX is *why* those signals have standard names and numbers across every Unix-like system.
---

## 7️⃣ POSIX Threads (Pthreads)

In the 1990s, **POSIX threads (Pthreads)** changed computing. A **thread** is like a lightweight process — it shares memory with other threads in the same program.

Before Pthreads, multithreaded programming was the **wild west** of incompatible approaches. Every OS did it differently. POSIX defined the standard:

- How to **create** threads (`pthread_create`)
- How to **synchronize** them with **mutexes** and **semaphores**
- How to **destroy** them cleanly

Today, nearly every multithreaded C or C++ program on Linux and macOS still uses the Pthreads API. It's that fundamental.

> 🧭 **DevOps angle:** When you see a multi-threaded server (nginx worker processes, Java app threads, Go goroutines — though Go does its own thing), Pthreads is often underneath. And as we'll see in the Processes note, **threads are just processes created with shared-memory flags** via `clone`.

---

## 8️⃣ The Shell & Utilities

POSIX doesn't just cover the kernel interface — it also defines the **shell** and **utilities**.

### 🐚 The POSIX Shell

Based on the original **Bourne shell** (`sh`), POSIX specifies how shell scripts should behave: variables, loops, conditionals, functions — all defined. This is why a POSIX-compliant shell script written 30 years ago can still run today.

### 🛠️ Standard Utilities

POSIX standardizes dozens of command-line tools:

| Tool | Purpose |
|---|---|
| `ls` | List directory contents |
| `cat` | Concatenate/print files |
| `grep` | Search text |
| `awk` | Text processing language |
| `sed` | Stream editor |
| `find` | Find files |
| `sort` | Sort lines |
| `echo` | Print text |

> This is why `ls` works the same on macOS, Linux, and BSD — they all implement the POSIX spec for `ls`.

---

## 9️⃣ File Permissions — The RWX Model

POSIX defines the classic **permission model**. Every file and directory has:

- An **owner** (a user)
- A **group** (a group of users)
- Permissions for **three categories**: owner, group members, everyone else

And for each category, you can grant three permissions: **read (r)**, **write (w)**, **execute (x)**.

```
   -rwxr-xr-x
   │├─┤├─┤├─┤
   │ │  │  │
   │ │  │  └── others:     r-x  (read + execute)
   │ │  └───── group:      r-x  (read + execute)
   │ └──────── owner:      rwx  (read + write + execute)
   └────────── file type:  -    (regular file; d = directory)
```

This is the classic **rwx** system — those mysterious strings you see when you run `ls -l`. POSIX defines exactly what these mean and how they're enforced.

---

## 🔟 Environment Variables

Every process inherits a set of **key-value pairs** from its parent — these are **environment variables**.

| Variable | What it tells programs |
|---|---|
| `PATH` | Where the shell should look for executables |
| `HOME` | Where your home directory is |
| `USER` | Your username |
| `SHELL` | Which shell you're using |
| `LANG` | Your language/locale |

When you type a command, the shell looks through every directory in `PATH` to find the right executable. This elegant mechanism is completely defined by POSIX.

```bash
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

$ PATH="/my/custom/bin:$PATH"   # prepend a custom directory
```

---

## 1️⃣1️⃣ Who's Actually POSIX-Compliant?

Here's a surprise: **Linux is not officially POSIX-certified** — but it follows the standard very closely in practice.

| OS | POSIX compliance |
|---|---|
| **macOS** | ✅ **Fully certified** (one of the few that passed the official test suite) |
| **Linux** | ⚠️ Not officially certified (mainly due to certification *cost*), but follows POSIX closely |
| **FreeBSD / OpenBSD / NetBSD** | ✅ Highly POSIX compatible |
| **Android** | ✅ Inherits most POSIX behavior (runs on Linux) |
| **Windows** | ❌ Historically not POSIX-compatible — but **WSL** changed this |

### 🪟 The Windows Story — WSL

Windows was built on a completely different philosophy and was historically not POSIX compatible. Microsoft included a minimal POSIX subsystem in older Windows versions, but it was barely functional.

The **game changer** was **WSL (Windows Subsystem for Linux)**, launched in **2016**. WSL lets you run a real Linux environment inside Windows. Suddenly, developers on Windows could run POSIX-compliant tools natively. **WSL 2** even ships a **full Linux kernel**.

> It's Microsoft's biggest acknowledgement that POSIX compatibility matters.

---

## 🌍 Why POSIX Still Matters Today

| Domain | Why POSIX matters |
|---|---|
| **Cloud computing** | Built on Linux servers. Linux follows POSIX. Code using standard file I/O, processes, signals, pipes runs on every major cloud without modification. |
| **DevOps** | Docker, Kubernetes, and shell scripts all rely on POSIX behavior. |
| **Open-source** | Apache, Nginx, PostgreSQL, Redis — all written assuming POSIX. |
| **Portability** | Write once, run on macOS / Linux / BSD / WSL. |

> Learn POSIX concepts and you understand the foundation of the modern internet.

There's also a beautiful philosophical lesson in POSIX — **the Unix philosophy**:

> *Do one thing and do it well. Write programs that work together. Use text streams as the universal interface.*

POSIX bakes this philosophy into a standard. `grep` doesn't know about `sort`. `sort` doesn't know about `head`. But pipe them together and they solve real problems. This **composability** — small tools that cooperate — is one of the most durable ideas in all of software engineering.

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **POSIX** | Portable Operating System Interface — the contract defining what a Unix-like OS must provide. |
| **Unix wars** | 1980s chaos: AIX, HP-UX, SunOS all incompatible. Code broke across systems. |
| **IEEE 1988** | POSIX published — ended the portability nightmare. |
| **What it covers** | File system, processes, threads, I/O, signals, shell, utilities, permissions, env vars. |
| **Interface, not internals** | POSIX defines the *outlet shape*, not the wiring. Kernels implement it however they want. |
| **Everything is a file** | Hard drives, sockets, keyboards, processes — all accessed via `open`/`read`/`write`/`close`. |
| **File descriptors** | Small integers (ticket stubs). 0=stdin, 1=stdout, 2=stderr, 3+=opened files. |
| **Standard streams** | stdin (input), stdout (output), stderr (errors) — separable and redirectable. |
| **Pipes** | Two file descriptors (read end + write end) connecting one program's output to another's input. |
| **Signals** | Tiny notifications: SIGINT (Ctrl+C), SIGTERM (polite), SIGKILL (nuclear), SIGSEGV (crash). |
| **Pthreads** | POSIX threads (1990s) — standard API for multithreaded programming with mutexes/semaphores. |
| **Shell & utilities** | POSIX defines `sh` behavior + `ls`, `cat`, `grep`, `awk`, `sed`, `find`, `sort`, `echo`. |
| **rwx permissions** | Owner/group/others × read/write/execute. Shown in `ls -l` output. |
| **Environment variables** | Key-value pairs inherited from parent: `PATH`, `HOME`, `USER`, `SHELL`. |
| **macOS** | Fully POSIX-certified (rare — passed the official test suite). |
| **Linux** | Not officially certified (cost), but follows POSIX closely in practice. |
| **WSL** | Microsoft's 2016 admission that POSIX matters — runs a real Linux kernel. |
| **Why it still matters** | Cloud, DevOps (Docker/K8s), and all open-source software assume POSIX. |

---

← Back to [How Apps Talk to the Linux Kernel](./README.md) | Next: [System Calls — The Bridge](./2-system-calls-the-bridge.md) →
