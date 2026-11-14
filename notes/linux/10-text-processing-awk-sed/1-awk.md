# 📊 AWK — The Data Analytics Workbench

> **Goal:** Master AWK — from the records/fields mental model to built-in variables (`$1`/`NR`/`NF`/`$NF`), pattern matching, CSV delimiters, filtering, built-in functions (`gsub`/`length`/`toupper`), `BEGIN`/`END` blocks, and AWK as a full scripting language.

---

## 📍 What Is AWK?

**AWK** is not just a standard Linux command — it's a powerful text-processing tool and a **domain-specific programming language**. Named after its creators (Alfred **A**ho, Peter **W**einberger, and Brian **K**ernighan), AWK is designed specifically for **extracting and manipulating text data in columns and rows**.

As a DevOps engineer, you'll constantly encounter text-based data: server logs, configuration files, CSV reports, or output from system commands (`df -h`, `ps aux`). While `grep` can find *lines*, AWK can slice those lines into *words and columns*, perform calculations, and filter data with surgical precision.

### 📥 How AWK Receives Data

1. **Directly reading a file:** `awk '{print $1}' file.txt`
2. **Taking output from another command via a pipe:** `ps aux | awk '{print $1}'`

### 📝 Basic Syntax

```bash
awk 'options [pattern] {action}' filename
```

```
   ┌──────────────────────────────────────────────────────────────┐
   │  AWK COMMAND ANATOMY:                                        │
   │                                                              │
   │  awk 'pattern {action}' filename                             │
   │       │      │                                               │
   │       │      └── WHAT TO DO (wrapped in { })                 │
   │       │          Usually {print ...}                         │
   │       │                                                      │
   │       └────────── WHEN TO RUN (a condition)                  │
   │                    Omit pattern → runs on EVERY line         │
   │                                                              │
   │  Examples:                                                   │
   │  awk '{print $2}' file.txt          (no pattern = all lines) │
   │  awk '/error/ {print $0}' log.txt   (pattern = lines with    │
   │                                      "error")                │
   │  awk '$3 > 50 {print $1}' data.txt  (pattern = column 3 >50) │
   └──────────────────────────────────────────────────────────────┘
```

- **`pattern`**: A condition (e.g., matching a keyword or checking if column 3 > 10). If you omit the pattern, AWK executes the action on **every single line**.
- **`action`**: What AWK should do (wrapped in curly braces `{}`). Usually `{print ...}`.

---

## 1️⃣ The Core Mental Model — Records and Fields

To understand AWK, you need to understand how it views a file:

```
   ┌──────────────────────────────────────────────────────────────┐
   │              HOW AWK VIEWS A FILE:                           │
   │                                                              │
   │   FILE:                                                      │
   │   ┌──────────────────────────────────────┐                   │
   │   │ PID USER %CPU COMMAND                │ ← Record 1 (NR=1) │
   │   │ 1234 root 0.5 nginx                  │                   │
   │   ├──────────────────────────────────────┤                   │
   │   │ 5678 john 2.1 python3                │ ← Record 2 (NR=2) │
   │   ├──────────────────────────────────────┤                   │
   │   │ 9012 app  0.3 docker                 │ ← Record 3 (NR=3) │
   │   └──────────────────────────────────────┘                   │
   │                                                              │
   │   Each LINE = a RECORD (tracked by NR)                       │
   │   Each WORD = a FIELD (tracked by $1, $2, $3...)             │
   │                                                              │
   │   For Record 2: "5678 john 2.1 python3"                      │
   │   $1 = 5678   (PID)                                          │
   │   $2 = john   (USER)                                         │
   │   $3 = 2.1    (%CPU)                                         │
   │   $4 = python3 (COMMAND)                                     │
   └──────────────────────────────────────────────────────────────┘
```

### 📊 Records (Rows)

By default, AWK reads a file **line by line**. Each line is treated as a **record**. AWK tracks the current line number using `NR`.

### 📊 Fields (Columns)

Within each line, words/tokens separated by whitespace (spaces or tabs) are treated as **fields**:

| Variable | What it represents |
|---|---|
| **`$1`** | The first column |
| **`$2`** | The second column |
| **`$3`** | The third column |
| **`$0`** | The **entire** line/record |
| **`NF`** | Number of Fields (total columns in the current line) |
| **`NR`** | Number of Records (current line number) |
| **`$NF`** | The LAST field (regardless of how many fields there are) |

> 💡 The `$NF` trick is incredibly useful — it gives you the last column without having to count how many columns exist. If a file has 6 columns, `$NF` = `$6`. If it has 10, `$NF` = `$10`.

