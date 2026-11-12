# 🎛️ Access Control Lists (ACLs)

> **Goal:** Understand ACLs — Linux's **fine-grained permission system** that goes beyond the standard owner/group/others model. Master `setfacl` (set) and `getfacl` (view) for scenarios where `chmod` isn't enough.

---

## 📍 When `chmod` Isn't Enough

Standard Linux permissions (`chmod`) give you **three buckets**: owner, group, and others. That's it. But what if you need to give **one specific user** access to a file without:
- Changing the file's ownership (`chown`)
- Changing the group membership
- Giving access to ALL "others"

```
   ┌──────────────────────────────────────────────────────────────┐
   │  THE LIMIT OF STANDARD PERMISSIONS (chmod):                  │
   │                                                              │
   │  File: config.yaml                                           │
   │  Owner: appuser  (rwx)                                       │
   │  Group: appgrp   (r-x)                                       │
   │  Others:          (r--)                                      │
   │                                                              │
   │  Q: "I need to give alice (who is NOT in appgrp) write       │
   │      access to config.yaml — but NOT give it to everyone."   │
   │                                                              │
   │  With chmod alone:                                           │
   │  ❌ Can't target a specific user                             │
   │  ❌ Adding alice to appgrp gives her ALL group files         │
   │  ❌ Giving others write = everyone can modify it             │
   │                                                              │
   │  With ACLs:                                                  │
   │  ✅ setfacl -m u:alice:rw- config.yaml                       │
   │  ✅ Alice gets read+write, nobody else is affected           │
   └──────────────────────────────────────────────────────────────┘
```

That's what **ACLs (Access Control Lists)** are for — they let you inject permissions for **specific users or groups** on top of the standard owner/group/others model.

---

## 1️⃣ What Are ACLs?

ACLs provide **fine-grained permissions for specific users or groups** beyond the standard owner-group-other model.

| Feature | Standard `chmod` | ACLs (`setfacl`) |
|---|---|---|
| **Permission targets** | 3 buckets: owner, group, others | Unlimited: any specific user or group |
| **Granularity** | One setting per bucket | Per-user, per-group rules |
| **Use case** | General file permissions | "Give alice access to this one file" |
| **Complexity** | Simple (rwx × 3) | More complex, but more precise |

```
   STANDARD PERMISSIONS (3 buckets):
   ┌──────────┐  ┌──────────┐  ┌──────────┐
   │  Owner   │  │  Group   │  │  Others  │
   │  rwx     │  │  r-x     │  │  r--     │
   └──────────┘  └──────────┘  └──────────┘
        │             │             │
        └─────────────┴─────────────┘
        Everyone falls into exactly one of these 3 buckets.

   WITH ACLs (unlimited specific users/groups):
   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
   │  Owner   │  │  Group   │  │  Others  │  │  alice   │  │   bob    │
   │  rwx     │  │  r-x     │  │  r--     │  │  rwx     │  │  rw-     │
   └──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘
                                                  │              │
                                          specific user    specific user
                                          (via setfacl)    (via setfacl)
```

---

## 2️⃣ `setfacl` — Setting ACLs

### 📝 The Command Formula

```bash
setfacl [options] [user/group]:[name]:[permissions] filename
```

### 📋 The Flags

| Flag | Meaning |
|---|---|
| **`-m`** | **M**odify — add or update an ACL entry |
| **`-x`** | Remove a **specific** ACL entry |
| **`-b`** | **Remove all** ACL entries (strip back to baseline) |
| **`-R`** | **R**ecursive — apply to a directory and everything inside |
| **`-k`** | Remove the default ACL from a directory |

### 📊 ACL Entry Syntax

```
   setfacl -m u:alice:rwx file.txt
               │  │     │
               │  │     └── permissions (r, w, x)
               │  └──────── username (or group name)
               └─────────── u for user, g for group
```

### 📋 Common Patterns

