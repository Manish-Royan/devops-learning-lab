# ✂️ SED — The Stream Editor Scalpel

> **Goal:** Master SED — from the Pattern Space mental model to viewing/filtering lines (`p`/`-n`), substitution (`s/old/new/g`/`-i`), deletion (`d`), insertion (`a`/`i`/`c`), writing to files (`w`), and regex anchors with POSIX character classes.

---

## 📍 What Is SED?

**SED** stands for **Stream Editor**. It's a powerful command-line text editor that processes text **line-by-line** from top to bottom — without ever opening a graphical window.

While AWK is designed for slicing text into **columns and rows**, SED is designed for **finding, replacing, inserting, and deleting lines** in a stream of text or a file on the fly.

Think of SED as a **non-interactive, automated text editor** — like a headless Vim or Notepad that processes a file programmatically.

### 📝 Basic Syntax

```bash
sed expression filename
```

---

## 1️⃣ The Core Mental Model — Pattern Space

When you pass a file to SED, it reads it into memory **line-by-line** into what it calls the **Pattern Space**, performs whatever transformation you asked for, and prints the result to your terminal.

```
   ┌──────────────────────────────────────────────────────────────┐
   │              HOW SED PROCESSES A FILE:                       │
   │                                                              │
   │   INPUT FILE                                                 │
   │   ┌──────────────┐                                           │
   │   │ Line 1       │──┐                                        │
   │   │ Line 2       │  │  Read one line                         │
   │   │ Line 3       │  ▼                                        │
   │   │ ...          │  ┌──────────────┐                         │
   │   └──────────────┘  │  PATTERN     │  ← one line in memory   │
   │                     │  SPACE       │                         │
   │                     └──────┬───────┘                         │
   │                            │  Execute sed command            │
   │                            ▼                                 │
   │                     ┌──────────────┐                         │
   │                     │  OUTPUT      │  ← result to terminal   │
   │                     │  STREAM      │     (or file with -i)   │
   │                     └──────────────┘                         │
   │                            │                                 │
   │                            ▼                                 │
   │                     (repeat for next line)                   │
   └──────────────────────────────────────────────────────────────┘
```

### 🔑 The Most Common Command: Substitution (`s`)

The absolute most common SED command is **substitution**:

```
   sed 's/old_text/new_text/' filename
        │  │       │
        │  │       └── Replace with this
        │  └────────── Find this
        └───────────── s = substitute command
```

---

## 2️⃣ Key Flags You Must Know

Before diving into examples, memorize these two flags — they change everything:

### 📋 `g` (Global)

By default, SED only replaces the **first** occurrence of a word on a line. Adding `g` at the end tells it to replace **all** occurrences on that line:

```bash
# Without g — only first occurrence replaced
$ echo "foo bar foo baz" | sed 's/foo/XXX/'
XXX bar foo baz

# With g — ALL occurrences replaced
$ echo "foo bar foo baz" | sed 's/foo/XXX/g'
XXX bar XXX baz
```

### 📋 `-i` (In-Place)

By default, SED prints the result to your screen and **leaves the original file untouched**. Adding **`-i`** modifies and saves the file **directly on disk**:

```bash
# Without -i — prints to terminal, file unchanged
$ sed 's/old/new/g' config.txt
(new content appears on screen)

$ cat config.txt
(original content — unchanged)

# With -i — modifies the file on disk
$ sed -i 's/old/new/g' config.txt

$ cat config.txt
(new content — file modified!)
```

> ⚠️ **`-i` is permanent.** There's no undo. Always test your SED command without `-i` first (preview the output on screen), then add `-i` once you're confident it's correct.

---

## 3️⃣ Viewing and Filtering Lines (`p` and `-n`)

By default, SED prints **all lines** of a file along with any modifications. To filter and display only specific lines, use the **`-n`** flag combined with the **print (`p`)** command.

### 📋 Print a Specific Line

```bash
$ sed -n '3p' filename
```
Prints only line 3.

### 📋 Print the Last Line

```bash
$ sed -n '$p' filename
```
The `$` symbol represents the last line.

### 📋 Print a Range of Lines

```bash
$ sed -n '3,5p' filename
```
Prints lines 3 through 5.

### 📋 Print Starting from a Line with Offset

```bash
$ sed -n '2,+2p' filename
```
Starts at line 2, prints 2 more lines (lines 2, 3, 4).

### 📋 Step-Based Printing

```bash
$ sed -n '1~2p' filename
```
Starts at line 1, prints every 2nd line (1, 3, 5, 7, ...).

### 📋 Read Expressions from a File

```bash
$ sed -f expression_file filename
```
Stores SED expressions in a separate file and executes them.

---

## 4️⃣ Searching and Pattern Matching

