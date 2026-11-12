# 👤 Users, Groups & Superuser

> **Goal:** Understand Linux's multi-user model — who you are (`whoami`/`id`), how to create and manage users (`useradd`/`passwd`/`userdel`/`usermod`), the `root` user, `sudo`, and the **Principle of Least Power** that governs all DevOps security decisions.

---

## 📍 Linux Is Multi-User

Unlike Windows (which feels single-user by default), Linux was designed from day one as a **multi-user operating system**. Multiple people can be logged in simultaneously, running processes, reading files, and — crucially — **the system enforces boundaries between them**.

Every file on the system belongs to a specific **user** and a specific **group**. Every process runs as a specific user. The kernel enforces: you can only access files your user/group has permission for.

This is why Linux is the backbone of servers — it safely hosts hundreds of users and services on one machine.

---

## 1️⃣ Who Am I? — `whoami` and `id`

### `whoami`

Displays the name of the **currently logged-in user** — identifies which user account you're operating under in the terminal session.

```bash
$ whoami
manishbishwakarma
```

### `id`

Shows your **UID** (User ID), **GID** (Group ID), and all groups you belong to:

```bash
$ id
uid=1000(manish) gid=1000(manish) groups=1000(manish),27(sudo),999(docker)
```

| Field | Meaning |
|---|---|
| `uid=1000(manish)` | Your user ID is 1000, named `manish` |
| `gid=1000(manish)` | Your primary group ID is 1000 (your own group) |
| `groups=...` | All groups you belong to: `manish`, `sudo`, `docker` |

> 🧭 **DevOps tip:** The `id` command is how you verify "is this user in the right group?" before debugging permission issues. If a user can't run Docker, check `id` — are they in the `docker` group?

---

## 2️⃣ The User Database — `/etc/passwd`

Every user on the system is listed in `/etc/passwd`:

```bash
$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
manish:x:1000:1000:manish:/home/manish:/bin/bash
```

Each line has 7 fields, separated by `:`:

```
   manish:x:1000:1000:manish:/home/manish:/bin/bash
   │    │  │    │   │     │          │
   │    │  │    │   │     │          └── login shell (/bin/bash = interactive)
   │    │  │    │   │     └── home directory
   │    │  │    │   └── GECOS field (full name / comment)
   │    │  │    └── GID (primary group ID)
   │    │  └── UID (user ID)
   │    └── password placeholder (x = password in /etc/shadow)
   └── username
```

> 💡 Despite the name "passwd", passwords are NOT stored here anymore — they're in `/etc/shadow` (readable only by root). The `x` means "look in shadow."

### 🔒 Restricted Shells — `/bin/false` and `/usr/sbin/nologin`

Notice some users have `/bin/false` or `/usr/sbin/nologin` as their shell. These are **restricted shells** — when a user's login shell is set to `/bin/false`, the user **cannot log in interactively**. The program simply returns false, preventing interactive login.

```
   ┌──────────────────────────────────────────────────────────────┐
   │  SHELL TYPES IN /etc/passwd:                                 │
   │                                                              │
   │  /bin/bash          → interactive login (normal users)       │
   │  /bin/sh            → POSIX shell (lighter)                  │
   │  /bin/false         → CANNOT log in (returns false)          │
   │  /usr/sbin/nologin  → CANNOT log in (prints a message)       │
   │                                                              │
   │  Why? Service accounts (www-data, nginx, postgres) need      │
   │  to exist as users (to own files and run processes), but     │
   │  nobody should ever log in AS them.                          │
   └──────────────────────────────────────────────────────────────┘
```

This is how service accounts work — `www-data` exists to own web files and run the nginx process, but no one should ever SSH in as `www-data`.

---

## 3️⃣ Creating & Managing Users

### ➕ Create a User: `useradd`

```bash
$ sudo useradd -m -s /bin/bash lulu
```

| Flag | Meaning |
|---|---|
| **`-m`** | Create the **home directory** (`/home/lulu`) |
| **`-s /bin/bash`** | Set the login **shell** to bash |

Without `-m`, the user is created but has no home directory. Without `-s`, they might get the default shell (often `/bin/sh`), which is less friendly.

### 🔑 Set a Password: `passwd`

```bash
$ sudo passwd lulu
New password:
Retype new password:
passwd: password updated successfully
```

The user can change their own password with `passwd` (no sudo needed). Only root (or sudo) can set another user's password.

### ➖ Delete a User: `userdel`

```bash
$ sudo userdel -r lulu
```

| Flag | Meaning |
|---|---|
| **`-r`** | **Remove** the home directory and mail spool along with the user |

Without `-r`, the user account is deleted but their home directory remains (useful if you want to preserve their files).

### 👥 Add User to a Group: `usermod`

