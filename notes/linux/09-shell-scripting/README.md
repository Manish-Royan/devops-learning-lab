# 🐧 Linux Learning Journey: Shell Scripting — Automating the CLI

> **Goal:** Master shell scripting from the ground up — what a shell is, how to write and run scripts, the shebang, `source` vs `bash` execution, variables/constants/arrays/strings, user input, arithmetic, control structures (`if`/`elif`/`case`), loops, and making your scripts system-wide commands via `PATH`.

---

## 📍 Why Shell Scripting Matters for DevOps

Everything we've learned so far — the CLI, file permissions, process management — has been **interactive**: you type a command, you get a result. But DevOps is about **automation at scale**. You don't want to type 50 commands to deploy an app. You want to write them once in a file and run that file.

That file is a **shell script**.

Shell scripting lets you:
- Combine multiple commands into a single automatable file
- Execute batch jobs (backups, deployments, log rotation)
- Schedule automation via **cron jobs**
- Accept arguments and user input dynamically
- Add conditional logic (`if` this fails, do that)
- Loop over lists (process every file in a directory)

If you're going to be a DevOps engineer, shell scripting is your **native language**. Python is great for complex logic, but Bash is everywhere — every server, every container, every CI/CD runner. And it's the glue that ties everything together.

> 💡 **Terminology note:** Beginners often ask: "Is it called bash scripting or shell scripting?" They're the same thing. `sh` is the old version, `bash` (Bourne Again SHell) is the newer version, and `zsh` is an even newer, more advanced version (often the default on macOS). When we say "shell scripting," we almost always mean Bash.

---

## 1️⃣ What Is a Shell? Types & Diagnostic Commands

### 🐚 What Is a Shell?

A **shell** is a user interface that allows you to interact with the underlying operating system and hardware (CPU, RAM, hard disk). It sits between you and the kernel:

```
   ┌──────────────────────────────────────────────────────────────┐
   │              LINUX SYSTEM ARCHITECTURE                       │
   │                                                              │
   │   ┌──────────────┐                                           │
   │   │ APPLICATIONS │  (nginx, docker, your scripts)            │
   │   └──────┬───────┘                                           │
   │          │                                                   │
   │   ┌──────▼───────┐                                           │
   │   │    SHELL     │  ← bash/zsh/fish — your interface         │
   │   │  (the CLI)   │     interprets your commands              │
   │   └──────┬───────┘                                           │
   │          │                                                   │
   │   ┌──────▼───────┐                                           │
   │   │   KERNEL     │  ← manages hardware, processes, memory    │
   │   └──────┬───────┘                                           │
   │          │                                                   │
   │   ┌──────▼───────┐                                           │
   │   │   HARDWARE   │  ← CPU, RAM, disk, NIC                    │
   │   └──────────────┘                                           │
   └──────────────────────────────────────────────────────────────┘
```

### 📋 Common Shell Types

| Shell | Full Name | Notes |
|---|---|---|
| **Bash** | **B**ourne **A**gain **SH**ell | Most common, widely used. Default on most Linux distros. |
| **sh** | Bourne Shell | The original. Old version. |
| **zsh** | Z Shell | More advanced. **Default on macOS.** |
| **ksh** | KornShell | Common on commercial Unix (AIX, Solaris). |
| **csh** | C Shell | C-like syntax. Older. |
| **fish** | Friendly Interactive Shell | User-friendly, but not POSIX-compliant. |

### 🔍 Essential Diagnostic Commands

```bash
# Check your current/default shell
$ echo $0
/usr/bin/bash

# List all available shells on the system
$ cat /etc/shells
# /bin/sh
# /bin/bash
# /usr/bin/zsh
# /usr/bin/fish

# Check your OS distribution version
$ cat /etc/os-release

# Check current logged-in user
$ whoami
```

> 🧭 **DevOps tip:** `echo $0` is the fastest way to check which shell you're in. This matters because scripts written for Bash may not work in `fish` or `zsh` — the shebang (covered next) solves this by explicitly specifying the interpreter.

---