### 🔍 Search for a Specific Pattern

```bash
$ sed -n '/India/p' filename
```
Finds and displays lines containing "India".

### 🔍 Multiple Expressions (`-e`)

To search for multiple patterns simultaneously (like "India" and "Germany"):

```bash
$ sed -n -e '/India/p' -e '/Germany/p' filename
```

Or print only specific lines with multiple expressions:

```bash
# Print only lines 2 and 5
$ sed -n -e '2p' -e '5p' filename
```

---

## 5️⃣ Substitution and Replacement (`s/.../.../g`)

### 📋 Basic Substitution

```bash
$ sed 's/old_word/new_word/g' filename
```
Replaces "old_word" with "new_word" globally across all lines.

### 📋 Line-Specific Substitution

Apply replacement only on a specific line number (e.g., line 2):

```bash
$ sed '2s/India/US/' filename
```

### 📋 Negated Substitution

Apply substitution everywhere **except** a specific line (e.g., excluding line 2):

```bash
$ sed '2!s/India/US/' filename
```
The `!` means "not this line."

### 📋 In-Place Modification

Save changes directly into the original file:

```bash
$ sed -i 's/old/new/g' filename
```

### 📋 Targeted Field/Value Updates

Combine pattern matching with substitution to modify specific values for a particular user:

```bash
# Update user "paul's" salary from 25000 to 35000
$ sed '/paul/s/25000/35000/g' filename

# Update paul's country from India to US
$ sed '/paul/s/India/US/g' filename
```

**Translation:** Find lines containing "paul", then within those lines, replace "25000" with "35000".

### 📋 Practical Config Update

```bash
# Change PORT=8080 to PORT=3000 in a config file
$ sed 's/PORT=8080/PORT=3000/' config.env

# Replace all localhost with 127.0.0.1
$ sed 's/localhost/127.0.0.1/g' hosts.conf

# Update CI/CD environment variable (in-place)
$ sed -i 's/API_ENV=staging/API_ENV=production/g' .env
```

---

## 6️⃣ Deleting Lines (`d`)

SED can remove lines based on line numbers or patterns:

### 📋 Delete a Specific Line

```bash
$ sed '1d' filename           # delete line 1
$ sed '3d' filename           # delete line 3
```

### 📋 Delete the Last Line

```bash
$ sed '$d' filename
```

### 📋 Delete a Range of Lines

```bash
$ sed '2,4d' filename         # delete lines 2 through 4
```

### 📋 Delete Lines Matching a Pattern

```bash
$ sed '/Belgium/d' filename   # delete lines containing "Belgium"
$ sed '/DEBUG/d' app.log      # delete lines containing "DEBUG"
```

### 📋 Delete Empty/Blank Lines

```bash
$ sed '/^$/d' filename
```
The `^$` regex matches empty lines (start `^` immediately followed by end `$`).

### 📋 Delete Lines Starting with `#` (Comments)

```bash
$ sed '/^#/d' nginx.conf
```
Strips all comment lines from a config file — great for viewing clean settings.

### 📋 Replace Tab with Space

```bash
$ sed 's/\t/ /g' filename
```

---

## 7️⃣ Inserting, Appending, and Changing Lines

SED can add or modify lines at specific positions:

### 📋 Append Data (`a`) — Add a Line AFTER

```bash
# Add "Hello User" after line 3
$ sed '3a\Hello User' filename

# Add text after lines matching "Paul"
$ sed '/Paul/a\new_text' filename
```

### 📋 Insert Data (`i`) — Add a Line BEFORE

```bash
# Add "Header text" before line 1
$ sed '1i\Header text' filename

# Add text before lines matching "Paul"
$ sed '/Paul/i\new_text' filename
```

### 📋 Change/Replace an Entire Line (`c`)

```bash
# Replace line 4's content entirely
$ sed '4c\New Line Content' filename
```

### 📋 Read External Files (`r`)

Insert the contents of an external text file after a specific line:

```bash
$ sed '3r external.txt' filename
```
Inserts the content of `external.txt` after line 3.

### 📋 Write Filtered Output to a New File (`w`)

```bash
# Extract all rows containing "India" and save them to a new file
$ sed -n '/India/w output.txt' filename
```

---

## 8️⃣ Advanced Operations

### 📋 Stop Execution Early (`q`)

Quit immediately after finding the first matching pattern:

```bash
$ sed '/India/q' filename       # stop after first "India"
$ sed '5q' filename             # stop after line 5
```

### 📋 Provide Exit Status

```bash
$ sed '/India/q 100' filename   # quit with exit code 100
```

### 📋 Execute External Command

```bash
$ sed '2e date' filename        # run `date` command at line 2
```

### 📋 Display Line Numbers (`=`)