```bash
$ sudo usermod -aG wheel lulu     # add to sudo/wheel group
$ sudo usermod -aG docker lulu    # add to docker group
```

| Flag | Meaning |
|---|---|
| **`-aG`** | **A**ppend to **G**roup — adds the user to the group WITHOUT removing them from other groups |

> ⚠️ **Critical:** Always use `-aG` (append), not just `-G` (which would REPLACE all their groups with the one you specify, booting them from everything else). This is a common mistake that locks users out.

```bash
# ✅ CORRECT — adds to docker group, keeps all others
sudo usermod -aG docker lulu

# ❌ WRONG — sets lulu's groups to ONLY docker, removes from sudo, etc.
sudo usermod -G docker lulu
```

---

## 4️⃣ Groups — The Collective Permission Mechanism

Groups let you assign permissions to a **collection of users** at once. Instead of adding alice, bob, and carol individually to a file's permissions, you create a `developers` group, add them all to it, and give the group access.

### 📋 Common System Groups

| Group | Purpose |
|---|---|
| **`wheel`** / **`sudo`** | Members can use `sudo` (become root) |
| **`docker`** | Members can run Docker without sudo |
| **`www-data`** | Web server user (nginx, Apache) |
| **`ssh`** / **`sshd`** | Members can SSH in |
| **`audio`**, **`video`** | Hardware access groups |

### 🛠️ Managing Groups

```bash
# Create a new group
$ sudo groupadd developers

# Add a user to a group
$ sudo usermod -aG developers manish

# List groups a user belongs to
$ groups manish
manish : manish sudo docker developers

# Remove a user from a group
$ sudo gpasswd -d manish developers
```

---

## 5️⃣ The Root User — Total Power

The **root user** (UID 0) has **inherent privileges to perform any operation on the system** — including creating, modifying, or deleting files in protected directories (like `/`).

```
   ┌──────────────────────────────────────────────────────────────┐
   │                   THE PRIVILEGE HIERARCHY                    │
   │                                                              │
   │   root (UID 0)                                               │
   │   ├── can read/write/delete ANY file                         │
   │   ├── can kill ANY process                                   │
   │   ├── can change ANY file's ownership                        │
   │   ├── can modify system config (/etc, /boot, /usr)           │
   │   └── "the system has no guardrails for root"                │
   │                                                              │
   │   regular user (manish, UID 1000)                            │
   │   ├── can only access files they own / are permitted         │
   │   ├── can only kill their own processes                      │
   │   ├── can modify their home directory                        │
   │   └── guarded by permission system                           │
   │                                                              │
   │   service account (www-data, UID 33)                         │ 
   │   ├── restricted shell (/usr/sbin/nologin)                   │
   │   ├── exists only to own files / run a service               │
   │   └── cannot log in interactively                            │
   └──────────────────────────────────────────────────────────────┘
```

Only root can make changes to system-level directories and files that regular users cannot access.

> 🚨 **The classic root mistake:** Running `rm -rf /` as root actually deletes the entire system. Running it as a regular user only deletes what that user can write to (much less damage). This is why you should NOT log in as root — use `sudo` for specific commands instead.

---

## 6️⃣ `sudo` — Superuser Do

**`sudo`** = **S**uperuser **do**. It executes a single command with **root privileges** without requiring you to log in as root.

```bash
# Run one command as root
$ sudo apt update
[sudo] password for manish:
...

# Switch to a full root shell
$ sudo su
#

# Or equivalently:
$ sudo -i
#
```

### 🆚 `sudo` vs `su` vs `sudo su`

| Command | What it does |
|---|---|
| **`sudo <command>`** | Run one command as root (prompts for YOUR password) |
| **`su`** | Switch to root user (prompts for ROOT password) |
| **`sudo su`** | Switch to root (prompts for YOUR password — easier) |
| **`sudo -i`** | Start a login shell as root (cleanest) |
| **`sudo -u alice <cmd>`** | Run a command as user alice |

### 📋 Who Can Use `sudo`?

Only users in the `sudo` or `wheel` group (depending on distro):

```bash
# Add a user to the sudo group
$ sudo usermod -aG sudo lulu

# On RHEL/Fedora/CentOS, the group is called 'wheel'
$ sudo usermod -aG wheel lulu
```

The sudo configuration is in `/etc/sudoers` — but **never edit it directly**. Always use:

```bash
$ sudo visudo    # validates syntax before saving — prevents locking yourself out
```

---

## 7️⃣ The Principle of Least Power

This is the **single most important security principle in DevOps**:

> **Give each user account only the minimum amount of permissions necessary to perform its required tasks.**

