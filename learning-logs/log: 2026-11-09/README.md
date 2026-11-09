# Learning Log — 09 November 2026 🐧

**Learning Focus:** Getting Hands-On with Linux — Running Linux, Bash & the CLI

**Learning Phase:** Linux Fundamentals

---

## 📚 What I Explored Today

Yesterday, I focused on understanding what Linux is and how its ecosystem evolved.

Today, I moved toward understanding how to interact with Linux through the command line and how to manage the environment in which Linux runs.

My learning covered the following areas:

### 1. Running Linux Environments

- Understanding Virtual Machines and their relationship with host and guest operating systems.
- Exploring the differences between virtualization and emulation.
- Understanding the architectural differences between WSL1 and WSL2.
- Learning about Multipass as a convenient way to create and manage Ubuntu virtual machines.

### 2. Bash and the Command-Line Interface

- Understanding Bash (Bourne Again Shell) and its role as a command interpreter.
- Exploring the REPL (Read-Eval-Print Loop) interaction model.
- Understanding the UNIX/Linux philosophy of treating many system resources through file-based interfaces.

### 3. Filesystem Navigation

- Understanding the Linux root directory `/`.
- Learning the difference between absolute and relative paths.
- Understanding `pwd`, `.`, `..`, and `~`.
- Exploring Tab Completion for faster and more accurate command execution.

### 4. Linux Commands, Arguments & Flags

- Understanding how commands receive arguments and options.
- Exploring `ls`, `which`, and the `/bin` directory.
- Understanding short options (`-l`, `-a`) and long options (`--all`).
- Exploring hidden files and directories through the dot (`.`) prefix.
- Learning how to use `--help` to understand command syntax.

### 5. Bash History & Terminal Productivity

- Understanding Bash command history and `~/.bash_history`.
- Exploring reverse history search using `Ctrl + R`.
- Understanding the `!!` history expansion operator.
- Learning useful terminal shortcuts such as `Ctrl + L`.

### 6. Keyboard Shortcuts & the Kill Ring

- Learning command-line editing shortcuts.
- Understanding `Ctrl + A`, `Ctrl + E`, `Ctrl + K`, `Ctrl + U`, and `Ctrl + Y`.
- Exploring the difference between the terminal's system clipboard and Readline's kill ring.
- Understanding why `Ctrl + C` should not be confused with ordinary copy operations in the terminal.

### 7. Linux Signals & Process Management

- Understanding what signals are and how they communicate with running processes.
- Exploring the differences between `SIGINT`, `SIGQUIT`, `SIGTERM`, and `SIGKILL`.
- Understanding why graceful termination is preferable to immediately killing a process.
- Exploring commands such as `kill`, `kill -l`, and `ps`.

---

## 💡 My Key Takeaways

**1. The Command Line Is More Than a Place to Execute Commands**

Understanding Bash, command arguments, flags, and filesystem navigation makes the Linux environment much easier to operate.

**2. Learn the Keyboard, Not Just the Commands**

Shortcuts such as `Ctrl + R`, `Ctrl + A`, `Ctrl + E`, and `Ctrl + K` can significantly improve command-line productivity.

**3. Command History Requires Security Awareness**

The Bash history mechanism is useful for productivity, but sensitive information should never be placed directly in command arguments.

**4. Not All Process Termination Is the Same**

Understanding signals such as `SIGINT`, `SIGTERM`, and `SIGKILL helps explain how processes respond to interruption and shutdown requests.

In particular, graceful termination gives applications an opportunity to clean up resources before exiting.

---

## 🎯 Why This Matters in My DevOps Journey

DevOps engineers regularly interact with Linux environments through terminals, remote servers, and automation tools.

Understanding how to navigate filesystems, execute commands, inspect processes, and manage application termination is fundamental to administering Linux-based infrastructure.

These concepts will become especially important when working with services, Docker containers, CI/CD pipelines, and eventually Kubernetes.

---

## 🔍 What Comes Next?

Continue strengthening my command-line knowledge and filesystem understanding, while gradually moving from basic Linux interaction toward practical system administration and troubleshooting.

---

## 📖 Detailed Technical Notes

For the complete explanations, command examples, diagrams, comparisons, and technical breakdown:

➡️ [Getting Hands-On with Linux: Running Linux, Bash & the CLI](../../notes/linux/02-getting-hands-on-linux/README.md)

---

**Learning in Public | Understanding the fundamentals, one step at a time.** 🚀