```bash
# Give a specific user read+write access
setfacl -m u:contractor:rw- project.txt

# Give a specific user full rwx access
setfacl -m u:alice:rwx /var/www/html/

# Give a GROUP read+execute access
setfacl -m g:qa-team:r-x /app/test-results/

# Recursive — apply to directory AND all contents
setfacl -R -m u:alice:rwx /var/www/html

# Remove a specific user's ACL entry
setfacl -x u:contractor project.txt

# Remove ALL ACL entries (strip back to baseline chmod)
setfacl -b project.txt
```

### 📝 Shorthand: `-Rm` (Combined Flags)

```bash
# These two are identical:
setfacl -R -m u:alice:rwx /var/www/html
setfacl -Rm u:alice:rwx /var/www/html
```

In Linux command syntax, short flags can be combined. Writing **`-Rm`** is a shortcut for **`-R -m`** (Recursive + Modify).

### 📝 Quoting Entries (Safety Habit)

```bash
# Without quotes (works for simple entries):
setfacl -m u:alice:rwx file.txt

# With quotes (safer for complex entries):
setfacl -m "u:alice:rwx" file.txt
```

Putting quotes around the ACL entry is a **safety habit** — it prevents the shell from misinterpreting special characters (like colons `:` or asterisks `*`) if your terminal parser gets confused. Optional for simple entries, recommended for scripts.

---

## 3️⃣ `getfacl` — Viewing ACLs

To **view** the ACL settings on a file or directory:

```bash
$ getfacl project.txt
# file: project.txt
# owner: appuser
# group: appgrp
user::rwx
group::r-x
other::r--
user:contractor:rw-
user:alice:rwx
group:qa-team:r-x
```

### 📊 How to Read `getfacl` Output

```
   # file: project.txt              ← filename
   # owner: appuser                 ← standard owner
   # group: appgrp                  ← standard group
   user::rwx                        ← owner's standard permissions
   group::r-x                       ← group's standard permissions
   other::r--                       ← others' standard permissions
   user:contractor:rw-              ← ACL: contractor gets rw-
   user:alice:rwx                   ← ACL: alice gets rwx
   group:qa-team:r-x                ← ACL: qa-team group gets r-x
```

The lines WITHOUT a name (`user::`, `group::`, `other::`) are the **standard permissions** — same as what `ls -l` shows. The lines WITH a name (`user:contractor:`, `user:alice:`, `group:qa-team:`) are the **ACL entries** — additional rules on top.

> 💡 When ACLs are present, `ls -l` shows a **`+`** at the end of the permission string:
> ```
> -rw-rw-r--+ 1 appuser appgrp 4096 Nov 5 project.txt
>            ↑
>            this + means "ACLs are present on this file"
> ```

---

## 4️⃣ Real DevOps Scenarios

### 🎯 Scenario 1: Onboarding a New Contributor

**Situation:** You manage a production web server hosting an app under `/var/www/html`. A new senior engineer named `alice` joins the team. She needs to deploy code, modify configs, and manage assets across the entire web directory tree immediately.

**The problem:** The directory is owned by `www-data:www-data`. You can't `chown` it to alice without breaking the web server's access. And adding alice to the `www-data` group would give her access to ALL www-data files system-wide.

**The ACL solution:**

```bash
$ sudo setfacl -R -m u:alice:rwx /var/www/html
```

**Translation:** Recursively (`-R`) modify (`-m`) the ACL to give user `alice` full `rwx` access across `/var/www/html` and every file and subfolder inside it.

**Result:** Alice gets full access to the web directory tree. The web server (`www-data`) keeps its ownership and access. No existing permissions are disrupted. Alice didn't need to be added to any group.

```
   BEFORE ACL:                          AFTER ACL:
   /var/www/html/                       /var/www/html/
   owner: www-data                      owner: www-data
   group: www-data                      group: www-data
   others: r-x                          others: r-x
                                        + ACL: alice = rwx

   alice: no access (just "others")    alice: FULL access (rwx)
   www-data: unaffected                 www-data: STILL unaffected ✅
```