```
   ┌──────────────────────────────────────────────────────────────┐
   │           THE PRINCIPLE OF LEAST POWER                       │
   │                                                              │
   │   ❌ BAD (over-privileged):                                  │
   │   - Running everything as root                               │
   │   - Giving users the 'sudo' group when they don't need it    │
   │   - chmod 777 on everything                                  │
   │   - Service accounts with /bin/bash shells                   │
   │                                                              │
   │   ✅ GOOD (least power):                                     │
   │   - Run services as dedicated users (www-data, postgres)     │
   │   - Give users access only to what they need                 │
   │   - chmod 644/755 instead of 777                             │
   │   - Service accounts with /usr/sbin/nologin                  │
   │   - Use sudo for specific commands, not a root shell         │
   │                                                              │
   │   WHY? Limits potential damage from:                         │
   │   - Security breaches (attacker gets limited access)         │ 
   │   - Accidental mistakes (rm -rf only destroys what user owns)│
   └──────────────────────────────────────────────────────────────┘
```

### 🎯 Why This Matters in Production

| Scenario | Over-privileged risk | Least-power mitigation |
|---|---|---|
| **Web server** | Running nginx as `root` → if exploited, attacker gets full system | Run as `www-data` → attacker only gets web-user access |
| **Database** | Running Postgres as `root` → SQL injection = root shell | Run as `postgres` user → isolated |
| **CI/CD runner** | Running builds as `root` → compromised pipeline = compromised host | Run as `gitlab-runner` user → limited blast radius |
| **Container** | Running as root inside container → container escape = host root | Use `USER appuser` in Dockerfile → safer |

> 🧭 **DevOps rule:** Every service should run as a dedicated, least-privilege user. Every team member should use `sudo` for specific commands, not a persistent root shell. Every file should have the minimum permissions that still allow the job to work. This isn't paranoia — it's standard practice.

---

## 8️⃣ Quick Reference Cheat Sheet

| Command | What it does |
|---|---|
| **`whoami`** | Show current username |
| **`id`** | Show UID, GID, and all groups |
| **`groups <user>`** | List groups a user belongs to |
| **`cat /etc/passwd`** | List all system users |
| **`sudo useradd -m -s /bin/bash <user>`** | Create a user with home dir + bash |
| **`sudo passwd <user>`** | Set a user's password |
| **`sudo userdel -r <user>`** | Delete a user + their home dir |
| **`sudo usermod -aG <group> <user>`** | Add user to a group (append!) |
| **`sudo groupadd <group>`** | Create a new group |
| **`sudo <command>`** | Run one command as root |
| **`sudo su` / `sudo -i`** | Switch to a root shell |
| **`sudo visudo`** | Edit sudoers file safely |
| **`su <user>`** | Switch to another user (needs their password) |

### 🧠 Memory Hooks

- **`useradd -m -s /bin/bash`** = make home (`-m`), set shell (`-s`)
- **`usermod -aG`** = **a**ppend to **G**roup (never use `-G` alone!)
- **`userdel -r`** = **r**emove home dir too
- **`sudo`** = **s**uperuser **do** (one command)
- **`su`** = **s**witch **u**ser

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **Multi-user Linux** | Every file belongs to a user and group. Kernel enforces boundaries between users. |
| **`whoami`** | Displays the currently logged-in user. |
| **`id`** | Shows UID, GID, and all group memberships. |
| **`/etc/passwd`** | System user database — 7 fields per line (username, x, UID, GID, GECOS, home, shell). |
| **`/bin/false` / `/usr/sbin/nologin`** | Restricted shells — user can't log in interactively. Used for service accounts. |
| **`useradd -m -s /bin/bash`** | Create user with home dir and bash shell. |
| **`passwd <user>`** | Set a user's password. |
| **`userdel -r`** | Delete user AND their home directory (`-r` = remove). |
| **`usermod -aG <group> <user>`** | Add user to a group. Always use `-aG` (append), never `-G` alone (replaces). |
| **`groups <user>`** | List groups a user belongs to. |
| **root (UID 0)** | Can do ANYTHING on the system — no guardrails. |
| **`sudo <command>`** | Run one command as root. Prompts for YOUR password. |
| **`sudo su` / `sudo -i`** | Switch to a root shell. |
| **`sudo visudo`** | Edit `/etc/sudoers` safely — always use this, never edit directly. |
| **`wheel` / `sudo` group** | Members can use `sudo`. (wheel on RHEL/Fedora, sudo on Debian/Ubuntu) |
| **Principle of Least Power** | Give each user only the minimum permissions necessary. Limits breach and mistake damage. |
| **Service accounts** | Run services as dedicated users (www-data, postgres) with restricted shells. Never as root. |
| **DevOps relevance** | Multi-user isolation + least power = the security foundation of every Linux server. |

---

← Back to [Permissions & Access Control](./README.md) | Next: [Permissions & Ownership](./2-permissions-ownership.md) →
