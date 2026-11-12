# 🔐 Permissions & Ownership — `chmod` + `chown`

> **Goal:** Decode the 10-character permission string, master `chmod` (both symbolic and octal modes), know the production baseline permission numbers (400/600/644/700/755), understand why `777` is forbidden, and know exactly when to use `chmod` vs `chown`.

---

## 📍 The Heart of Linux Security

This is the most important note in the entire Day 5 set. Every "Permission denied" error, every deployment failure, every security hardening checklist — they all come back to two commands:

- **`chmod`** (change mode) — controls *what actions* are allowed (read, write, execute)
- **`chown`** (change owner) — controls *who owns* the file

Master these two and you've mastered 80% of Linux security operations.

---

## 1️⃣ `ls -l` — Reading the Permission String

The command `ls` lists files. Adding the **`-l`** flag gives **long format** — a detailed breakdown including ownership, size, modification time, and **permissions**.

```bash
$ ls -l
-rwxr-xr--  1 john developers  4096 Nov 5 10:30 script.sh
```

That very first 10-character chunk on the far left (`-rwxr-xr--`) is the **permission and file type string**. This is your map to understanding who can do what with that file.

### 📊 The Full `ls -l` Output Annotated

```
   -rwxr-xr--  1 john developers  4096 Nov 5 10:30 script.sh
   │          │  │    │          │      │             │
   │          │  │    │          │      │             └── filename
   │          │  │    │          │      └── modification time
   │          │  │    │          └── size (bytes)
   │          │  │    └── group (developers)
   │          │  └── owner (john)
   │          └── hard link count
   └── permission + file type string (10 chars)
```

---

## 2️⃣ Deconstructing the 10-Character String

The string is divided into **four logical parts**:

```
   - rwx r-x r--
   │  │   │   │
   │  │   │   └── Others (everyone else)
   │  │   └────── Group
   │  └────────── Owner
   └───────────── File Type
```

### 📁 Part 1: File Type (Character 1)

The very first character tells you what kind of object you're looking at:

| Character | Meaning |
|---|---|
| **`-`** | A **standard file** (text, script, image, etc.) |
| **`d`** | A **directory** (folder) |
| **`l`** | A **symbolic link** (a shortcut pointing to another file) |

### 👤 Part 2: Owner Permissions (Characters 2–4)

Applies to the specific user who owns the file (in our example, `john`).

In our example (`rwx`): the owner has **r**ead, **w**rite, and **e**xecute permissions.

### 👥 Part 3: Group Permissions (Characters 5–7)

Applies to a collection of users grouped together (in our example, the `developers` group). Every file belongs to a primary group, and any user in that group shares these permissions.

In our example (`r-x`): group members have **r**ead and **e**xecute, but **no write** (the `-`).

### 🌍 Part 4: Others / World Permissions (Characters 8–10)

Applies to everyone else on the system who is neither the owner nor in the file's group.

In our example (`r--`): everyone else only has **read** permission. They cannot modify or execute it.

### ✅ Summary Check

For `-rwxr-xr--`:

1. It's a **file** (`-`)
2. The **owner** can read, write, and execute it (`rwx`)
3. The **group** can read and execute it, but not change it (`r-x`)
4. **Everyone else** can only read it (`r--`)

```
   ┌──────────────────────────────────────────────────────────────┐
   │              DECODING -rwxr-xr--                             │
   │                                                              │
   │   -   rwx    r-x    r--                                      │
   │   │    │      │      │                                       │
   │   │    │      │      └── Others: read only                   │
   │   │    │            (can't write, can't execute)             │
   │   │    └─────── Group: read + execute                        │
   │   │                 (can't write — the dash blocks it)       │
   │   └────────────── Owner: read + write + execute (full)       │
   │                                                              │
   │   File type: regular file (-)                                │
   │                                                              │
   │   In plain English:                                          │
   │   "john can do everything, developers can run & read it,     │
   │    everyone else can just read it."                          │
   └──────────────────────────────────────────────────────────────┘
```