> 🧭 **DevOps advantage:** ACLs are **non-destructive**. You inject access for one user without touching anyone else's permissions. This is why ACLs are the go-to for temporary access grants.

---

### 🎯 Scenario 2: Contractor Access (Targeted)

**Situation:** A freelance contractor (`bob`) needs read+write access to ONE specific config file for debugging — but not the whole directory.

```bash
$ sudo setfacl -m u:bob:rw- /etc/myapp/config.yaml
```

Bob gets read+write on `config.yaml` only. He can't touch other files in `/etc/myapp/`. He's not added to any group. When his contract ends, you remove his access:

```bash
$ sudo setfacl -x u:bob /etc/myapp/config.yaml
```

The `-x` flag removes that specific user's ACL entry — surgical removal.

---

### 🎯 Scenario 3: Offboarding & Security Cleanup

**Situation:** The contractor's engagement is over. Security compliance requires auditing the system to ensure no external users retain backdoor permissions. Multiple files may have ACL entries from various contractors over time.

```bash
$ sudo setfacl -b sensitive-config.conf
```

The **`-b`** flag stands for "remove all." It **deletes ALL extended ACL entries** on that file or directory, leaving only the **baseline standard permissions** (owner, group, others) dictated by traditional `chmod`.

```
    BEFORE setfacl -b:                      AFTER setfacl -b:
   ┌───────────────────────┐             ┌───────────────────────┐
   │ sensitive-config.conf │             │ sensitive-config.conf │
   │ owner: appuser  rwx   │             │ owner: appuser  rwx   │
   │ group: appgrp   r-x   │             │ group: appgrp   r-x   │
   │ others:          r--  │             │ others:          r--  │
   │ ACL: bob       rw-    │             │ (no ACL entries)      │
   │ ACL: temp_user rwx    │             │                       │
   └───────────────────────┘             └───────────────────────┘
   Multiple contractor rules             Stripped back to baseline
```

Instead of manually deleting individual user rules one by one, `setfacl -b` strips ALL custom ACL rules in a single stroke — returning the file strictly to its standard baseline state.

> 🧭 **DevOps use case:** Run `setfacl -b` during offboarding to guarantee no lingering access. Especially important for compliance audits (SOC 2, HIPAA) where you must prove no external users retain access.

---

## 5️⃣ Default ACLs (For New Files)

You can set a **default ACL** on a directory — any new file created inside it automatically inherits the ACL:

```bash
# Set a default ACL: alice gets rwx on any new file in this directory
$ sudo setfacl -d -m u:alice:rwx /var/www/html
```

| Flag | Meaning |
|---|---|
| **`-d`** | Set **default** ACL — applies to new files created inside the directory |

```bash
# Verify default ACL is set
$ getfacl /var/www/html
# file: var/www/html
# owner: www-data
# group: www-data
user::rwx
group::r-x
other::r-x
default:user:alice:rwx      ← new files inherit this
default:group::r-x
default:other::r-x

# Now any new file created here automatically grants alice rwx:
$ touch /var/www/html/newfile.txt
$ getfacl /var/www/html/newfile.txt
user::rw-
user:alice:rwx              ← inherited from default ACL
group::r-x
other::r--
```

> 💡 Default ACLs are how you ensure "every file created in this shared directory automatically grants access to the right people" without manually running `setfacl` on each new file.

---

## 6️⃣ ACLs vs Standard Permissions — When to Use Which

| Situation | Use |
|---|---|
| Standard file permissions (owner/group/others) | `chmod` |
| Need to give ONE specific user access without changing ownership or group | `setfacl` |
| Need to give ONE specific group access | `setfacl` |
| Temporary contractor access to specific files | `setfacl` |
| Need permissions to propagate to new files automatically | `setfacl -d` (default ACL) |
| Cleanup all custom access rules at once | `setfacl -b` |
| Web server needs to own the entire directory | `chown` (not ACL) |