---

## 2️⃣ Basic Practical Examples

### 📋 Printing a Specific Column

```bash
$ awk '{print $2}' sample.txt
```
**Translation:** For every line, print column 2 (`$2`).

### 📋 Printing Multiple Columns

```bash
$ awk '{print $4, $2}' sample.txt
```
Print columns 4 and 2 (in any order you want, separated by commas).

### 📋 Printing with Custom Formatting

```bash
$ awk '{print $1 " - " $3}' users.txt
```
String concatenation — prints `$1`, a dash, and `$3` together.

### 📋 Printing the Last Column Dynamically

```bash
$ awk '{print $NF}' sample.txt
```
Instead of counting columns manually, use `NF` to fetch the last column (e.g., Salary):

```bash
$ awk '{print $NF}' employees.txt
37000
32000
25000
45000
```

### 📋 Printing with Line Numbers (`NR`)

```bash
$ awk '{print NR, $0}' sample.txt
1 id fname lname desig dept salary
2 101 Luna Rastogi Manager Loan 37000
3 102 Mochi Mohan Chashier Cash 32000
```

---

## 3️⃣ Search and Pattern Matching

### 🔍 Searching for a Specific Word

Place the search pattern inside forward slashes before the action block:

```bash
$ awk '/manish/ {print $0}' sample.txt
```
Prints every line containing "manish".

### 🔍 Displaying Line Numbers with Search

```bash
$ awk '/Bob/ {print NR, $0}' sample.txt
5 104 Bob Philip Accountant Account 45000
9 108 Latte Bob Manager IT 75000
```

### 🔍 Printing a Specific Line (e.g., Line 6)

```bash
$ awk 'NR == 6 {print $0}' sample.txt
```

### 🔍 Printing a Range of Lines (e.g., Lines 3–6)

```bash
$ awk 'NR == 3, NR == 6 {print $0}' sample.txt
```

Or using comparison operators:

```bash
$ awk 'NR >=2 && NR <=10 {print}' app.log
```

### 🔍 Searching Multiple Words

Use `||` (OR) to look for multiple words:

```bash
$ awk '/raven/ || /salina/ || /alex/ {print $0}' sample.txt
```

Or use regex pipe `|` inside the pattern:

```bash
$ awk '/word1|word2/ {print $2}' file
```

### 🔍 Case-Insensitive Search

Initialize `IGNORECASE = 1` in a `BEGIN` block:

```bash
$ awk 'BEGIN {IGNORECASE = 1} /luna/ {print $0}' sample.txt
```

Without `IGNORECASE`, AWK is case-sensitive — `/luna/` won't match "Luna".

### 🔍 Matching a Specific Field (`~` operator)

The `~` symbol means "matches". To find lines where the **second field** contains the letter "a":

```bash
$ awk '$2 ~ /a/ {print $0}' sample.txt
```

---

## 4️⃣ Handling Delimiters (CSVs & Custom Separators)

By default, AWK uses **whitespace** (spaces or tabs) as delimiters. If your file uses commas (CSV) or other separators, you must specify the field separator with **`-F`**:

### 📋 For CSV Files

```bash
$ awk -F',' '{print $2}' employees.csv
```
**Translation:** Set the field separator to `,` (`-F','`), and print column 2.

### 📋 For `/etc/passwd` (Colon-Separated)

```bash
$ awk -F: '{print $1, $7}' /etc/passwd
```
**Translation:** Set separator to `:` (`-F:`), print username (col 1) and shell (col 7).

### 📋 For Multiple Delimiters (Comma, Colon, or Hyphen)

```bash
$ awk -F'[,:-]' '{print $2}' multi_delimiter.txt
```
Use square brackets `[ ]` to specify multiple delimiter characters.

### 📋 Common `-F` Patterns

| Flag | Separator | Use case |
|---|---|---|
| `-F' '` | Space (default) | Standard text files |
| `-F','` | Comma | CSV files |
| `-F:` | Colon | `/etc/passwd`, `/etc/shadow` |
| `-F'\t'` | Tab | TSV files |
| `-F'[,:-]'` | Multiple | Mixed-delimiter files |

---

## 5️⃣ Conditions and Filtering

AWK can filter data based on **numeric values** in specific columns.

### 📋 Filter by Numeric Column

```bash
# Print employees making more than 50,000 (CSV file, salary is last column)
$ awk -F',' '$NF > 50000 {print $0}' employees.csv
```

### 📋 Filter by Column Value

