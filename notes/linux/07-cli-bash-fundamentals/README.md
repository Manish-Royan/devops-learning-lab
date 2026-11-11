# 🐧 Linux Learning Journey: CLI & Bash Fundamentals (Editors, Files, Streams & Pipes)

> **Goal:** Master the everyday CLI toolkit — text editors (Nano & Vim), file viewing/management, archiving, brace expansion & wildcards, I/O streams & redirection, and pipelines — the skills you'll use in every DevOps session.

---

## 📍 Why This Matters for DevOps

Everything you do as a DevOps engineer — editing configs on a remote server, inspecting logs, compressing artifacts for deployment, chaining commands in a pipeline — runs through the command line. There's no GUI on a production server. There's no "open in TextEdit." There's just you, a shell, and the toolkit in this note.

By the end of today, you'll be able to:
- Edit any file on any server (Nano for quick fixes, Vim for power)
- View logs of any size efficiently (`less`, `tail -f`)
- Create, copy, move, and (carefully!) delete files and directories
- Bundle and compress artifacts with `tar`
- Generate filenames in bulk with brace expansion
- Match files precisely with wildcards
- Redirect any stream (`>`, `>>`, `2>`, `2>/dev/null`, `<`)
- Chain commands into pipelines (`|`)

Let's go.

---

## 1️⃣ Text Editors — Nano vs Vim

You'll need to edit files on servers constantly — config files, scripts, cron jobs. Two editors dominate Linux: **Nano** (simple) and **Vim** (powerful).

### 🟢 GNU Nano — The Simple Editor

A lightweight, beginner-friendly command-line editor. No modes, no learning curve — just open and type, like any normal editor. Great for quick fixes and embedded systems.

| Action | Shortcut |
|---|---|
| **Open/Create** | `nano filename.txt` |
| **Save (Write Out)** | `Ctrl + O` → press `Enter` |
| **Exit** | `Ctrl + X` |
| **Search** | `Ctrl + W` |
| **Cut line** | `Ctrl + K` |
| **Paste** | `Ctrl + U` |

```
   ┌─────────────────────────────────────────────┐
   │  GNU nano 6.2              filename.txt     │
   ├─────────────────────────────────────────────┤
   │  Just type here. No modes.                  │
   │  Cursor moves with arrow keys.              │
   │  Save: Ctrl+O, Enter. Exit: Ctrl+X.         │
   │                                             │
   ├─────────────────────────────────────────────┤
   │ ^G Help   ^O Write Out  ^W Where Is  ^K Cut │
   │ ^X Exit   ^R Read File  ^\ Replace  ^U Paste│
   └─────────────────────────────────────────────┘
```

> 🧭 **DevOps tip:** Nano is perfect for "SSH in, fix one line in nginx.conf, get out" workflows. Don't overthink it — if you just need to change a port number, use Nano.

### 🟣 Vim (Vi Improved) — The Modal Powerhouse

Vim is a **modal editor** — meaning what your keystrokes do *depends on which mode you're in*. This is the part that confuses beginners, but it's also what makes Vim insanely efficient once it clicks.

#### 📜 A Brief History of Vim

To understand why Vim is modal, you have to understand where it came from.

**Line editors** were the original text editors. They worked one line at a time — you'd type a command like `:5` to go to line 5, then `s/old/new/` to substitute text on that line. Why? Because **memory was incredibly precious** in that era — you couldn't load an entire text file into memory. Output was often printed (literally, on paper), so you wanted to process line by line to save screen space. This was the era of **bits**, not kilobytes or megabytes.

```
   TIMELINE:
   ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
   │   ed     │───▶│   ex     │───▶│   vi     │───▶│   vim    │
   │ (line    │     │ (line    │     │ (visual  │     │ (Vi      │
   │  editor) │     │  editor) │     │  mode    │     │ IMproved)│
   │ 1970s    │     │          │     │  added)  │     │ 1991     │
   └──────────┘     └──────────┘     └──────────┘     └──────────┘
                                                       │
                                                       ▼
                                              Originally "Vi IMitation"
                                              Later renamed "Vi IMproved"
```

**Vim** was originally created as a **clone of vi** — the name stood for **"Vi IMitation"**. Later, as it grew more powerful, it was renamed to **"Vi IMproved"**. That's why the editor you use today is called Vim.

#### 🔀 The Two Modes You Must Know

Vim has multiple modes, but the two you'll use 95% of the time are:

| Mode | Purpose |
| :--- | :--- |
| **Normal Mode** | Navigate and execute editing commands |
| **Insert Mode** | Insert and edit text |
| **Command-line Mode** | Execute commands such as `:w`, `:q`, and `:wq` |

```
   ┌────────────────────────────────────────────────────────┐
   │                  VIM MODE SWITCHING                    │
   │                                                        │
   │    ┌─────────────┐         i           ┌─────────────┐ │
   │    │  COMMAND    │ ─────────────────▶ │   INSERT    │ │
   │    │   MODE      │                     │    MODE     │ │
   │    │ (default)   │ ◀───────────────── │             │ │
   │    └─────────────┘       Esc           └─────────────┘ │
   │                                                        │
   │  In Command Mode: move with h/j/k/l, delete with x,    │
   │  save with :w, quit with :q, save+quit with :wq        │
   │                                                        │
   │  In Insert Mode: type text normally, Esc to go back    │
   └────────────────────────────────────────────────────────┘
```

#### ⌨️ Essential Vim Commands

| Action | Keys |
|---|---|
| **Open** | `vim filename.txt` |
| **Enter Insert Mode** | Press `i` |
| **Return to Command Mode** | Press `Esc` |
| **Save** | `:w` |
| **Quit** | `:q` |
| **Save & Quit** | `:wq` |
| **Force Quit (discard changes)** | `:q!` or `:qa!` |
| **Move cursor** | `h` (left), `j` (down), `k` (up), `l` (right) |
| **Delete character** | `x` |
| **Delete line** | `dd` |
| **Copy (yank) line** | `yy` |
| **Paste** | `p` |
| **Search** | `/searchterm` then `n` for next |

> 💡 **The #1 Vim survival tip:** If you're stuck and don't know what mode you're in, **press `Esc` twice** to guarantee you're in Command Mode, then `:q!` to exit without saving. This is the "panic exit" — memorize it.

> 🧭 **DevOps angle:** Vim is on most Linux server, Docker container, rescue environment. Nano often isn't. If you can't escape Vim, you can't survive production. Learn at least: `i`, `Esc`, `:wq`, `:q!`. That's the minimum viable Vim.

---

## 2️⃣ Reading & Viewing Files

You'll spend a surprising amount of time reading files — logs, configs, scripts, output. Different tools for different jobs:

| Command | Syntax | Key Characteristics |
|---|---|---|
| **`less`** | `less textfile.txt` | Read-only viewer. Efficient for large files; loads content incrementally and allows smooth scrolling. **Preferred over `more`.** |
| **`cat`** | `cat textfile.txt` | Concatenates and prints full file contents directly to stdout. **Best suited for short files.** |
| **`tail`** | `tail textfile.txt` / `tail -n 3 textfile.txt` | Outputs the last 10 lines (or specified `-n` lines). |
| **`head`** | `head textfile.txt` / `head -n 3 textfile.txt` | Outputs the first 10 lines (or specified `-n` lines). |
| **`tail -f`** | `tail -f textfile.txt` | Monitors files/logs **in real time** as new lines are appended. Exit with `Ctrl + C`. |
| **`man`** | `man command` (or `command --help`) | Displays system manual pages. `--help` gives a quick brief summary. |

### 📊 When to Use What

```
   ┌────────────────────────────────────────────────────────────┐
   │  FILE SIZE / USE CASE        USE THIS                      │
   ├────────────────────────────────────────────────────────────┤
   │  Tiny file (< 1 screen)      cat file.txt                  │
   │  Medium/large file           less file.txt                 │
   │  First N lines               head -n 20 file.txt           │
   │  Last N lines                tail -n 20 file.txt           │
   │  Live log monitoring         tail -f /var/log/syslog       │
   │  How does this command work? man grep  (or grep --help)    │
   └────────────────────────────────────────────────────────────┘
```

### 🔍 `less` — The Power Viewer

`less` is the gold standard for reading large files because it **loads incrementally** — it doesn't read the whole file into memory, so it opens instantly even on multi-gigabyte logs.

```
   $ less /var/log/syslog
   # Inside less:
   #   /searchterm    → search forward
   #   ?searchterm    → search backward
   #   n              → next match
   #   N              → previous match
   #   g              → go to top
   #   G              → go to bottom
   #   q              → quit
```

### 📋 `cat` — The Quick Dump