```bash
$ sed '=' filename
```
Prints line numbers above each respective line.

### 📋 Show Hidden Characters (`l`)

```bash
$ sed -n 'l' filename           # show hidden characters (tabs, end-of-line)
$ sed -n 'l 50' filename        # wrap content at 50 characters
```

---

## 9️⃣ Regular Expressions (Regex) in SED

SED supports powerful regex for pattern matching:

### 📋 Regex Anchors

| Symbol | Meaning | Example |
|---|---|---|
| **`^`** | Start of line | `sed -n '/^B/p' names` — lines starting with B |
| **`$`** | End of line | `sed -n '/a$/p' names` — lines ending with a |
| **`.`** | Any single character | `/^S...a$/` — 5-letter names starting S, ending a |
| **`[]`** | Character set | `/[AC]/` — lines containing A or C |
| **`[^]`** | Exclusive set | `/[^0-9]/` — lines containing non-digits |
| **`*`** | Zero or more | `/ab*c/` — a, then zero+ b's, then c |

### 📋 Practical Regex Examples

```bash
# Find names starting with V
$ sed -n '/^V/p' names

# Find names ending with 'a'
$ sed -n '/a$/p' names

# Find a 5-letter name starting with S and ending with a
$ sed -n '/^S...a$/p' names

# Find names starting with only A and C
$ sed -n '/[AC]/p' names

# Find names starting with A to D
$ sed -n '/^[A-D]/p' names
```

### 📋 POSIX Character Classes

SED supports specialized character classes for advanced filtering:

| Class | Matches |
|---|---|
| **`[[:digit:]]`** | Lines containing numbers |
| **`[[:alpha:]]`** | Lines containing alphabets |
| **`[[:alnum:]]`** | Lines containing alphanumeric |
| **`[[:upper:]]`** | Lines containing uppercase letters |
| **`[[:lower:]]`** | Lines containing lowercase letters |
| **`[[:space:]]`** | Lines containing spaces |
| **`[[:punct:]]`** | Lines containing punctuation marks |
| **`[[:blank:]]`** | Lines containing blanks/tabs |

```bash
# Find lines containing numbers
$ sed -n '/[[:digit:]]/p' filename

# Find lines containing uppercase letters
$ sed -n '/[[:upper:]]/p' filename
```

---

## 🔟 Real DevOps Scenarios

| Scenario | SED Command | Why it matters |
|---|---|---|
| **Updating CI/CD env vars** | `sed -i 's/API_ENV=staging/API_ENV=production/g' .env` | Instantly update config flags on disk |
| **Stripping comments from config** | `sed '/^#/d' nginx.conf` | Delete all lines starting with `#` for clean viewing |
| **Deleting empty lines** | `sed '/^$/d' file.txt` | Clean up files by removing blank lines |
| **Replacing a port in config** | `sed -i 's/PORT=8080/PORT=3000/' config.env` | Update a service port without opening an editor |
| **Stripping DEBUG logs** | `sed '/DEBUG/d' app.log` | Remove all DEBUG lines from a log file |
| **Updating a user's salary** | `sed '/paul/s/25000/35000/g' employees.txt` | Targeted update for a specific user |
| **Replacing localhost** | `sed 's/localhost/127.0.0.1/g' hosts.conf` | Replace all localhost references |
| **Viewing clean config** | `sed -n '/^#/d; /^$/d' nginx.conf` | Strip comments AND empty lines |

---

## 1️⃣1️⃣ Quick Reference Cheat Sheet

### 📋 Viewing Lines

| Task | Command |
|---|---|
| **Print specific line** | `sed -n '3p' file` |
| **Print last line** | `sed -n '$p' file` |
| **Print range** | `sed -n '3,5p' file` |
| **Print with offset** | `sed -n '2,+2p' file` |
| **Print every 2nd line** | `sed -n '1~2p' file` |
| **Search pattern** | `sed -n '/India/p' file` |
| **Multiple patterns** | `sed -n -e '/India/p' -e '/Germany/p' file` |

### 📋 Substitution

| Task | Command |
|---|---|
| **Basic replace** | `sed 's/old/new/' file` |
| **Global replace** | `sed 's/old/new/g' file` |
| **In-place edit** | `sed -i 's/old/new/g' file` |
| **Line-specific** | `sed '2s/old/new/' file` |
| **Negated (except line)** | `sed '2!s/old/new/' file` |
| **Pattern-targeted** | `sed '/paul/s/25000/35000/g' file` |

### 📋 Deletion

| Task | Command |
|---|---|
| **Delete line N** | `sed '3d' file` |
| **Delete last line** | `sed '$d' file` |
| **Delete range** | `sed '2,4d' file` |
| **Delete by pattern** | `sed '/Belgium/d' file` |
| **Delete empty lines** | `sed '/^$/d' file` |
| **Delete comments** | `sed '/^#/d' file` |