```
   ┌──────────────────────────────────────────────────────────────┐
   │  DECISION TREE: chmod vs chown vs setfacl                    │
   │                                                              │
   │  Need to change what actions are allowed?                    │
   │    → chmod (r/w/x for owner/group/others)                    │
   │                                                              │
   │  Need to change who owns the file?                           │
   │    → chown (user:group)                                      │
   │                                                              │
   │  Need to give a SPECIFIC user/group access without           │
   │  changing ownership or group membership?                     │
   │    → setfacl (fine-grained, non-destructive)                 │
   │                                                              │
   │  Need to clean up all custom access rules?                   │
   │    → setfacl -b (strip all ACLs)                             │
   └──────────────────────────────────────────────────────────────┘
```

---

## 7️⃣ Quick Reference Cheat Sheet

| Command | What it does |
|---|---|
| **`setfacl -m u:alice:rwx file`** | Give alice rwx on file |
| **`setfacl -m g:qa-team:r-x file`** | Give qa-team group read+execute |
| **`setfacl -R -m u:alice:rwx dir/`** | Recursive — alice gets rwx on entire directory tree |
| **`setfacl -Rm u:alice:rwx dir/`** | Same as above (combined flags) |
| **`setfacl -d -m u:alice:rwx dir/`** | Default ACL — new files inherit alice's access |
| **`setfacl -x u:alice file`** | Remove alice's specific ACL entry |
| **`setfacl -b file`** | Remove ALL ACL entries (strip to baseline) |
| **`getfacl file`** | View ACL settings on a file |
| **`ls -l` with `+`** | The `+` after permissions means ACLs are present |

### 🧠 Memory Hooks

- **`setfacl -m`** = **m**odify (add/update an entry)
- **`setfacl -x`** = remove one specific entry (e**x**clude)
- **`setfacl -b`** = **b**lank out / remove ALL entries
- **`setfacl -R`** = **r**ecursive (directory tree)
- **`setfacl -d`** = **d**efault (new files inherit it)
- **`getfacl`** = **get** (view) the ACL

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **ACLs** | Fine-grained permissions for specific users/groups, beyond owner/group/others. |
| **`setfacl -m u:alice:rwx`** | Give alice rwx on a file — non-destructive, doesn't change ownership or group. |
| **`setfacl -m g:team:r-x`** | Give a specific group read+execute access. |
| **`setfacl -R -m`** | Recursive — apply to directory and everything inside. Shortcut: `-Rm`. |
| **`setfacl -d -m`** | Default ACL — new files created in the directory inherit the rule. |
| **`setfacl -x u:alice`** | Remove ONE specific user's ACL entry (surgical removal). |
| **`setfacl -b file`** | Remove ALL ACL entries — strip back to baseline `chmod` permissions. |
| **`getfacl file`** | View the ACL settings on a file. Shows standard perms + ACL entries. |
| **`ls -l` shows `+`** | The `+` after the permission string means ACLs are present on that file. |
| **ACL entry format** | `[u or g]:[name]:[rwx]` — e.g., `u:alice:rwx`, `g:qa:r-x`. |
| **Quotes `"u:alice:rwx"`** | Safety habit — prevents shell misinterpreting special chars. Optional for simple entries. |
| **Non-destructive** | ACLs inject access for specific users WITHOUT changing ownership, group, or standard permissions. |
| **Onboarding scenario** | `setfacl -R -m u:alice:rwx /var/www/html` — alice gets full access without disrupting web server. |
| **Offboarding scenario** | `setfacl -b sensitive-config.conf` — strip ALL custom ACL rules in one stroke. |
| **`chmod` vs `chown` vs `setfacl`** | `chmod` = what actions. `chown` = who owns. `setfacl` = specific user access without changing ownership. |
| **DevOps relevance** | ACLs = surgical access control. Perfect for contractors, onboarding, compliance cleanup — without disrupting existing permission structures. |

---

← Back to [Permissions & Access Control](./README.md) | Previous: [Permissions & Ownership](./2-permissions-ownership.md) | Next: [Inodes & Links](./4-inodes-links.md) →
