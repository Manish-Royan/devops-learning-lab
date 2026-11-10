# Learning Log — 10 November 2026 🐧

**Learning Focus:** Linux Internals — Boot Process, Kernel, System Calls, Memory & Process Management

**Learning Phase:** Linux Fundamentals → System Internals

---

## 📍 Today's Learning Experience

Today was different from my previous Linux learning sessions.

I initially intended to continue strengthening my Linux fundamentals, but as I started exploring how things actually work underneath the operating system, I became increasingly curious about its internals.

Instead of rushing through more commands, I dedicated the day to understanding the mechanisms behind Linux.

My exploration covered several interconnected areas, from the moment a machine boots to how applications communicate with the kernel and how the operating system manages processes and memory.

## 📚 What I Explored Today

### 1. The Linux Boot Process

Explored what happens between pressing the power button and reaching a login prompt.

- CPU initialization and the Reset Vector.
- BIOS vs UEFI firmware.
- POST (Power-On Self-Test).
- MBR and EFI System Partition (ESP).
- GRUB bootloader.
- Linux kernel and initramfs.
- Systemd, PID 1, services, and targets.
- Legacy initialization through SysVinit.

**The question I wanted to understand:** How does a computer go from powered off to a fully operational Linux environment?

### 2. POSIX — The Contract

Explored how POSIX helps standardize interfaces across Unix-like operating systems.

- The historical portability problem and the UNIX wars.
- What POSIX standardizes and what it doesn't.
- File descriptors (0, 1, 2).
- Standard input, standard output, and standard error.
- I/O redirection and pipes.
- Signals and POSIX threads.
- Shell utilities, permissions, and environment variables.

**The question I wanted to understand:** Why can many programs and shell scripts work across different Unix-like systems?

### 3. System Calls — The Bridge Between Applications and the Kernel

Explored how applications request services from the Linux kernel.

- User space vs kernel space.
- The privilege boundary.
- The system-call lifecycle.
- The role of C libraries such as glibc.
- Common system calls for files, processes, networking, and memory.
- Architecture-specific syscall numbers.
- Understanding `strace` as a system-call tracing tool.

**The question I wanted to understand:** What actually happens when an application needs to access a file, allocate memory, or communicate with the network?

### 4. Processes — The Runtime

Explored how Linux creates, executes, and manages processes.

- `task_struct` and process bookkeeping.
- `fork()` and process creation.
- `exec()` and replacing a process image.
- `clone()` and configurable resource sharing.
- Copy-on-Write (COW).
- Parent and child processes.
- Zombie and orphan processes.
- Threads and process sharing.
- The relationship between namespaces and containers.

**The question I wanted to understand:** What actually happens when I execute a command in the terminal?

### 5. Linux Memory Management

Explored memory management from both the application and kernel perspectives.

**The user-facing model:**

- RAM and page cache.
- Understanding `free` vs `available`.
- Virtual memory and memory isolation.
- Swap space and swappiness.
- Memory pressure and the OOM killer.
- `oom_score` and `oom_score_adj`.
- Huge pages and memory overcommit.

**The kernel internals:**

- Virtual-to-physical address translation.
- Memory Management Unit (MMU).
- Translation Lookaside Buffer (TLB).
- Page tables and page-table walks.
- Page faults.
- Virtual Memory Areas (VMAs).
- `struct page` and memory models.
- Compound pages and folios.

### 6. Disk and Memory Monitoring

Explored the essential Linux monitoring commands:

- `df` — filesystem disk-space usage.
- `du` — file and directory disk usage.
- `free` — memory and swap statistics.

### 7. Process Monitoring and Management

Explored how to observe and control running processes.

**Process inspection:**
- `ps` and its different options.
- `top` and real-time process monitoring.
- Understanding CPU and memory statistics.
- Reading process information.

**Process control and automation:**
- Exit codes and `$?`.
- Command chaining using `&&`, `||`, and `;`.
- Command substitution using `$()`.
- Background and foreground jobs.
- `jobs`, `bg`, and `fg`.
- Process priorities using `nice` and `renice`.
- `nohup` and handling terminal disconnection.

---

## 💡 My Biggest Takeaway

Today, I started connecting several concepts that initially appeared unrelated.

A command executed in the terminal is not simply an instruction that produces output.

It involves a shell, processes, libraries, system calls, and the Linux kernel.

The kernel is responsible for managing resources such as CPU, memory, filesystems, and hardware access, while user-space applications interact with it through controlled interfaces.

Understanding these relationships gives me a much deeper perspective on what happens underneath Linux commands and applications.

---

## 🎯 Why This Matters in My DevOps Journey

I don't want to become someone who only memorizes Linux commands or knows how to use Docker and Kubernetes without understanding the environment underneath them.

I want to understand:

- How a Linux machine boots.
- How processes are created and managed.
- How applications communicate with the kernel.
- How memory is allocated and managed.
- How to inspect system resources.
- How to investigate problems instead of blindly executing commands.

These are the foundations I want to build before moving further into infrastructure automation, containers, CI/CD, and cloud technologies.

---

## 🔍 What Comes Next?

My next step is to reinforce these concepts through hands-on Linux experiments.

Rather than only reading about system calls, processes, memory, and boot configurations, I want to gradually observe their behavior using actual Linux commands and troubleshooting exercises.

Understanding the theory is important, but connecting it with real system behavior is where the learning becomes meaningful.

---

## 📖 Detailed Technical Notes

I've organized today's exploration into separate technical chapters:

1. [Linux Boot Process — Power Button to Login](../../notes/linux/03-boot-process/README.md)
2. [How Applications Talk to the Linux Kernel](../../notes/linux/04-kernel-syscalls-and-processes/README.md)
3. [Linux Memory — From RAM to OOM Killer](../../notes/linux/05-memory-and-disk/README.md)
4. [Linux Process Management](../../notes/linux/06-process-management/README.md)

---

**Learning in Public | Going beyond commands to understand the system underneath.** 🚀