## 2️⃣ Creating & Running Your First Script

### 📝 The Shebang (`#!`)

The **shebang** is the first line of a script. It specifies **which interpreter** should be used to execute the file.

```bash
#!/bin/bash
```

```
   ┌──────────────────────────────────────────────────────────────┐
   │  THE SHEBANG — HOW IT WORKS:                                 │
   │                                                              │
   │  #!/bin/bash                                                 │
   │  │                                                           │
   │  │  ├── # = normally a comment, BUT...                       │
   │  │  ├── ! = ...makes it special (shebang!)                   │
   │  │  └── /bin/bash = path to the interpreter                  │
   │  │                                                           │
   │  │  When you run ./script.sh:                                │
   │  │  1. Bash reads the first line                             │
   │  │  2. Sees #!/bin/bash                                      │
   │  │  3. Says: "use /bin/bash to interpret this file"          │
   │  │                                                           │
   │  │  Why it's powerful:                                       │
   │  │  - If the shebang was #!/usr/bin/python3,                 │
   │  │    bash would run it as Python — no need to invoke        │
   │  │    python3 explicitly.                                    │
   │  │  - Guarantees the right interpreter runs the script,      │
   │  │    even if invoked from fish, zsh, or PowerShell.         │
   └──────────────────────────────────────────────────────────────┘
```

> 💡 The shebang `#` looks like a comment, but the `!` makes it special. It's the one case where `#` is not treated as a comment.

### 📝 Your First Script

```bash
# Create the file
$ vim mybash.sh

#!/bin/bash
echo "Hello Buddy"

# Save and quit: :wq
```

**File extension:** While the `.sh` extension is not strictly mandatory (the shebang determines the interpreter, not the extension), it's strongly recommended as a **standard convention** to identify shell script files.

### 🏃 Running the Script — Four Ways

| Method | Command | Needs execute permission? |
|---|---|---|
| **1. Relative path** | `./mybash.sh` | ✅ Yes (`chmod +x` first) |
| **2. Absolute path** | `/home/manish/mybash.sh` | ✅ Yes |
| **3. Interpreter directly** | `bash mybash.sh` | ❌ No |
| **4. System-wide** (after PATH setup) | `mybash.sh` | ✅ Yes |

```bash
# Step 1: Give execute permission
$ chmod +x mybash.sh

# Step 2: Run using relative path
$ ./mybash.sh
Hello Buddy

# Step 3: Or run directly using the interpreter (no chmod needed)
$ bash mybash.sh
Hello Buddy
```

> ⚠️ If you try `./mybash.sh` without execute permissions, you get: `bash: ./mybash.sh: Permission denied`. Recall Day 5 — fix with `chmod +x`.

### 📋 Process Control While Running

| Shortcut | Action |
|---|---|
| **`Ctrl+C`** | **Terminate** the script (SIGINT) |
| **`Ctrl+Z`** | **Stop/pause** the script (suspend) |

---

## 3️⃣ `source` vs `bash` — The Subprocess Question

This is one of the most important distinctions in shell scripting, and it confuses every beginner.

### 📊 The Two Execution Models

```bash
# Method 1: bash (creates a SUBPROCESS)
$ bash mybash.sh

# Method 2: source (runs in CURRENT process)
$ source mybash.sh
# OR (equivalent — the dot works in more distros):
$ . mybash.sh
```

```
   ┌──────────────────────────────────────────────────────────────┐
   │  bash mybash.sh           source mybash.sh                   │
   │  (or . mybash.sh)                                            │
   │                                                              │
   │  ┌──────────────┐         ┌──────────────┐                   │
   │  │ Parent shell │         │ Parent shell │                   │
   │  │  (your term) │         │  (your term) │                   │
   │  └──────┬───────┘         │   │          │                   │
   │         │ spawns          │   │ runs IN  │                   │
   │         ▼                 │   │ current  │                   │
   │  ┌──────────────┐         │   │ process  │                   │
   │  │ Subprocess   │         │   ▼          │                   │
   │  │ (new bash)   │         │  script runs │                   │
   │  │  runs script │         │  HERE — no   │                   │
   │  │  cd ~/temp   │         │  subprocess  │                   │
   │  │  ...then     │         │  cd ~/temp   │                   │
   │  │  EXITS       │         │  ← STAYS!    │                   │
   │  └──────────────┘         └──────────────┘                   │
   │                                                              │
   │  cd changes DON'T           cd changes DO                    │
   │  affect parent shell        affect current shell             │
   │  (subprocess died)          (same process)                   │
   └──────────────────────────────────────────────────────────────┘
```