`cat` dumps the entire file to stdout. Fine for small files, terrible for large ones — it'll flood your terminal with thousands of lines you can't scroll back through. For anything longer than a screen, use `less`.

### ⏱️ `tail -f` — The DevOps Essential

`tail -f` (follow) is your **live log viewer**. It shows the last lines of a file and keeps watching — when new lines are appended, they appear on screen automatically.

```bash
# Watch nginx access logs live
$ tail -f /var/log/nginx/access.log

# Watch a Docker container's logs live
$ docker logs -f my-container

# Watch syslog and filter for errors in real time
$ tail -f /var/log/syslog | grep "ERROR"
```

> 🧭 **DevOps tip:** `tail -f` + `grep` is the bread-and-butter debugging pattern. When you deploy an app and want to watch it start up, `tail -f /var/log/myapp.log | grep -i error` catches problems the moment they happen.

---

## 3️⃣ Directory & File Management

### 📁 Creating Directories & Files

**`mkdir`** — Creates new folders. Use `-p` to create nested path structures at once (no errors if parent doesn't exist):

```bash
mkdir myfolder                           # single directory
mkdir -p path/to/nested/folder           # creates all intermediate dirs at once
```

**`touch`** — Creates empty files or updates file modification timestamps without modifying content:

```bash
touch file1.txt file2.txt file3.txt      # creates three empty files
touch existing.txt                        # updates its timestamp (doesn't change content)
```

### 📋 Copying, Moving & Renaming

**`cp`** (copy) — Copies files or directories. **Preserves the original.**

```bash
cp file.txt destination.txt              # copy a file
cp -R dir1/ dir2/                         # recursive copy (copy a whole directory)
```

**`mv`** (move) — Moves or renames files and directories. **Removes from the original location.**

```bash
mv oldname.txt newname.txt               # rename a file
mv file.txt /target/dir/                 # move a file to another directory
```

### 🆚 `cp` vs `mv` — The Critical Difference

```
   cp (COPY) — original preserved:
   ┌──────────┐          ┌──────────┐
   │ file.txt │ ───────▶│ file.txt │   ← copy created here
   │ (source) │          │ (dest)   │
   └──────────┘          └──────────┘
        │
        └── still here ✅

   mv (MOVE) — original removed:
   ┌──────────┐          ┌──────────┐
   │ file.txt │ ───────▶│ file.txt │   ← moved here
   │ (source) │          │ (dest)   │
   └──────────┘          └──────────┘
        │
        └── gone ❌
```

> `cp` creates a copy while keeping the original. `mv` moves or renames and **removes the file from its original location**. Use `cp` when you want two copies; use `mv` when you want to relocate or rename.

### ⚠️ `rm` — The Danger Zone

**Deletions in Linux CLI are permanent.** There is no trash bin, no recycle bin, no undo. `rm` doesn't move files to trash — it **unlinks them from the filesystem**, and the data is gone.

```
   ┌──────────────────────────────────────────────────────────────┐
   │                    ⚠️  RM DANGER LEVELS  ⚠️                  │
   │                                                              │
   │   rm file.txt              → deletes one file                │
   │   rm -r folder/            → deletes directory recursively   │
   │   rm -rf folder/           → force recursive, no prompts     │
   │   rm -i file*              → interactive, asks y/n each      │
   │                                                              │
   │   ☠️  NEVER RUN:                                             │
   │   rm -rf *                  → deletes everything in cwd      │
   │   rm -rf /                  → deletes the ENTIRE SYSTEM      │
   │                                                              │
   │   "rm -rf * or rm -rf / can delete the entire system."       │
   └──────────────────────────────────────────────────────────────┘
```

| Command | What it does |
|---|---|
| **`rm file.txt`** | Removes a file permanently |
| **`rm -r folder/`** | Recursively deletes a directory and all its contents |
| **`rm -rf folder/`** | Force recursive deletion — **no confirmation prompts** |
| **`rm -i file*`** | Interactive mode — prompts for y/n confirmation before each deletion |

> 🚨 **Safety rule:** Always review your target before pressing Enter on `rm -rf`. Double-check you're in the right directory (`pwd`). Consider using `rm -i` for bulk deletions — the extra "y/n" prompts have saved many careers.

> 🧭 **DevOps horror story:** Someone runs `rm -rf $VAR/*` where `$VAR` is unexpectedly empty — the command becomes `rm -rf /*` and deletes the entire system. Always quote variables: `rm -rf "$VAR"/*` and check `$VAR` is set first. This has happened at real companies.

---

## 4️⃣ Archiving & Compression (`tar`)

Unlike `zip` tools (which compress and archive in one step), **`tar` (Tape Archive)** bundles multiple files/directories into a **single contiguous file**, which can then *optionally* be compressed using `gzip`.

### 📦 The `tar` Command Reference

| Operation | Command Syntax | Flag Breakdown |
|---|---|---|
| **Create archive** | `tar -cf archive.tar file1.txt folder1/` | `-c`: Create, `-f`: output filename |
| **Create compressed (.tar.gz)** | `tar -czf archive.tar.gz file1.txt folder1/` | `-z`: compress with gzip |
| **Extract compressed archive** | `tar -xzf archive.tar.gz -C /destination/` | `-x`: Extract, `-C`: target directory |

### 🔧 Flag Meanings

| Flag | Meaning |
|---|---|
| **`-c`** | **C**reate a new archive |
| **`-x`** | E**x**tract from an archive |
| **`-f`** | Specify the **f**ilename |
| **`-z`** | Compress/decompress with g**z**ip |
| **`-v`** | **V**erbose — show files as they're processed |
| **`-C`** | Change to **C** directory before extracting |
| **`-t`** | Lis**t** contents without extracting |

### 📊 Create vs Extract Flow

```
   CREATE (bundle + compress):
   ┌─────────┐  ┌─────────┐
   │ file1   │  │ folder/ │
   └────┬────┘  └────┬────┘
        │            │
        └─────┬──────┘
              ▼  tar -czf archive.tar.gz file1 folder/
   ┌───────────────────┐
   │  archive.tar.gz   │  ← single compressed file
   └───────────────────┘

   EXTRACT (unbundle + decompress):
   ┌───────────────────┐
   │  archive.tar.gz   │
   └─────────┬─────────┘
             ▼  tar -xzf archive.tar.gz -C /destination/
   ┌─────────┐  ┌─────────┐
   │ file1   │  │ folder/ │   ← extracted to /destination/
   └─────────┘  └─────────┘
```

### 📋 Common Patterns

```bash
# Create a compressed backup of a directory
tar -czf backup-$(date +%Y%m%d).tar.gz /home/paul/projects/

# Extract a downloaded tarball
tar -xzf node-v20.0.0.tar.gz -C /opt/

# List contents without extracting (peek inside)
tar -tzf archive.tar.gz

# Add verbose to see what's happening
tar -czvf archive.tar.gz folder/    # shows each file as it's added
```

> 🧭 **DevOps use case:** `tar` is how you bundle application artifacts for deployment. `tar -czf app.tar.gz dist/` → ship the single file → `tar -xzf app.tar.gz -C /var/www/` on the server. It's also how most open-source software is distributed (download a `.tar.gz`, extract, `./configure && make && make install`).

---

## 5️⃣ Brace Expansion & Wildcards

Bash evaluates **brace expansion** and **wildcards** *before* handing arguments over to the specified program. The program never sees the braces or wildcards — it just receives the expanded list of filenames.

###  Brace Expansion `{}`

Generates **arbitrary string permutations** automatically — it doesn't matter if the files exist or not.

```bash
# Create a list of numbered files
touch file{1..5}.txt
# creates: file1.txt file2.txt file3.txt file4.txt file5.txt

# Alphabetical ranges
echo {a..e}
# Output: a b c d e

# Numeric ranges with steps
echo {0..100..20}
# Output: 0 20 40 60 80 100

# Permutation sets (combine two ranges)
echo {a..b}{1..3}
# Output: a1 a2 a3 b1 b2 b3
```

```
   ┌──────────────────────────────────────────────────────────────┐
   │  BRACE EXPANSION GENERATES STRINGS (files need not exist):   │
   │                                                              │
   │  {1..5}        →  1 2 3 4 5                                  │
   │  {a..e}        →  a b c d e                                  │
   │  {0..100..20}  →  0 20 40 60 80 100                          │
   │  {a..b}{1..3}  →  a1 a2 a3 b1 b2 b3                          │
   │                                                              │
   │  Use case: create many files/dirs at once                    │
   │  mkdir -p /var/log/{nginx,apache,postgres}                   │
   │  → creates three directories in one command                  │
   └──────────────────────────────────────────────────────────────┘
```

### 🃏 Wildcard Matching

Matches **existing filenames** in the directory tree. Unlike brace expansion, wildcards only match files that actually exist.

| Pattern | Matches |
|---|---|
| **`*`** | **Zero or more** characters |
| **`?`** | **Exactly one** character |
| **`\`** | Escapes spaces or special characters |

### 🆚 `*` vs `?` — The Key Distinction

```bash
# * matches zero or more characters
ls file*.txt
# matches: file.txt, file1.txt, file10.txt, file_test.txt

# ? matches exactly ONE character
ls file?.txt
# matches: file1.txt (one char between "file" and ".txt")
# does NOT match: file.txt (zero chars) or file10.txt (two chars)
```

```
   ┌──────────────────────────────────────────────────────────────┐
   │  WILDCARD MATCHING (only matches EXISTING files):            │
   │                                                              │
   │  Pattern: file*.txt                                          │
   │  ✅ file.txt      (zero chars between "file" and ".txt")     │
   │  ✅ file1.txt     (one char)                                 │
   │  ✅ file10.txt    (two chars)                                │
   │  ✅ file_test.txt (many chars)                               │
   │                                                              │
   │  Pattern: file?.txt                                          │
   │  ❌ file.txt      (zero chars — ? needs exactly one)         │
   │  ✅ file1.txt     (one char)                                 │
   │  ❌ file10.txt    (two chars — ? needs exactly one)          │
   └──────────────────────────────────────────────────────────────┘
```

### 🔧 Escaping Special Characters

If a filename contains a space or special character, escape it with `\`:

```bash
touch file\ name.txt              # creates a file named "file name.txt"
ls file\ name.txt                 # lists it
rm "file name.txt"                # or quote the whole name (often easier)
```

> 🧭 **DevOps tip:** When writing scripts, **always quote variables that might contain spaces**: `rm "$filename"` not `rm $filename`. Without quotes, a filename like `my file.txt` becomes two arguments (`my` and `file.txt`), and `rm` happily deletes both — if they exist.

---

## 6️⃣ Input/Output Streams & Redirection

Per the Unix philosophy, programs handle data flow via **three standard I/O streams**, each identified by an integer:

```
   ┌──────────────────────────────────────────────────────────────┐
   │                   THE THREE STANDARD STREAMS                 │
   │                                                              │
   │   0 — stdin  (Standard Input)                                │
   │       │  Data input stream                                   │
   │       │  Default: keyboard (or redirected source file)       │
   │       │                                                      │
   │   1 — stdout (Standard Output)                               │
   │       │  Default output destination                          │
   │       │  Default: terminal screen                            │
   │       │                                                      │
   │   2 — stderr (Standard Error)                                │
   │          Error logging destination                           │
   │          Default: terminal screen (but separate from stdout) │
   └──────────────────────────────────────────────────────────────┘
```

| fd | Name | Default | Purpose |
|---|---|---|---|
| **0** | `stdin` | Keyboard | Where a program reads input |
| **1** | `stdout` | Screen | Where a program writes normal output |
| **2** | `stderr` | Screen | Where a program writes error messages |

### 🔄 Redirection Operators

| Code | Example | Behavior |
|---|---|---|
| **`>` or `1>`** | `echo "Hello" > file.txt` | Redirects **stdout** to file (**overwrites** existing content) |
| **`>>` or `1>>`** | `echo "Hello" >> file.txt` | Redirects **stdout** to file (**appends** to end) |
| **`2>`** | `cat nonexistent.txt 2> error.log` | Redirects **stderr** to a designated log file |
| **`2> /dev/null`** | `ls /root 2> /dev/null` | **Discards** error messages completely (sends to the bit bucket) |
| **`<`** | `grep "pattern" < input.txt` | Feeds file contents into the program's **stdin** |

### 📊 Redirection Patterns

```
   ┌──────────────────────────────────────────────────────────────┐
   │  REDIRECTION EXAMPLES:                                       │
   │                                                              │
   │  echo "Hello" > file.txt                                     │
   │       stdout ──▶ file.txt (overwrite)                       │
   │                                                              │
   │  echo "World" >> file.txt                                    │
   │       stdout ──▶ file.txt (append)                          │
   │       file.txt now contains:                                 │
   │         Hello                                                │
   │         World                                                │
   │                                                              │
   │  cat nonexistent.txt 2> error.log                            │
   │       stderr ──▶ error.log                                  │
   │                                                              │
   │  ls /root 2> /dev/null                                       │
   │       stderr ──▶ /dev/null (discarded — bit bucket)         │
   │                                                              │
   │  grep "error" < input.txt                                    │
   │       input.txt ──▶ stdin of grep                           │
   └──────────────────────────────────────────────────────────────┘
```

### 🎯 Combining Streams

The classic pattern — send stdout to a file, discard stderr:

```bash
# Save output, throw away errors
command > output.log 2> /dev/null

# Redirect stdout to all.log, then redirect stderr
# to the same destination as stdout.
command > all.log 2>&1

# Redirect stderr to the ORIGINAL stdout first,
# then redirect stdout to all.log.
command 2>&1 > all.log
```

> 🧭 **DevOps tip:** `2> /dev/null` is how you silence permission-denied errors when searching the filesystem: `find / -name "*.conf" 2> /dev/null`. Without it, you'd get dozens of "Permission denied" lines cluttering your results.

---

## 7️⃣ Pipelines (`|`) & Process Control

The **pipe operator (`|`)** connects the **stdout of one program directly into the stdin of another**. This is the Unix philosophy made literal — small tools that do one thing well, composed into powerful pipelines.

```
   ┌──────────┐    stdout     ┌──────────┐    stdout     ┌──────────┐
   │ program A│ ──[fd 1]───▶ │ program B│ ──[fd 1]───▶ │ program C│ ──▶ screen
   │ (ps aux) │     │ pipe    │ (grep)   │     │ pipe    │ (head)   │
   └──────────┘     │         └──────────┘     │         └──────────┘
                    ▼                          ▼
              program B reads           program C reads
              A's output as stdin       B's output as stdin
```

### 📋 Common Pipeline Patterns

```bash
# Filter running processes using ps and grep
ps aux | grep "python"

# Automated confirmation using 'yes'
yes | rm -i file*          # feeds 'y' continuously into rm's prompt

# Feed 'n' continuously (decline everything)
yes n | rm -i file*

# Background process control
yes > /dev/null &          # run in background ('&')
kill -9 <PID>              # force kill by ID
```

### 🔍 How a Pipe Works — Step by Step

When you run `cat ls.txt | grep "ls error.txt"`:

```
   STEP 1: cat ls.txt starts, reads the file, writes to stdout
           ┌──────────┐
           │ cat      │ ──stdout──▶ (pipe)
           │ ls.txt   │
           └──────────┘

   STEP 2: grep reads from stdin (which is the pipe), searches for "ls error.txt"
           ┌──────────┐
           │ grep     │ ◀──stdin── (pipe)
           │"ls err.."│
           └──────────┘
               │
               ▼
           stdout (matching lines printed to screen)
```

The pipe connects `cat`'s **stdout** to `grep`'s **stdin**. `grep` searches for the pattern and outputs matching lines.

### ⚠️ Pipeline Exit Status — `pipefail`

By default, Bash returns the exit status of the last command in a pipeline.

```bash
cat missing.txt | grep "Linux"
echo $?
```

The first command can fail, while the second command returns successfully.

For more reliable automation, enable:

```bash
set -o pipefail
```

Now a failure in an earlier pipeline command can cause the whole pipeline to return a non-zero status.
> This is particularly useful in CI/CD shell scripts.



### 🎯 `grep` Is Smart About Pipes

Here's a subtle and brilliant detail: **`grep` detects whether its output is going to a terminal or being piped to another program.**

```
   ┌──────────────────────────────────────────────────────────────┐
   │  grep OUTPUT BEHAVIOR:                                       │
   │                                                              │
   │  Output to TERMINAL (human):                                 │
   │  $ grep "error" log.txt                                      │
   │  → matches shown WITH COLOR highlighting                     │
   │  → human can read it easily                                  │
   │                                                              │
   │  Output PIPED to another program (machine):                  │
   │  $ grep "error" log.txt | wc -l                              │
   │  → matches shown WITHOUT COLOR                               │
   │  → clean text for the next program to parse                  │
   │                                                              │
   │  Why? Color codes (ANSI escape sequences) would corrupt      │
   │  machine parsing. grep is smart enough to strip them when    │
   │  it detects the output isn't going directly to a terminal.   │
   └──────────────────────────────────────────────────────────────┘
```

This is why `grep "error" log.txt` looks colorful on your screen, but `grep "error" log.txt | wc -l` gives a clean count — `grep` removed the color codes because it knew the output was being piped.

### 📊 Building Complex Pipelines

```bash
# Find the top 5 memory-consuming processes
ps aux | sort -rnk 4 | head -5

# Count how many times each IP hit your nginx server
cat /var/log/nginx/access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10

# Find all .conf files, excluding permission-denied errors, show their sizes
find / -type f -name "*.conf" -print0 2>/dev/null | xargs -0 du -h
```

Each command does one thing well — the pipe composes them into something powerful. This is the Unix philosophy from Day 1, now in action.

### 🔄 Background Processes with `&` and `kill`

```bash
# Run a process in the background
yes > /dev/null &
[1] 12345                    # job ID [1], PID 12345

# Find it later
ps aux | grep yes

# Kill it by PID
kill -9 12345                # SIGKILL (force kill, no cleanup)

# Or by job ID
kill %1                      # kills job [1]
```

> 🧭 **DevOps reminder:** Recall from Day 2 — `kill -9` is SIGKILL (no cleanup). Always try `kill -15` (SIGTERM) first to let the process clean up gracefully. Use `-9` only as a last resort when a process is truly hung.

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **Nano** | Simple, beginner-friendly editor. `Ctrl+O` save, `Ctrl+X` exit. Good for quick fixes. |
| **Vim** | Modal power editor. On every server. Press `i` for Insert, `Esc` for Command, `:wq` save+quit, `:q!` force quit. |
| **Vim history** | Born from line editors (memory was precious — bits not MB). Originally "Vi IMitation", renamed "Vi IMproved". |
| **Vim modes** | Command (default, run commands) and Insert (type text). `Esc` → Command, `i` → Insert. |
| **`less`** | Best viewer for large files — loads incrementally, search with `/`. |
| **`cat`** | Dumps full file to stdout. Only for short files. |
| **`head`** | First N lines. `head -n 20 file.txt`. |
| **`tail`** | Last N lines. `tail -n 20 file.txt`. |
| **`tail -f`** | Live log monitoring. `Ctrl+C` to exit. DevOps essential. |
| **`man`** | System manual. `man grep` or `grep --help` for quick summary. |
| **`mkdir -p`** | Create nested directories at once. |
| **`touch`** | Create empty files or update timestamps. |
| **`cp`** | Copy — preserves original. `-R` for recursive (directories). |
| **`mv`** | Move/rename — removes from original location. |
| **`rm`** | Permanent deletion — no trash bin. Always review targets. |
| **`rm -rf`** | Force recursive — no prompts. ☠️ DANGEROUS. Never `rm -rf /` or `rm -rf *` carelessly. |
| **`rm -i`** | Interactive — asks y/n before each deletion. Safer for bulk deletes. |
| **`tar -czf`** | Create compressed archive (.tar.gz). `-c` create, `-z` gzip, `-f` filename. |
| **`tar -xzf`** | Extract compressed archive. `-x` extract, `-C` target directory. |
| **`tar -tzf`** | List contents without extracting. |
| **Brace expansion `{}`** | Generate arbitrary strings (files need not exist). `touch file{1..5}.txt`. |
| **`*` wildcard** | Matches zero or more characters. `ls file*.txt`. |
| **`?` wildcard** | Matches exactly one character. `ls file?.txt`. |
| **`\` escape** | Escapes spaces/special chars in filenames. `touch file\ name.txt`. |
| **stdin (0)** | Standard input — default: keyboard. |
| **stdout (1)** | Standard output — default: terminal screen. |
| **stderr (2)** | Standard error — default: terminal screen (separate from stdout). |
| **`>`** | Redirect stdout to file (overwrite). |
| **`>>`** | Redirect stdout to file (append). |
| **`2>`** | Redirect stderr to file. |
| **`2> /dev/null`** | Discard stderr completely (bit bucket). |
| **`<`** | Feed file contents into stdin. |
| **`2>&1`** | Merge stderr into stdout (must come AFTER `>`). |
| **`\|` (pipe)** | Connect stdout of one program to stdin of another. Unix philosophy in action. |
| **`grep` + pipe** | `grep` auto-detects piped output and removes color codes for clean machine parsing. |
| **`&`** | Run a process in the background. |
| **`kill -9`** | Force kill by PID (SIGKILL). Use `kill -15` first if possible. |
| **DevOps relevance** | These are the daily tools — edit configs, read logs, bundle artifacts, chain commands, redirect output. Master them and you can operate any Linux server. |