### 📋 Insertion/Modification

| Task | Command |
|---|---|
| **Append after line** | `sed '3a\text' file` |
| **Insert before line** | `sed '1i\text' file` |
| **Change entire line** | `sed '4c\new text' file` |
| **Read external file** | `sed '3r external.txt' file` |
| **Write matches to file** | `sed -n '/India/w output.txt' file` |

### 📋 Advanced

| Task | Command |
|---|---|
| **Quit after match** | `sed '/India/q' file` |
| **Quit at line N** | `sed '5q' file` |
| **Show line numbers** | `sed '=' file` |
| **Show hidden chars** | `sed -n 'l' file` |
| **Read expressions from file** | `sed -f expr_file file` |

### 🧠 Memory Hooks

- **`s`** = **s**ubstitute
- **`g`** = **g**lobal (all occurrences on a line)
- **`-i`** = **i**n-place (modify file on disk)
- **`-n`** = suppress automatic print (use with `p`)
- **`p`** = **p**rint
- **`d`** = **d**elete
- **`a`** = **a**ppend (after)
- **`i`** = **i**nsert (before)
- **`c`** = **c**hange (replace entire line)
- **`w`** = **w**rite to file
- **`r`** = **r**ead from file
- **`q`** = **q**uit
- **`$`** = last line
- **`^`** = start of line

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **SED** | Stream Editor — non-interactive text editor. Processes line-by-line. |
| **Pattern Space** | The memory area where SED holds one line at a time for processing. |
| **`s/old/new/`** | Substitute — replace first occurrence of "old" with "new". |
| **`s/old/new/g`** | Global substitute — replace ALL occurrences on a line. |
| **`-i`** | In-place editing — modifies the file on disk. No undo! |
| **`-n`** | Suppress automatic printing. Use with `p` to print only matches. |
| **`p`** | Print command. `sed -n '3p' file` prints line 3. |
| **`d`** | Delete command. `sed '3d' file` deletes line 3. |
| **`$`** | Last line. `sed -n '$p' file` prints the last line. |
| **Range** | `sed -n '3,5p' file` prints lines 3–5. |
| **Offset** | `sed -n '2,+2p' file` starts at line 2, prints 2 more. |
| **Step** | `sed -n '1~2p' file` prints every 2nd line starting at 1. |
| **`-e`** | Multiple expressions. `sed -e '2p' -e '5p'` (with `-n`). |
| **`-f`** | Read expressions from a file. |
| **`a`** | Append — add a line AFTER the specified line. |
| **`i`** | Insert — add a line BEFORE the specified line. |
| **`c`** | Change — replace an entire line with new text. |
| **`w`** | Write — write matched lines to a new file. |
| **`r`** | Read — insert contents of an external file after a line. |
| **`q`** | Quit — stop processing after a match. |
| **`=`** | Display line numbers. |
| **`l`** | Show hidden characters (tabs, end-of-line markers). |
| **Line-specific substitution** | `sed '2s/old/new/' file` — only on line 2. |
| **Negated substitution** | `sed '2!s/old/new/' file` — everywhere EXCEPT line 2. |
| **Pattern-targeted** | `sed '/paul/s/25000/35000/g' file` — only on lines containing "paul". |
| **`^`** | Start of line anchor. `/^#/d` deletes comment lines. |
| **`$` (in regex)** | End of line anchor. `/a$/p` prints lines ending with "a". |
| **`.`** | Any single character. `/^S...a$/` = 5-char names starting S, ending a. |
| **`[]`** | Character set. `/[AC]/p` = lines containing A or C. |
| **`[A-D]`** | Range. `/^[A-D]/p` = lines starting with A through D. |
| **POSIX `[[:digit:]]`** | Lines containing digits. |
| **POSIX `[[:alpha:]]`** | Lines containing alphabets. |
| **POSIX `[[:upper:]]`** | Lines containing uppercase. |
| **POSIX `[[:space:]]`** | Lines containing spaces. |
| **`/^$/d`** | Delete empty lines (start immediately followed by end). |
| **`/^#/d`** | Delete comment lines (start with `#`). |
| **`/\t/ /g`** | Replace tabs with spaces: `sed 's/\t/ /g' file`. |
| **AWK vs SED** | SED = line manipulation (replace/delete/insert). AWK = column extraction + calculations. |
| **DevOps relevance** | SED = config updates, comment stripping, log cleanup, find-and-replace. The scalpel of Linux text processing. |

---

← Back to [Text Processing](./README.md) | Previous: [AWK](./1-awk.md) |