### 🔍 Why This Matters

If your script contains `cd ~/temp`:
- **`bash mybash.sh`** → the subprocess changes to `~/temp`, runs, then exits. Your terminal is **back where it started**.
- **`source mybash.sh`** → the script runs in your current shell. The `cd` **persists** — you're now in `~/temp`.

```bash
# Demonstration:
$ cat mybash.sh
#!/bin/bash
cd ~/temp
echo "I am now in $(pwd)"

$ pwd
/home/manish

$ bash mybash.sh
I am now in /home/manish/temp

$ pwd
/home/manish        # ← still here! subprocess died

$ source mybash.sh
I am now in /home/manish/temp

$ pwd
/home/manish/temp   # ← MOVED! same process
```

> 🧭 **DevOps note:** The `source` command only works in some Linux distributions, while the dot notation (`. script.sh`) works in more distributions. Both execute the script in the current shell rather than creating a subprocess. Use `source` for loading config files (like `. ~/.bashrc`), use `bash` for running independent scripts.

---

## 4️⃣ Comments

### 📝 Single-Line Comments

Use `#` at the beginning of a line or after code:

```bash
# This is a single-line comment
echo "Hello"   # This is also a comment
```

Everything after `#` on a line is treated as a comment and not executed.

### 📝 Multi-Line Comments (Here-Doc Technique)

Use the here-doc technique for multi-line comments:

```bash
: << 'COMMENT'
This is a
multi-line comment
that spans several lines.
COMMENT

echo "This line runs"
```