```bash
# Print lines where column 3 is greater than 50
$ awk '$3 > 50 {print $1, $3}' metrics.txt
```

### 📋 Combined Conditions

```bash
# Logs between two timestamps (column 2 is a timestamp)
$ awk '$2 >= "08:50:00" && $2 <= "08:59:59" {print $2, $4}' app.log
```

### 📋 Practical Case: Fetching Logs by Time Range

```bash
# "From 08:50:00 to 9:00:00 there were issues — find out what happened"
$ awk '$2 >= "08:50:00" && $2 <= "08:59:59" {print $2, $4}' app.log
```

---

## 6️⃣ Built-in String and Numeric Functions

AWK has powerful built-in functions for text manipulation:

### 📋 `gsub()` — Global Substitution

Replaces a word **globally** across the text:

```bash
$ awk '{gsub("raven", "king"); print $0}' sample.txt
```
Every occurrence of "raven" becomes "king" on each line.

### 📋 `length()` — String Length

Finds the character length of a specific field:

```bash
$ echo "Hi Paul" | awk '{print length($2)}'
4
```
Length of "Paul" = 4 characters.

### 📋 `index()` — Find Position

Returns the position of a substring within a string:

```bash
$ echo "Hi Paul" | awk '{print index($0, "Paul")}'
4
```
"Paul" starts at position 4 in "Hi Paul".

### 📋 `toupper()` / `tolower()` — Case Conversion

Changes text casing:

```bash
$ awk '{print toupper($5)}' sample.txt
$ awk '{print tolower($5)}' sample.txt
```

### 📋 `split()` — Split into Array

Splits a string into an array:

```bash
$ awk -F, '{split($0, arr, ","); print arr[1]}' file
```

### 📋 Function Summary

| Function | What it does | Example |
|---|---|---|
| **`gsub("old", "new")`** | Replace all occurrences | `gsub("raven", "king")` |
| **`length($2)`** | Character count of field | `length($2)` → 4 |
| **`index($0, "word")`** | Position of substring | `index($0, "Paul")` → 4 |
| **`toupper($5)`** | Convert to uppercase | `toupper($5)` |
| **`tolower($5)`** | Convert to lowercase | `tolower($5)` |
| **`split($0, arr, ",")`** | Split into array | `split($0, arr, ",")` |

---

## 7️⃣ Counting and Aggregation

AWK excels at counting and summing data — perfect for log analysis.

### 📋 Count Matching Lines

```bash
# Count how many times "INFO" appears in a log
$ awk '/INFO/ {count++} END {print count}' app.log
16
```

### 📋 Formatted Count

```bash
$ awk '/INFO/ {count++} END {print "The count of INFO IS", count}' app.log
The count of INFO IS 16
```

### 📋 Count a Specific IP

```bash
awk '$1 == "9.67.116.98" {count++}
     END {print count+0}' app.log
```

### 📋 Print Specific Fields for Matches

```bash
# Look for "INFO" and print columns 1, 2, 3, and 5
$ awk '/INFO/ {print $1, $2, $3, $5}' app.log
```

---

## 8️⃣ `BEGIN` and `END` Blocks

AWK scripts can be split into **three logical phases**:

```
   ┌──────────────────────────────────────────────────────────────┐
   │              AWK EXECUTION PHASES:                           │
   │                                                              │
   │   1. BEGIN block    ← runs ONCE before reading any lines     │
   │      (setup: initialize vars, print headers)                 │
   │                                                              │
   │   2. Main body       ← runs for EVERY line                   │
   │      pattern {action}                                        │
   │                                                              │
   │   3. END block       ← runs ONCE after all lines processed   │
   │      (cleanup: print totals, summaries)                      │
   │                                                              │
   │   Structure:                                                 │
   │   awk '                                                      │
   │     BEGIN { start_action }                                   │
   │     pattern/condition { action }                             │
   │     END { end_action }                                       │
   │   ' filename                                                 │
   └──────────────────────────────────────────────────────────────┘
```

### 📋 Example: Printing Headers and Footers

```bash
$ awk 'BEGIN {print "--- START OF REPORT ---"} {print $1} END {print "--- END OF REPORT ---"}' file.txt
--- START OF REPORT ---
Alice
Bob
Charlie
--- END OF REPORT ---
```

### 📋 Example: Calculating Total Salary

```bash
$ awk '
BEGIN { sum = 0 }
NR > 1 { sum += $NF }
END { print "Total Salary Paid:", sum }
' employees.csv
Total Salary Paid: 234000
```

