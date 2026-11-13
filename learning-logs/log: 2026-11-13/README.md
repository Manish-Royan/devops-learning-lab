# Learning Log — 13 November 2026 🐧 (Linux Shell Scripting)

## 🎯 Today's Learning Objective

Understand how shell scripting works and how Bash can be used to automate repetitive Linux tasks.

Until now, most of my Linux interactions involved executing commands manually through the terminal. Today, I explored how multiple commands can be combined into scripts that can be executed whenever needed.

## 📚 Topics Explored

### 1. Shell Fundamentals
- What a shell is and how it interacts with the operating system.
- Common shells: Bash, sh, Zsh, Ksh, Csh, and Fish.
- Checking the current shell and inspecting `/etc/shells`.
- Understanding `/etc/os-release`.

### 2. Creating and Executing Shell Scripts
- Creating Bash scripts.
- Understanding the shebang (`#!/bin/bash`).
- Making scripts executable using `chmod +x`.
- Executing scripts through relative paths, absolute paths, and the Bash interpreter.
- Understanding the relationship between file permissions and script execution.

### 3. `source` vs. `bash`
- Executing a script in a separate Bash process.
- Running a script in the current shell using `source` or `.`.
- Understanding why changes such as `cd` persist when a script is sourced but not when executed in a separate process.

### 4. Variables and Constants
- Declaring and accessing variables.
- Understanding variable assignment syntax.
- Command substitution using `$(...)`.
- Using `readonly` to prevent reassignment.

### 5. Special Variables and User Input
- Understanding `$0`, `$1`, `$2`, `$@`, `$#`, `$$`, and `$?`.
- Passing command-line arguments to scripts.
- Reading interactive input using `read`.

### 6. Arrays and String Manipulation
- Creating and accessing indexed arrays.
- Understanding associative arrays.
- Iterating over array elements.
- String length, uppercase, lowercase, replacement, and substring operations.

### 7. Arithmetic Operations
- Using `let`.
- Performing arithmetic with `$((...))`.
- Understanding integer division and modulo.
- Generating simple random numbers with `$RANDOM`.

### 8. Conditional Statements
- Understanding `if`, `elif`, and `else`.
- Exploring `[ ]` and the `test` command.
- String, numeric, and file-condition checks.
- Using `case` statements for multiple matching conditions.

### 9. Loops
- `for` loops.
- `while` loops.
- Iterating over arrays, ranges, and files.
- Understanding how loops help automate repetitive operations.

### 10. PATH and Personal Command Directories
- Understanding the `PATH` environment variable.
- How the shell searches for executable commands.
- Why `./` is used to execute a script from the current directory.
- Adding a personal scripts directory to `PATH`.

## 💡 My Biggest Takeaway

The most interesting part of today's learning was understanding that a shell script is more than a collection of commands.

By introducing variables, arguments, conditions, arrays, and loops, we can make scripts respond to different situations instead of executing the same fixed sequence of commands every time.

This is where the real power of automation begins.

## 🎯 Connection to DevOps

Shell scripting is an important skill for working with Linux-based infrastructure.

It can help automate tasks such as:

- Application deployment.
- Backup operations.
- Log processing.
- Environment setup.
- Repetitive system administration tasks.
- Scheduled jobs and maintenance operations.

I am beginning to see how the Linux fundamentals I've studied over the previous days connect to practical automation.

## 🔭 What I Want to Explore Next

- Bash Scripting.
- Script exit codes and error handling.
- Command chaining and conditional execution.
- Safer scripting practices.
- Practical automation exercises.


## 📖 Detailed Technical Notes

Today's exploration is documented in the following technical chapter:

➡️ [ Linux Shell Scripting](../../notes/linux/09-shell-scripting/README.md)

The chapter includes detailed explanations, command examples, permission tables, diagrams, and practical DevOps scenarios.

---

**Learning in Public | Understanding from Commands to Automation - One practical step at a time.** 🚀