The `:` command does nothing (it's a no-op), and `<< 'COMMENT'` redirects everything until `COMMENT` into it — effectively discarding the text as a comment.

---

## 5️⃣ Variables & Constants

### 📝 Variable Assignment & Usage

```bash
NAME="Manish"
AGE=22
echo "My name is $NAME and I am $AGE years old."
# Output: My name is Manish and I am 22 years old.
```

### ⚠️ The #1 Beginner Trap: No Spaces Around `=`

```bash
# ✅ CORRECT — no spaces
NAME="Manish"

# ❌ WRONG — spaces around =
NAME = "Manish"    # bash thinks NAME is a command to run!
```

Bash is strict: **no spaces around the equals sign** when assigning variables.

### 📋 Accessing Variables

| Syntax | When to use |
|---|---|
| **`$VAR`** | Simple access |
| **`${VAR}`** | When concatenating with other text (removes ambiguity) |

```bash
FILE_PREFIX="file"
touch $FILE_PREFIX1.txt      # ← bash looks for variable "FILE_PREFIX1"!
touch ${FILE_PREFIX}1.txt    # ← correct: creates "file1.txt"
```

### 📝 Storing Command Outputs in Variables

```bash
HOST_NAME=$(hostname)
echo "Machine name is $HOST_NAME"

# Or with backticks (old style — avoid):
HOST_NAME=`hostname`         # works but not recommended
```

The `$(...)` syntax (subcommand from Day 4) captures a command's output and stores it in the variable.

### 🔒 Constants (`readonly`)

Prevent variables from being modified later in the script:

```bash
readonly DEVOPS="LINUX"
DEVOPS="CHANGED"    # ← bash: DEVOPS: readonly variable (error!)
```

> 🧭 **DevOps use case:** Use `readonly` for configuration values that should never change during script execution — like version numbers, API endpoints, or critical paths. If something tries to modify them, the script errors out loudly instead of silently behaving differently.

### 📝 Modifying Variables

Variables can be reassigned dynamically during script execution:

```bash
NAME="Manish"
echo $NAME        # Manish
NAME="Lulu"
echo $NAME        # Lulu
```

---

## 6️⃣ Special Variables & Arguments

Bash provides **special variables** that give you information about how the script was invoked.

### 📊 The Special Variables

| Variable | What it holds |
|---|---|
| **`$0`** | The name of the script itself (the command that was run) |
| **`$1`** | The 1st argument passed to the script |
| **`$2`** | The 2nd argument |
| **`$@`** | All arguments as a list |
| **`$#`** | Number of arguments passed |
| **`$$`** | The script's own PID |
| **`$?`** | Exit code of the last command (recall Day 3) |

### 📝 Using Arguments

```bash
#!/bin/bash

# $0 is the script name, $1 is the first argument
DESTINATION=$1
echo "Script name: $0"
echo "Destination: $DESTINATION"
```

```bash
$ ./mybash.sh /tmp/myfiles
Script name: ./mybash.sh
Destination: /tmp/myfiles
```

> 🧭 **Best practice:** Assign `$1` to a named variable immediately (`DESTINATION=$1`). Using `$1` throughout the script makes it hard to read; `DESTINATION` is self-documenting.

### 📝 Interactive Prompts with `read`

```bash
# Basic input
echo "What is your name?"
read NAME

# Interactive prompt (combines message + input on one line)
read -r -p "Enter your name: " NAME # -r prevents backslashes from being interpreted as escape characters.
echo "Your name is $NAME"
```

The **`-p`** flag combines the prompt message and input reading into a single line — much cleaner than `echo` + `read`.

### 📝 Full Example: Arguments + Prompts

```bash
#!/bin/bash

DESTINATION=$1                              # from command-line argument
read -p "enter a file prefix: " FILE_PREFIX # from interactive prompt

mkdir -p "$DESTINATION" # Unquoted expansions can undergo word splitting and pathname expansion. This can produce unexpected results.
cd "$DESTINATION"
touch ${FILE_PREFIX}{1..10}.txt
echo "done creating files in $DESTINATION"
```



---

## 7️⃣ Arrays

### 📝 Indexed Arrays

Arrays let you store multiple values in one variable. Use **parentheses `()`** to create an array:

```bash
# Define an array
creamheros=(Lulu Lala "President MoMo" ChuChu "Dr. CoCo" Dung-Tung TT)

# Access a specific element (zero-indexed)
echo ${creamheros[0]}        # Lulu
echo ${creamheros[2]}        # President MoMo

# Print ALL elements
echo ${creamheros[*]}        # all items
echo ${creamheros[@]}        # same (preferred for loops)

# Get the LENGTH of the array
echo ${#creamheros[@]}       # 7
```

### 📊 Breaking Down `${#creamheros[@]}`

```
   ${#creamheros[@]}
     │  │         │
     │  │         └── [@] = select ALL items
     │  └──────────── creamheros = array name
     └─────────────── # = length operator (count items)
```

| Symbol | Meaning |
|---|---|
| `$` | Look up a variable |
| `creamheros` | Name of the array |
| `[@]` | Selects all items inside the array |
| `#` | Length operator — counts how many items |

> 🧭 **DevOps use case:** Constantly used to validate lists — e.g., counting active nodes (`active_nodes=${#servers[@]}`) or checking if backup files list is empty (`0`).

### 📝 Associative Arrays (Key-Value)

Declare and initialize key-value pairs:

```bash
declare -A MY_ASSOC_ARRAY
MY_ASSOC_ARRAY=( [name]="Manish" [age]=22 [city]="Kathmandu" )

echo "Name is ${MY_ASSOC_ARRAY[name]}"      # Manish
echo "City is ${MY_ASSOC_ARRAY[city]}"      # Kathmandu
```

The `declare -A` flag tells Bash this is an **A**ssociative array (key-value, not indexed by number).

### 📝 Looping Through an Array

```bash
#!/bin/bash

creamheros=(Lulu Lala "President MoMo" ChuChu "Dr. CoCo" Dung-Tung TT)

# Loop safely (quotes preserve spaces in elements like "President MoMo")
for cat in "${creamheros[@]}"
do
    echo "Giving churu snack to: $cat"
done

echo "Butler successfully fed all ${#creamheros[@]} cats!"
```

> ⚠️ Always quote `"${array[@]}"` in loops. Without quotes, elements with spaces (like `"President MoMo"`) would split into separate words.

---

## 8️⃣ String Operations

Bash has built-in string manipulation — no need for `awk` or `sed` for simple operations:

| Operation | Syntax | Description |
|---|---|---|
| **Length** | `${#MY_VAR}` | Returns the character count of the string |
| **Uppercase** | `${MY_VAR^^}` | Converts string to uppercase |
| **Lowercase** | `${MY_VAR,,}` | Converts string to lowercase |
| **Replace (first)** | `${MY_VAR/bdy/manish}` | Replaces the **first** occurrence of `bdy` with `manish` |
| **Slicing** | `${MY_VAR:4:5}` | Extracts a substring starting at index 4, length 5 |

### 📋 Examples

```bash
MY_VAR="hello buddy"

echo ${#MY_VAR}           # 12 (length)
echo ${MY_VAR^^}          # HELLO BUDDY (uppercase)
echo ${MY_VAR,,}          # hello buddy (lowercase)
echo ${MY_VAR/bdy/manish}   # hello manish (replace first occurrence)
echo ${MY_VAR:6:4}        # budd (slice from index 6, length 4)
```

> 🧭 **DevOps use case:** String operations are how you parse filenames, normalize input, and manipulate paths in scripts. `${FILE^^}` converts a filename to uppercase; `${FILE%.txt}` strips the `.txt` extension.

---

## 9️⃣ Arithmetic Operations

Bash isn't great at math by default — you need special syntax.

### 📝 Using `let`

```bash
let sum=10+2
echo $sum        # 12

x=5
y=3
let mult=x*y
echo $mult       # 15
```

### 📝 Using Double Parentheses `$(( ... ))` — Preferred

```bash
echo "Subtraction is $((10 - 2))"     # Subtraction is 8
echo "Addition is $((5 + 3))"         # Addition is 8
echo "Multiplication is $((4 * 6))"   # Multiplication is 24
echo "Division is $((20 / 6))"        # Division is 3 (integer only!)
echo "Modulo is $((20 % 6))"          # Modulo is 2
```

The **double parentheses `(( ... ))`** convert to the `let` program internally. This is the preferred syntax because it's cleaner and supports all arithmetic operators.

### 📝 Random Numbers

```bash
# $RANDOM gives a random number 0-32767
echo $RANDOM

# Get a random number between 1 and 10
NUM_TO_GUESS=$(( $RANDOM % 10 + 1 ))
echo $NUM_TO_GUESS    # e.g., 7
```

> 💡 Modulo (`%`) doesn't work in plain Bash — you need `$(( ))` or `let`. This is why you see `$(( $RANDOM % 10 + 1 ))` for random number generation.

---

## 🔟 Control Structures — `if` / `elif` / `else`

### 📝 The `if` Statement

```bash
if [ condition ]; then
    echo "condition was true"
fi
```

The **`fi`** (which is "if" spelled backwards) closes the statement.

### 📊 The Square Brackets `[ ]` — Syntax Sugar for `test`

Here's something that confuses everyone: the square brackets `[ ]` are just **syntax sugar for the `test` command**.

```
   ┌──────────────────────────────────────────────────────────────┐
   │  SQUARE BRACKETS = SYNTAX SUGAR FOR test                     │
   │                                                              │
   │  if [ -z $DESTINATION ]; then                                │
   │      ↑                                                       │
   │      bash rearranges this into:                              │
   │                                                              │
   │  test -z $DESTINATION                                        │
   │      │                                                       │
   │      ├── returns 0 (true) if DESTINATION is empty            │
   │      └── returns 1 (false) if DESTINATION has content        │
   │                                                              │
   │  The if checks the exit code ($?):                           │
   │  - 0 = true → run the then block                             │
   │  - non-zero = false → skip the then block                    │
   └──────────────────────────────────────────────────────────────┘
```

### 📋 The `test` Command — Common Flags

You can test these directly before putting them in an `if`:

```bash
# String tests
test -z ""          && echo "empty"     # true (string is empty)
test -z "hello"     && echo "empty"     # false
test "a" = "a"      && echo "equal"     # true

# Numeric tests
test 15 -eq 15      && echo "equal"     # true (equal)
test 15 -gt 10      && echo "greater"   # true (greater than)
test 15 -lt 10      && echo "less"      # false (less than)

# File tests
test -e ~/output.txt && echo "exists"   # true if file exists
test -w ~/output.txt && echo "writable" # true if file exists AND writable
test -f ~/output.txt && echo "is file"  # true if it's a regular file
test -d ~/temp       && echo "is dir"   # true if it's a directory
```

> 💡 Run `man test` to see all available tests. Always test your condition interactively (`test ... && echo true`) before embedding it in an `if` statement — much faster than debugging a script that behaves wrong.

### 📝 Full `if` / `elif` / `else` Example

```bash
#!/bin/bash

if [ $1 -gt 10 ]; then
    echo "greater than ten"
elif [ $1 -lt 10 ]; then
    echo "less than 10"
else
    echo "equals 10"
fi
```

```bash
$ ./check_number 13
greater than ten

$ ./check_number 5
less than 10

$ ./check_number 10
equals 10
```

### 📝 Practical Example: Defaulting an Empty Argument

```bash
#!/bin/bash

DESTINATION=$1

if [ -z $DESTINATION ]; then    # -z = "is this string empty?"
    echo "no path provided, defaulting to ~/temp"
    DESTINATION=~/temp
fi

mkdir -p $DESTINATION
cd $DESTINATION
echo "working in $DESTINATION"
```

---

## 1️⃣1️⃣ Case Statements

The `case` statement is Bash's version of a **switch statement** — cleaner than a long chain of `elif` when checking one variable against many values.

```bash
#!/bin/bash

case $1 in
    "smile")
        echo ":)"
        ;;
    "sad")
        echo ":("
        ;;
    "laugh")
        echo ":D"
        ;;
    *)
        echo "I don't know that one yet!"
        ;;
esac
```

### 📊 Case Statement Anatomy

```
   case $1 in                    ← start case, check $1
       "smile")                  ← first pattern
           echo ":)"             ← what to do
           ;;                    ← double semicolon = end of this case
       "sad")
           echo ":("
           ;;
       *)
           echo "default"        ← * = wildcard (default case)
           ;;
   esac                          ← "case" backwards = end case
```

| Part | Meaning |
|---|---|
| **`case $VAR in`** | Start the case, check `$VAR` |
| **`"pattern")`** | The value to match (closing paren marks the pattern end) |
| **`;;`** | End of this case block (like `break` in C/Java) |
| **`*)`** | Wildcard — the default/fallthrough case |
| **`esac`** | Close the case statement ("case" spelled backwards) |

```bash
$ ./smiley laugh
:D

$ ./smiley cry
I don't know that one yet!
```

> 🧭 **DevOps use case:** Case statements are perfect for handling command-line flags or menu-driven scripts. `case $1 in start) ... ;; stop) ... ;; restart) ... ;; esac` is how many init scripts work.

---

## 1️⃣2️⃣ Loops

### 📝 `for` Loops

```bash
# Loop over an array
for cat in "${creamheros[@]}"
do
    echo "Feeding: $cat"
done

# Loop over a range
for i in {1..5}
do
    echo "Number $i"
done

# Loop over files
for file in *.txt
do
    echo "Processing $file"
done
```

### 📝 `while` Loops

```bash
#!/bin/bash

NUM_TO_GUESS=$(( $RANDOM % 10 + 1 ))
GUESSED_NUM=0

echo "guess a number between 1 and 10"

while [ $NUM_TO_GUESS -ne $GUESSED_NUM ]
do
    read -p "your guess: " GUESSED_NUM
done

echo "you got it!"
```

The `while` loop runs **as long as the condition is true**. Here it keeps prompting until the user guesses the right number.

### 📊 Loop Anatomy

| Loop | When to use |
|---|---|
| **`for`** | When you know the items to iterate over (array, range, file list) |
| **`while`** | When you want to repeat until a condition changes (user input, polling) |

> 💡 Functions in Bash: You *can* write functions in Bash, but when logic gets complex enough to need functions, most DevOps engineers reach for **Python** instead. Bash is procedural by convention; Python is better for complex logic.

---

## 1️⃣3️⃣ PATH & Making Scripts System-Wide

### 📍 The `PATH` Variable

`PATH` contains a **colon-separated list** of directories where executable programs are stored. Bash searches these directories **left to right** to find and execute commands.

```bash
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
```

```
   ┌──────────────────────────────────────────────────────────────┐
   │  HOW BASH FINDS A COMMAND:                                   │
   │                                                              │
   │  You type: $ node                                            │
   │      │                                                       │
   │      ▼                                                       │
   │  Bash reads PATH left to right:                              │
   │  /usr/local/sbin/node?  ❌ not found                         │
   │  /usr/local/bin/node?   ❌ not found                         │
   │  /usr/sbin/node?        ❌ not found                         │
   │  /usr/bin/node?         ❌ not found                         │
   │  /snap/bin/node?        ✅ FOUND! → runs /snap/bin/node      │
   └──────────────────────────────────────────────────────────────┘
```

### 📝 The `./` Prefix

When you run `./mybash`, the `./` tells Bash: "don't search PATH — run this file from the **current directory**."

```bash
$ ./mybash           # run from current directory
$ mybash             # bash searches PATH for "mybash"
```

### 📝 Making Your Script a System-Wide Command

What if you want to run `mybash` from **anywhere**, like a real system command?

```bash
# 1. Create your own directory for personal scripts
$ mkdir ~/my_bin

# 2. Move your script there (and remove the .sh extension)
$ mv mybash ~/my_bin/

# 3. Add ~/my_bin to your PATH
$ vi ~/.bashrc
    export PATH=~/my_bin:$PATH    # prepend my_bin to PATH
# :wq

# 4. Reload .bashrc
$ source ~/.bashrc

# 5. Now run it from anywhere!
$ cd /tmp
$ mybash            # ✅ works! bash finds it in ~/my_bin
```

```
   ┌──────────────────────────────────────────────────────────────┐
   │  ADDING TO PATH:                                             │
   │                                                              │
   │  PATH=~/my_bin:$PATH                                         │
   │         │       │                                            │
   │         │       └── existing PATH (appended after colon)     │
   │         └────────── your new directory (prepended)           │
   │                                                              │
   │  The colon (:) separates entries.                            │
   │  Without :$PATH, you'd OVERWRITE the entire variable!        │
   │                                                              │
   │  Result: ~/my_bin:/usr/local/bin:/usr/bin:/bin:...           │
   │          ↑ your dir is now searched FIRST                    │
   └──────────────────────────────────────────────────────────────┘
```

> ⚠️ **Critical:** Always append `:$PATH` when modifying PATH. Without it, you overwrite the entire variable and break every system command. `export PATH=~/my_bin` (no `:$PATH`) = disaster.

> 🧭 **DevOps use case:** This is how you create personal utility scripts that behave like system commands. Add `~/my_bin` to PATH, drop your scripts there, and they're available everywhere — with tab completion too! (Type `myb` + Tab → `mybash` auto-completes.)

---

## 1️⃣4️⃣ `mkdir -p` — The Useful Flag

The **`-p`** flag does two things for `mkdir`:
1. **Prevents errors** if the directory already exists
2. **Creates parent directories** as needed

```bash
# Without -p: errors if any parent doesn't exist
$ mkdir -p a/b/c      # creates a/, then a/b/, then a/b/c/ — no errors

# Without -p: would fail if a/ doesn't exist
$ mkdir a/b/c         # mkdir: cannot create directory 'a/b/c': No such file or directory
```

This is why you see `mkdir -p` in almost every script — it's idempotent (safe to run multiple times) and creates nested paths in one command.

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **Shell** | User interface between you and the kernel. Bash is the most common; zsh is macOS default. |
| **`echo $0`** | Check your current shell. |
| **`cat /etc/shells`** | List all available shells on the system. |
| **Shebang `#!/bin/bash`** | First line — specifies the interpreter. `#!` is special (not a comment). |
| **`.sh` extension** | Not mandatory but strongly recommended as convention. |
| **`chmod +x script.sh`** | Give execute permission before running `./script.sh`. |
| **Four ways to run** | `./script.sh` (relative), `/path/script.sh` (absolute), `bash script.sh` (interpreter), system-wide (after PATH setup). |
| **`source` vs `bash`** | `source`/`.` runs in current process (changes persist). `bash` creates subprocess (changes don't persist). |
| **`. script.sh`** | Dot notation — works in more distros than `source`. |
| **Single-line comment** | `#` — everything after is ignored. |
| **Multi-line comment** | `: << 'COMMENT' ... COMMENT` (here-doc technique). |
| **Variable assignment** | `NAME="value"` — NO spaces around `=`. |
| **Variable access** | `$NAME` or `${NAME}` (braces for ambiguity). |
| **Command output in var** | `HOST_NAME=$(hostname)` — subcommand syntax. |
| **`readonly`** | Make a variable a constant — can't be modified later. |
| **`$0`** | Script name itself. |
| **`$1`, `$2`, ...** | Command-line arguments. |
| **`$@`** | All arguments as a list. |
| **`$#`** | Number of arguments. |
| **`read -p "prompt" VAR`** | Interactive input with a prompt message. |
| **Indexed array** | `arr=(a b c)` — access with `${arr[0]}`, all with `${arr[@]}`, length with `${#arr[@]}`. |
| **Associative array** | `declare -A arr` — key-value: `arr=([name]="Manish")`, access `${arr[name]}`. |
| **String length** | `${#VAR}` |
| **String uppercase** | `${VAR^^}` |
| **String lowercase** | `${VAR,,}` |
| **String replace** | `${VAR/old/new}` — replaces first occurrence. |
| **String slice** | `${VAR:start:length}` |
| **Arithmetic** | `$(( ... ))` — preferred syntax. Also `let`. |
| **`$RANDOM`** | Random number 0-32767. Use `$(( $RANDOM % N + 1 ))` for 1-N. |
| **`if [ condition ]; then ... fi`** | If statement. `fi` = "if" backwards. |
| **`[ ]` brackets** | Syntax sugar for `test` command. Returns 0=true, 1=false. |
| **`-z`** | test flag: "is string empty?" |
| **`-e`** | test flag: "does file exist?" |
| **`-w`** | test flag: "does file exist AND is it writable?" |
| **`-eq`, `-gt`, `-lt`** | Numeric tests: equal, greater than, less than. |
| **`man test`** | See all test conditions. |
| **`elif`** | Else-if branch. |
| **`case $VAR in ... esac`** | Switch statement. `;;` ends each case, `*)` is default, `esac` = "case" backwards. |
| **`for item in list; do ... done`** | For loop — iterate over arrays, ranges, file lists. |
| **`while [ condition ]; do ... done`** | While loop — repeat until condition is false. |
| **PATH** | Colon-separated list of directories bash searches for commands (left to right). |
| **`export PATH=~/my_bin:$PATH`** | Prepend a directory to PATH. NEVER forget `:$PATH` or you overwrite it. |
| **`./script`** | Run from current directory (don't search PATH). |
| **`mkdir -p`** | Create nested dirs without errors if they exist. Idempotent. |
| **Functions in Bash** | Possible, but for complex logic, use Python instead. |
| **DevOps relevance** | Shell scripting = automation language. Cron jobs, deployment scripts, batch operations — all Bash. Master it to automate everything. |