**Breakdown:**
- `BEGIN { sum = 0 }` — initialize the sum variable before reading any lines
- `NR > 1 { sum += $NF }` — for every line after the first (skip header), add the last column (salary) to sum
- `END { print "Total Salary Paid:", sum }` — after all lines, print the total

### 📋 Example: Average Salary

```bash
$ awk -F, '{sum += $NF} END {print sum/NR}' employees.csv
```
Divide total by number of records (NR).

### 📋 Example: Count Lines (Ignoring Header)

```bash
$ awk 'NR > 1 {count++} END {print count}' /etc/passwd
```
Count users by skipping the first line (header).

---

## 9️⃣ AWK as a Full Programming Language

AWK isn't limited to one-liners — it supports variables, conditionals, loops, arrays, and functions.

### 📋 Variables

```bash
awk '{count++; total += $2} END {print "Count:", count, "Total:", total}' file
```

### 📋 If-Else

```bash
# Print "High" for salary > 50k, else "LOW"
awk -F, '{if ($NF > 50000) print $1, "High"; else print $1, "LOW"}' employees.csv
```

### 📋 Arrays

```bash
awk -F, '{split($0, arr, ","); print arr[1], arr[3]}' file
```

### 📋 For Loop

```bash
# Print all fields of each line
awk '{for (i=1; i<=NF; i++) print $i}' file
```

### 📋 While Loop

```bash
awk '{i=1; while (i<=NF) {print $i; i++}}' file
```

---

## 🔟 AWK Script Files

For complex AWK programs, store the script in a file and run it with `-f`:

### 📋 Method 1: Run an AWK Script File

```bash
# Save your AWK code in a file (e.g., script.awk)
$ cat script.awk
BEGIN {print "Starting analysis..."}
$3 > 50 {print $1, $3}
END {print "Done."}

# Run it
$ awk -f script.awk data.txt
```

### 📋 Method 2: Executable AWK Script (with Shebang)

```bash
#!/bin/awk -f

BEGIN {print "Starting analysis..."}
$3 > 50 {print $1, $3}
END {print "Done."}
```

```bash
$ chmod +x script.awk
$ ./script.awk data.txt
```

The shebang `#!/bin/awk -f` tells the system to use AWK as the interpreter.

---

## 1️⃣1️⃣ Real DevOps Scenarios

| Scenario | AWK Command | Why it matters |
|---|---|---|
| **Parsing web server logs** | `awk '{print $1}' /var/log/nginx/access.log` | Extract client IP addresses to analyze visitors or trace DDoS sources |
| **Summing disk space** | `df -h \| awk '{print $5, $6}'` | Extract disk utilization % and mount points |
| **Finding memory-hogging processes** | `ps aux \| awk '$4 > 5.0 {print $1, $2, $11}'` | Filter processes using more than 5% memory |
| **Counting HTTP status codes** | `awk '{print $9}' access.log \| sort \| uniq -c` | Count error frequencies (200, 404, 500) |
| **Counting log entries** | `awk '/INFO/ {count++} END {print count}' app.log` | Count how many times "INFO" appears |
| **Fetching logs by time range** | `awk '$2 >= "08:50:00" && $2 <= "08:59:59" {print}' app.log` | Extract logs from a specific time window |
| **Parsing CSV files** | `awk -F, '{print $2}' employees.csv` | Extract specific columns from CSVs |
| **Parsing /etc/passwd** | `awk -F: '{print $1, $7}' /etc/passwd` | List usernames and their shells |
| **Top 5 IPs from access log** | `awk '{print $1}' access.log \| sort \| uniq -c \| sort -rn \| head -5` | Find the most active visitors |

---

## 1️⃣2️⃣ Quick Reference Cheat Sheet