---

## 3️⃣ What `r`, `w`, and `x` Actually Mean

The letters always stand for the same three actions, but **how they behave depends on whether the item is a file or a directory**:

| Permission | Letter | For a **File** | For a **Directory** |
|---|---|---|---|
| **Read** | `r` | Can open and read the file's contents | Can **list** the files inside (e.g., `ls`) |
| **Write** | `w` | Can modify, save, or delete the file's contents | Can **create, delete, or rename** files *inside* that directory |
| **Execute** | `x` | Can **run** the file as a program or script | Can **"enter"** or access the directory (e.g., `cd`) |

### ⚠️ The Directory Trap

This is the #1 confusion point for beginners:

```
   ┌──────────────────────────────────────────────────────────────┐
   │  DIRECTORY PERMISSIONS — DIFFERENT FROM FILES!               │
   │                                                              │
   │  On a DIRECTORY:                                             │
   │  - r  = can LIST contents (ls)                               │
   │  - w  = can CREATE/DELETE files inside it                    │
   │  - x  = can ENTER it (cd into it)                            │
   │                                                              │
   │  On a FILE:                                                  │
   │  - r  = can READ contents (cat)                              │
   │  - w  = can MODIFY contents (edit)                           │
   │  - x  = can RUN it (./script.sh)                             │
   │                                                              │
   │  Common gotcha:                                              │
   │  If a directory has r but NOT x, you can ls it but can't     │
   │  cd into it. You need BOTH r and x to use a directory.       │
   │                                                              │
   │  Another gotcha:                                             │
   │  If you have w on a directory but NOT on a file inside it,   │
   │  you can DELETE the file (because w on dir = delete files    │
   │  inside) even though you can't edit its contents!            │
   └──────────────────────────────────────────────────────────────┘
```

> 🧭 **DevOps gotcha:** This is why `755` (`rwxr-xr-x`) is the standard for directories — the owner can do everything, others can enter (`x`) and list (`r`) but can't create/delete files inside. And `644` (`rw-r--r--`) is standard for files inside those directories.

---

## 4️⃣ `chmod` — Symbolic Mode (Letters)

**`chmod`** stands for **"change mode"**. It grants or revokes permissions (`r`, `w`, `x`).

### 📝 The Formula

```
   chmod [who][action][permission] filename
         │      │       │
         │      │       └── r, w, or x (what to change)
         │      └────────── +, -, or = (how to change it)
         └───────────────── u, g, o, or a (who to affect)
```

### 👥 Who (`[who]`)

| Letter | Target |
|---|---|
| **`u`** | **U**ser (the file owner) |
| **`g`** | **G**roup (members of the file's group) |
| **`o`** | **O**thers (everyone else) |
| **`a`** | **A**ll (`u` + `g` + `o`) |

### 🎯 Action (`[action]`)

| Symbol | Meaning |
|---|---|
| **`+`** | **Add** this permission |
| **`-`** | **Remove** this permission |
| **`=`` | **Set exact** permission (overwrites everything else for that target) |

### 🔧 Permission (`[permission]`)

`r` (read), `w` (write), `x` (execute).

### 📋 Symbolic Mode Examples

```bash
# Add write permission for Others
chmod o+w file.txt
# Translation: Others (o) + Add (+) write (w) → anyone can now edit file.txt
# ⚠️ Be careful on shared servers!

# Remove ALL permissions from Others
chmod o-rwx file.txt
# Translation: Others (o) - Remove (-) rwx → complete lockdown for outsiders

# Set exact permissions in one command
chmod u=rwx,g=rw,o=r file.txt
# Translation: Owner gets rwx, Group gets rw-, Others gets r--
# Useful when you want to explicitly enforce a multi-tier structure

# Recursive — apply to directory and everything inside
chmod -R u=rwx directory/
# The -R flag applies to the directory AND all subdirectories and files
```

```
   ┌──────────────────────────────────────────────────────────────┐
   │  SYMBOLIC MODE EXAMPLES:                                     │
   │                                                              │
   │  chmod o+w file.txt        Others can now WRITE file.txt     │
   │  chmod o-rwx file.txt      Others lose ALL access            │
   │  chmod u+x deploy.sh       Owner can now EXECUTE deploy.sh   │
   │  chmod g-w config.yaml     Group loses WRITE on config       │
   │  chmod u=rwx,g=rx,o=r f    Set exact: owner=rwx, grp=rx, o=r │
   │  chmod -R u+rwx dir/       Recursive: owner gets rwx on all  │
   │  chmod a+r index.html      ALL can read (a = all)            │
   └──────────────────────────────────────────────────────────────┘
```

---

## 5️⃣ `chmod` — Octal (Numeric) Mode

Instead of typing letters, DevOps engineers frequently use **3-digit numbers**. This is based on adding up the values of the permissions:

### 🔢 The Number Values

| Permission | Value |
|---|---|
| **Read (`r`)** | **4** |
| **Write (`w`)** | **2** |
| **Execute (`x`)** | **1** |

To find a digit, just **add them together**:

```
   ┌──────────────────────────────────────────────────────────────┐
   │              OCTAL PERMISSION CALCULATOR                     │
   │                                                              │
   │   rwx  = 4 + 2 + 1 = 7  (full)                               │
   │   rw-  = 4 + 2 + 0 = 6  (read + write)                       │
   │   r-x  = 4 + 0 + 1 = 5  (read + execute)                     │
   │   r--  = 4 + 0 + 0 = 4  (read only)                          │
   │   -wx  = 0 + 2 + 1 = 3  (write + execute, rare)              │
   │   -w-  = 0 + 2 + 0 = 2  (write only, very rare)              │
   │   --x  = 0 + 0 + 1 = 1  (execute only, rare)                 │
   │   ---  = 0 + 0 + 0 = 0  (no access)                          │
   │                                                              │
   │   THREE DIGITS = [Owner] [Group] [Others]                    │
   │                                                              │
   │   chmod 755  →  Owner=7(rwx)  Group=5(r-x)  Others=5(r-x)    │
   │   chmod 644  →  Owner=6(rw-)  Group=4(r--)   Others=4(r--)   │
   │   chmod 600  →  Owner=6(rw-)  Group=0(---)   Others=0(---)   │
   │   chmod 400  →  Owner=4(r--)  Group=0(---)   Others=0(---)   │
   └──────────────────────────────────────────────────────────────┘
```

### 📋 Octal Examples

```bash
chmod 755 script.sh      # rwxr-xr-x — owner full, others read+execute
chmod 644 index.html     # rw-r--r-- — owner write, others read
chmod 600 secret.env     # rw------- — owner only, everyone else locked out
chmod 400 key.pem        # r-------- — owner can only READ (for SSH keys)
chmod 700 ~/.ssh          # rwx------ — owner only, full access to .ssh dir
```

> 💡 **Pro tip:** Once you memorize that r=4, w=2, x=1, you can decode any 3-digit permission in your head. `755`? 7=rwx, 5=r-x, 5=r-x → `rwxr-xr-x`. `640`? 6=rw-, 4=r--, 0=--- → `rw-r-----`. It becomes second nature.

---

## 6️⃣ Production Baselines — The Numbers You Must Know

These are the permission numbers every DevOps engineer must recognize on sight:

| Octal | Symbolic | Use Case | Why? |
|---|---|---|---|
| **`400`** | `-r--------` | **AWS EC2 / SSH Private Keys (`.pem`)** | SSH clients (like OpenSSH) will **strictly reject** your private key if group or others have any access. Must be locked down so *only the owner can read it*. |
| **`600`** | `-rw-------` | **App Configs / DB Secrets** | `.env` files, database credentials, SSH `authorized_keys`. Only the service user running the app can read/write them. |
| **`644`** | `-rw-r--r--` | **Standard Files & Static Web Pages** | Owner can modify (`6`), group and others can only read (`4`). Perfect for HTML/CSS files served by Nginx or Apache. |
| **`700`** | `-rwx------` | **Private Scripts & User Binaries** | Shell scripts (`deploy.sh`) in your home directory that only you should run and modify. |
| **`755`** | `-rwxr-xr-x` | **Directories & Executable Binaries** | Owner can do everything (`7`), others can enter and list contents (`5`). |

### 📊 Why `755` on Directories?

**Why read/execute on directories for others?**

- **Execute (`x`)** lets you `cd` into the directory
- **Read (`r`)** lets you `ls` the files inside it

Without both, outsiders can't traverse or inspect the folder. For a web server's document root, you want visitors (running as `www-data`) to be able to enter the directory and read files — so `755` is correct.

```
   ┌──────────────────────────────────────────────────────────────┐
   │  PRODUCTION PERMISSION CHEAT SHEET:                          │
   │                                                              │
   │  SSH private key (.pem)         → 400  (r--------)           │
   │  .env / secrets file            → 600  (rw-------)           │
   │  HTML/CSS/JS web files          → 644  (rw-r--r--)           │
   │  Your deploy.sh script          → 700  (rwx------)           │
   │  /var/www/html directory        → 755  (rwxr-xr-x)           │
   │  ~/.ssh directory               → 700  (rwx------)           │
   │  ~/.ssh/authorized_keys         → 600  (rw-------)           │
   │  /etc/nginx/nginx.conf          → 644  (rw-r--r--)           │
   └──────────────────────────────────────────────────────────────┘
```

---

## 7️⃣ The Anti-Pattern: Never `777` in Production

You will see tutorials or desperate developers online suggesting:

> *"My script says permission denied! Just run `chmod 777 script.sh`!"*

**Never do this in production.**

```
   ┌──────────────────────────────────────────────────────────────┐
   │              ☠️  THE 777 ANTI-PATTERN  ☠️                    │
   │                                                              │
   │  chmod 777 file = rwxrwxrwx                                  │
   │                                                              │
   │  EVERYONE can:                                               │
   │  - READ the file                                             │
   │  - WRITE/modify the file                                     │
   │  - EXECUTE the file                                          │
   │                                                              │
   │  Why it's dangerous:                                         │
   │  If your app has a vulnerability, ANY malicious user or      │
   │  process on that server can:                                 │
   │  - Modify your code (inject malware)                         │
   │  - Execute arbitrary payloads                                │
   │  - Overwrite your scripts                                    │
   │                                                              │
   │  ✅ Instead, apply the Principle of Least Privilege:         │
   │  - 755 for directories and executables                       │
   │  - 644 for regular files                                     │
   │  - 600 for secrets                                           │
   │  - 400 for SSH keys                                          │
   └──────────────────────────────────────────────────────────────┘
```

Always apply the **Principle of Least Privilege** — the minimum permissions that still allow the job to work. `777` is the opposite of that.

> 🧭 **DevOps rule:** If you ever type `chmod 777`, stop and ask: "Why is this permission denied?" Fix the root cause (wrong owner, wrong group, missing execute bit) — don't nuke the entire permission model.

---

## 8️⃣ `chown` — Changing Ownership

While `chmod` controls *what actions* can be performed, **`chown` controls *who owns* the file or directory**. Every file is bound to a specific **User** and a specific **Group**.

### 📝 The Anatomy

```
   sudo chown [options] new_owner:new_group filename
        │              │       │          │
        │              │       │          └── target file/dir
        │              │       └── optional group (after colon)
        │              └── new owner username
        └── almost always needs sudo (transferring resources between users)
```

### 📋 Syntax Variations

```bash
# Change ONLY the owner
sudo chown alice app.py
# Result: alice owns it, group unchanged

# Change owner AND group
sudo chown alice:developers app.py
# Result: owner=alice, group=developers

# Change ONLY the group (leave user blank before colon)
sudo chown :developers app.py
# Result: owner unchanged, group=developers
# (Alternatively: sudo chgrp developers app.py)

# Recursive — apply to directory AND everything inside
sudo chown -R www-data:www-data /var/www/html
# Result: www-data owns the entire web root tree
```

### 📊 `chown` vs `chmod` — The Golden Rule

| Use `chown` when... | Use `chmod` when... |
|---|---|
| The process has the right permissions (`rwx`), but belongs to the **wrong user or group** | The user/group is correct, but they **lack a specific permission bit** (like missing `x` on a script) |
| You're transferring ownership between users | You're adjusting what the current owner can do |
| Example: Nginx needs to own `/var/www/uploads` | Example: `deploy.sh` needs execute permission |

```
   ┌──────────────────────────────────────────────────────────────┐
   │  THE DIAGNOSTIC DECISION TREE:                               │
   │                                                              │
   │  "Permission denied" on a file. Ask yourself:                │
   │                                                              │
   │  Q1: Is it the WRONG USER/GROUP trying to access it?         │
   │      → YES: use sudo chown to fix ownership                  │
   │                                                              │
   │  Q2: Does the RIGHT USER lack the correct r/w/x bit?         │
   │      → YES: use chmod (or sudo chmod) to adjust permissions  │
   │                                                              │
   │  Q3: Do you need root privileges to do this at all?          │
   │      → YES: use sudo (prefix the command)                    │
   └──────────────────────────────────────────────────────────────┘
```

---

## 9️⃣ Real DevOps Scenarios — `chmod` vs `chown` vs `sudo`

| Scenario | Tool | Why? |
|---|---|---|
| **A:** You cloned a git repo, but `./deploy.sh` says "Permission denied" | **`chmod +x deploy.sh`** | The file belongs to you, but lacks the execute bit. You just need to change the mode. |
| **B:** Web app (running as `www-data`) can't upload to `/var/www/uploads` | **`sudo chown -R www-data:www-data /var/www/uploads`** | Nginx doesn't just need write permissions — it needs to **own** the directory to manage files properly. |
| **C:** You created a secret DB password file as `root`, but the app (running as `appuser`) needs to read it | **`sudo chmod 644 secret.env`** OR **`sudo chown appuser secret.env`** | Either give others read access (`644`) or reassign ownership to the app user (`chown`). |
| **D:** You need to modify `/etc/nginx/nginx.conf` (owned by root) | **`sudo chmod 600 /etc/nginx/nginx.conf`** | You don't own the file, so plain `chmod` fails. `sudo` overrides ownership restrictions. |
| **E:** SSH key rejected: "permissions too open" | **`chmod 400 my-key.pem`** | SSH strictly requires private keys to be `400` — only owner can read. Group/others must have zero access. |

### 🔧 The `sudo` + `chmod` Interaction

**The rule of who can change permissions:**
1. The **file owner** can change permissions on their own files using plain `chmod`
2. The **root user** can change permissions on **any file on the entire system**, regardless of who owns it

```bash
# You DON'T own this file → FAILS
chmod 600 /etc/nginx/nginx.conf
# ❌ chmod: changing permissions of '/etc/nginx/nginx.conf': Operation not permitted

# Sudo → root overrides ownership → SUCCEEDS
sudo chmod 600 /etc/nginx/nginx.conf
# ✅ Success
```

---

## 🔟 Quick Reference Cheat Sheet

### 📋 `chmod` Cheat Sheet

| Command | Effect |
|---|---|
| `chmod +x script.sh` | Add execute for all (shortcut for `a+x`) |
| `chmod u+x script.sh` | Add execute for owner only |
| `chmod o-w file.txt` | Remove write from others |
| `chmod u=rwx,g=rx,o=r file` | Set exact permissions |
| `chmod -R 755 directory/` | Recursive octal |
| `chmod 755 script.sh` | Owner full, group/others read+execute |
| `chmod 644 file.txt` | Owner read+write, others read |
| `chmod 600 secrets.env` | Owner only, read+write |
| `chmod 400 key.pem` | Owner read only (SSH keys) |
| `chmod 700 ~/.ssh` | Owner only, full access |

### 📋 `chown` Cheat Sheet

| Command | Effect |
|---|---|
| `sudo chown alice file` | Change owner to alice |
| `sudo chown alice:devs file` | Change owner=alice, group=devs |
| `sudo chown :devs file` | Change group only (to devs) |
| `sudo chown -R www-data:www-data /var/www` | Recursive ownership change |

### 🧠 Memory Hooks

- **`chmod`** = **ch**ange **mod**e (what actions: r/w/x)
- **`chown`** = **ch**ange **own**er (who owns it)
- **Octal values**: **r**ead=4, **w**rite=2, e**x**ecute=1 → add them
- **Production baselines**: 400/600/644/700/755 — memorize these five
- **`777`** = ☠️ forbidden in production

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **`ls -l`** | Shows the 10-char permission string + owner + group + size + date. |
| **10-char string** | `-rwxr-xr--` = file type + owner(rwx) + group(r-x) + others(r--). |
| **File types** | `-` = file, `d` = directory, `l` = symbolic link. |
| **`r` for file** | Can read contents. |
| **`r` for dir** | Can list contents (`ls`). |
| **`w` for file** | Can modify contents. |
| **`w` for dir** | Can create/delete/rename files INSIDE the directory. |
| **`x` for file** | Can run it as a program. |
| **`x` for dir** | Can enter it (`cd`). Need BOTH `r` and `x` to use a directory. |
| **`chmod` symbolic** | `[who][action][perm]` — `u`/`g`/`o`/`a` + `+`/`-`/`=` + `r`/`w`/`x`. |
| **`chmod` octal** | r=4, w=2, x=1. Add them per target. Three digits = owner/group/others. |
| **`400`** | SSH private keys — owner read only. SSH rejects if others have access. |
| **`600`** | App configs, `.env` files, DB secrets, `authorized_keys`. Owner read/write only. |
| **`644`** | Standard files, HTML/CSS/JS. Owner writes, others read. |
| **`700`** | Private scripts, `~/.ssh` directory. Owner only, full access. |
| **`755`** | Directories & executable binaries. Owner full, others can enter/list. |
| **`777`** | ☠️ NEVER in production. Everyone can read/write/execute — security disaster. |
| **`chmod -R`** | Recursive — apply to directory and everything inside it. |
| **`chown`** | Change *who owns* the file. `sudo chown user:group file`. Almost always needs sudo. |
| **`chown user:group`** | Change both owner and group. |
| **`chown :group`** | Change group only (leave user blank before colon). |
| **`chown -R`** | Recursive ownership change — directory and all contents. |
| **`chmod` vs `chown`** | `chmod` = what actions allowed. `chown` = who owns it. |
| **Owner can `chmod` own files** | File owner can change permissions on their own files (no sudo needed). |
| **Root can `chmod` anything** | Root can change permissions on ANY file, regardless of ownership. Needs `sudo`. |
| **Diagnostic questions** | Wrong user/group? → `chown`. Right user, missing bit? → `chmod`. Need root? → `sudo`. |
| **DevOps relevance** | Permission errors = #1 deployment failure. Master `ls -l`, `chmod`, `chown` to fix in minutes. |

---

← Back to [Permissions & Access Control](./README.md) | Previous: [Users, Groups & Superuser](./1-users-groups-sudo.md) | Next: [ACLs](./3-acls.md) →