| Task | AWK Command |
|---|---|
| **Print specific column** | `awk '{print $2}' file.txt` |
| **Print multiple columns** | `awk '{print $1, $3}' file.txt` |
| **Print last column** | `awk '{print $NF}' file.txt` |
| **Print entire line** | `awk '{print $0}' file.txt` |
| **Print with line numbers** | `awk '{print NR, $0}' file.txt` |
| **Print specific line (e.g., 6)** | `awk 'NR == 6 {print $0}' file.txt` |
| **Print range (lines 3–6)** | `awk 'NR == 3, NR == 6 {print $0}' file.txt` |
| **Search keyword** | `awk '/keyword/ {print $0}' file.txt` |
| **Search multiple words** | `awk '/word1\|word2/ {print $0}' file.txt` |
| **Case-insensitive search** | `awk 'BEGIN{IGNORECASE=1} /word/ {print}' file.txt` |
| **Match specific field** | `awk '$2 ~ /a/ {print $0}' file.txt` |
| **CSV field separator** | `awk -F, '{print $1}' file.csv` |
| **Colon separator (/etc/passwd)** | `awk -F: '{print $1, $7}' /etc/passwd` |
| **Multiple delimiters** | `awk -F'[,:-]' '{print $2}' file.txt` |
| **Filter by numeric column** | `awk '$3 > 50 {print $0}' file.txt` |
| **Count matching lines** | `awk '/INFO/ {count++} END {print count}' app.log` |
| **Sum a column** | `awk '{sum += $NF} END {print sum}' file.txt` |
| **Average a column** | `awk '{sum += $NF} END {print sum/NR}' file.txt` |
| **Global substitution** | `awk '{gsub("old", "new"); print}' file.txt` |
| **String length** | `awk '{print length($2)}' file.txt` |
| **Uppercase** | `awk '{print toupper($5)}' file.txt` |
| **BEGIN/END blocks** | `awk 'BEGIN{print "Start"} {print $1} END{print "Done"}' file.txt` |
| **Run script from file** | `awk -f script.awk data.txt` |
| **Executable AWK script** | `#!/bin/awk -f` (shebang) |

### 🧠 Memory Hooks

- **`NR`** = **N**umber of **R**ecord (line number)
- **`NF`** = **N**umber of **F**ields (column count)
- **`$NF`** = the LAST field (dollar-NF)
- **`$0`** = the whole line (zero = all)
- **`-F`** = **F**ield separator
- **`BEGIN`/`END`** = before/after the file
- **`gsub`** = **g**lobal **sub**stitution

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **AWK** | Powerful text-processing programming language. Named after Aho, Weinberger, Kernighan. |
| **Records** | Each line = a record. Tracked by `NR` (Number of Record). |
| **Fields** | Each word/column = a field. `$1`, `$2`, `$3`... |
| **`$0`** | The entire line/record. |
| **`NR`** | Current line number (Number of Record). |
| **`NF`** | Number of Fields (columns) in current line. |
| **`$NF`** | The LAST field — regardless of column count. Incredibly useful. |
| **Syntax** | `awk 'pattern {action}' filename`. Omit pattern = runs on every line. |
| **Pattern** | A condition (keyword match, numeric comparison). Decides WHEN to run the action. |
| **Action** | What to do, in `{ }`. Usually `{print ...}`. |
| **`-F`** | Set field separator. `-F,` for CSV, `-F:` for /etc/passwd, `-F'[,:-]'` for multiple. |
| **Search** | `/keyword/ {print $0}` — prints lines containing "keyword". |
| **Case-insensitive** | `BEGIN {IGNORECASE=1} /word/ {print}` |
| **`~` operator** | Matches a field against a regex: `$2 ~ /a/` (field 2 contains "a"). |
| **Line filtering** | `NR == 6`, `NR >= 2 && NR <= 10`, `NR == 3, NR == 6` (range). |
| **Numeric filtering** | `$3 > 50 {print $0}` — column 3 greater than 50. |
| **Time range filtering** | `$2 >= "08:50:00" && $2 <= "08:59:59"` — string comparison on timestamps. |
| **`gsub("old", "new")`** | Global substitution — replace all occurrences. |
| **`length($2)`** | Character count of a field. |
| **`index($0, "word")`** | Position of substring in a line. |
| **`toupper()` / `tolower()`** | Case conversion. |
| **`split($0, arr, ",")`** | Split a string into an array. |
| **`BEGIN` block** | Runs once BEFORE reading any lines. Setup, headers, init. |
| **`END` block** | Runs once AFTER all lines processed. Summaries, totals. |
| **Counting** | `/INFO/ {count++} END {print count}` — count matching lines. |
| **Sum** | `{sum += $NF} END {print sum}` — sum a column. |
| **Average** | `{sum += $NF} END {print sum/NR}` — average a column. |
| **Script file** | `awk -f script.awk data.txt` — run AWK code from a file. |
| **Executable script** | `#!/bin/awk -f` shebang — run AWK scripts directly. |
| **Full programming** | AWK supports variables, if/else, arrays, for/while loops, functions. |
| **DevOps relevance** | AWK = log parsing, CSV analysis, metrics extraction, counting, summing. The data analytics workbench of Linux. |

---

← Back to [Text Processing](./README.md) | Next: [SED](./2-sed.md) →
