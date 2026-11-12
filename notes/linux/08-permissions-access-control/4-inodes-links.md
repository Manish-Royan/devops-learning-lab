# 🔗 Inodes & File Links (Hard vs Symbolic)

> **Goal:** Understand **inodes** (the data structure behind every file), **hard links** (multiple names pointing to the same inode), and **symbolic links** (shortcuts pointing to a path) — and know when to use each.

---

## 📍 What's Actually Behind a Filename?

Here's a fact that surprises most beginners: **the filename is NOT the file.**

When you create `notes.txt`, Linux doesn't store "notes.txt = some data." Instead:

1. The file's **data** is written to disk blocks
2. An **inode** (index node) is created — a metadata structure describing the file
3. The **filename** is just a pointer (a directory entry) that maps the name → inode number

```
   ┌────────────────────────────────────────────────────────────────┐
   │  THE THREE LAYERS OF A FILE:                                   │
   │                                                                │
   │  DIRECTORY ENTRY          INODE                DATA BLOCKS     │
   │  ┌────────────┐          ┌──────────┐         ┌──────────┐     │
   │  │ notes.txt  │────────▶│ inode #42│───────▶│ block 1  │     │
   │  │ → inode 42 │          │ metadata │         │ block 7  │     │
   │  └────────────┘          │          │         │ block 15 │     │
   │                          └──────────┘         └──────────┘     │
   │                                                                │
   │  The filename is just a label.                                 │
   │  The inode is the real file (metadata + pointer to data).      │
   │  The data blocks hold the actual content.                      │
   └────────────────────────────────────────────────────────────────┘
```

This separation between **filename** and **inode** is what makes hard links possible.

---

## 1️⃣ What Is an Inode?

An **inode** (index node) is a data structure storing **file metadata** — everything about the file *except the filename itself*.

### 📊 What an Inode Stores

| Stored in inode | NOT stored in inode |
|---|---|
| **Owner** (UID) | Filename |
| **Group** (GID) | |
| **Permissions** (r, w, x) | |
| **File size** | |
| **Timestamps** (created, modified, accessed) | |
| **Data block pointers** (where the actual data lives on disk) | |
| **Link count** (how many filenames point to this inode) | |

### 🔍 Inspecting Inodes

```bash
# Show inode numbers in a listing
$ ls -li
12345 -rw-r--r-- 1 paul paul 4096 Nov 5 notes.txt
^^^^^
inode number

# Get detailed inode info
$ stat notes.txt
  File: notes.txt
  Size: 4096            Blocks: 8          IO Block: 4096   regular file
Device: 801h/2049d      Inode: 12345       Links: 1
Access: (0644/-rw-r--r--)  Uid: (1000/paul)   Gid: (1000/paul)
Access: 2026-11-05 10:30:00
Modify: 2026-11-05 10:30:00
Change: 2026-11-05 10:30:00
```

The `ls -li` command shows the inode number as the first column. The `stat` command shows everything the inode stores.

### 📊 The `ls -l` Link Count

Remember that number after the permission string in `ls -l`?

```
   -rw-r--r-- 1 paul paul 4096 Nov 5 notes.txt
              ^
              this is the LINK COUNT
```

That number is the **hard link count** — how many filenames (directory entries) point to this same inode. A newly created file has a link count of `1`. If you create a hard link, it becomes `2`. And so on.

---

## 2️⃣ Hard Links — Same Inode, Multiple Names

A **hard link** is a directory entry that points to the **same inode** as the original file. Both names are equally "the file" — there's no original and copy.

### 📝 Creating a Hard Link

```bash
$ ln original.txt hardlink.txt
```

### 📊 What Happens Under the Hood

```
   BEFORE hard link:
   ┌──────────────┐
   │ original.txt │ ──┐
   └──────────────┘   │
                      ▼
                 ┌──────────┐
                 │ inode #42│  link count: 1
                 │ metadata │
                 │ + data   │
                 │ pointers │
                 └────┬─────┘
                      │
                      ▼
                 ┌──────────┐
                 │ data     │
                 │ blocks   │
                 └──────────┘

   AFTER ln original.txt hardlink.txt:
   ┌──────────────┐
   │ original.txt │ ────────┐
   └──────────────┘         │
                            ▼
                         ┌──────────┐
   ┌──────────────┐      │ inode #42│  link count: 2  ← now 2!
   │ hardlink.txt │ ──▶ │ metadata │
   └──────────────┘      │ + data   │
                         │ pointers │
                         └────┬─────┘
                              │
                              ▼
                        ┌──────────┐
                        │ data     │
                        │ blocks   │
                        └──────────┘
```

Both `original.txt` and `hardlink.txt` point to **the same inode** (same metadata, same data blocks). They're literally the same file with two names.

### ✅ Key Properties of Hard Links

1. **Share the exact same inode number** — verify with `ls -li`
2. **True synchronization** — editing either one updates both (they're the same data)
3. **Deleting the original doesn't destroy the data** — the inode (and data) persists until the link count hits zero
4. **Link count increments** — visible in `ls -l` output
5. **Cannot cross file systems** — hard links must be on the same filesystem (same partition)
6. **Cannot link to directories** — only to files (to prevent filesystem loops)

### 📋 The Deletion Test

```bash
$ echo "Hello World" > original.txt
$ ln original.txt hardlink.txt
$ ls -li
12345 -rw-r--r-- 2 paul paul 12 Nov 5 original.txt    ← link count: 2
12345 -rw-r--r-- 2 paul paul 12 Nov 5 hardlink.txt    ← SAME inode (12345)

$ rm original.txt          # delete the "original"
$ ls -li
12345 -rw-r--r-- 1 paul paul 12 Nov 5 hardlink.txt    ← link count: 1, data STILL HERE

$ cat hardlink.txt
Hello World                # ← data is intact!
```

**The data only disappears when the LAST link is deleted** (link count reaches zero). This is the key difference from symbolic links.

> 🧭 **DevOps use case:** Hard links are how backup tools like `rsync --link-dest` and Time Machine create space-efficient snapshots — multiple "copies" that share the same data blocks, with only changed files taking extra space.

---

## 3️⃣ Symbolic (Soft) Links — A Shortcut to a Path

A **symbolic link** (symlink) is a **separate file** with its **own inode** whose data contains the **target path string**. It's a shortcut — a pointer to a filename, not to the inode.

### 📝 Creating a Symbolic Link

```bash
$ ln -s original.txt symlink.txt
```

### 📊 What Happens Under the Hood

```
   ┌──────────────┐         ┌──────────────┐
   │ original.txt │         │ symlink.txt  │
   │              │         │ (type: l)    │
   │ inode #42    │         │ inode #99    │
   │ data: "Hello"│         │ data:        │
   └──────────────┘         │"original.txt"│ ← just the PATH STRING
                            └──────┬───────┘
                                   │
                                   │ follows the path
                                   ▼
                            ┌──────────────┐
                            │ original.txt │
                            │ inode #42    │
                            │ data: "Hello"│
                            └──────────────┘
```

The symlink has its **own inode** (inode #99). Its data is just the text string `"original.txt"` — the path to follow. When you access `symlink.txt`, the kernel reads its data, sees it's a symlink, and follows the path to `original.txt`.

### ✅ Key Properties of Symbolic Links

1. **Own inode** — separate file (type `l` in `ls -l`)
2. **Points to a PATH, not an inode** — if the target is renamed or moved, the symlink breaks
3. **Can cross file systems** — because it's just a path string
4. **Can link to directories** — unlike hard links
5. **If the original is deleted, the symlink becomes a broken link** (dangling symlink)

### 📋 The Deletion Test (Contrast with Hard Links)

```bash
$ echo "Hello World" > original.txt
$ ln -s original.txt symlink.txt
$ ls -li
12345 -rw-r--r-- 1 paul paul 12 Nov 5 original.txt    ← inode 12345
67890 lrwxrwxrwx 1 paul paul 11 Nov 5 symlink.txt -> original.txt  ← inode 67890, type l

$ rm original.txt          # delete the target
$ ls -li
67890 lrwxrwxrwx 1 paul paul 11 Nov 5 symlink.txt -> original.txt  ← still here, but...

$ cat symlink.txt
cat: symlink.txt: No such file or directory    ← BROKEN LINK!

$ ls -l symlink.txt
lrwxrwxrwx 1 paul paul 11 Nov 5 symlink.txt -> original.txt (broken)
```

When the target is deleted, the symlink still exists but **points to nothing** — it's a "broken" or "dangling" symlink. This is the key difference from hard links.

---

## 4️⃣ Hard Links vs Symbolic Links — Side by Side

| Property | Hard Link | Symbolic Link (Symlink) |
|---|---|---|
| **Command** | `ln original.txt hardlink.txt` | `ln -s original.txt symlink.txt` |
| **Inode** | **Shares** the same inode as original | Has its **own** inode |
| **What it points to** | The inode (the actual file data) | A path string (the filename) |
| **File type in `ls -l`** | Looks like a regular file (`-`) | Shows as link (`l`) with `->` |
| **If original is deleted** | **Data persists** — link count decreases, data survives until count hits 0 | **Breaks** — symlink becomes a dangling pointer |
| **If original is renamed** | Hard link still works (points to inode) | Symlink **breaks** (points to old name) |
| **If original is moved** | Hard link still works | Symlink **breaks** (path no longer valid) |
| **Crosses file systems?** | ❌ No — must be same filesystem | ✅ Yes — it's just a path string |
| **Can link to directories?** | ❌ No (prevents filesystem loops) | ✅ Yes |
| **Link count** | Increments the original's link count | Doesn't affect the original's link count |
| **Storage** | Just a directory entry (tiny) | A separate file containing the path string |

```
   ┌──────────────────────────────────────────────────────────────┐
   │  HARD LINK                    SYMBOLIC LINK                  │
   │                                                              │
   │  original.txt ──┐             original.txt                   │
   │                 ├──▶ inode   symlink.txt ──▶ "original.txt"│
   │  hardlink.txt ──┘             (own inode, type l)            │
   │                                                              │
   │  Both point DIRECTLY          Points to the PATH             │
   │  to the same inode            If target deleted: BREAKS      │
   │  If target deleted:           Can cross filesystems          │
   │  DATA SURVIVES                Can link to directories        │
   │  Same filesystem only         Tiny separate file             │
   └──────────────────────────────────────────────────────────────┘
```

---

## 5️⃣ When to Use Which?

### 🎯 Use Hard Links When:

- You want **backup-like redundancy** — deleting one name doesn't destroy the data
- You want **space efficiency** — no extra data stored, just another directory entry
- You're on the **same filesystem** and don't need to cross partitions
- You're working with **files** (not directories)

```bash
# Space-efficient backup snapshots
ln /data/important.db /backup/important.db
# Both names point to same data; deleting one doesn't lose the data
```

### 🎯 Use Symbolic Links When:

- You need to link **across file systems** (different partitions/disks)
- You need to link to **directories**
- You want a **shortcut** that's transparent about what it points to
- You're pointing to a **versioned path** (e.g., `/usr/bin/python -> python3.11`)
- You're managing **configuration flexibility** (e.g., `current-release -> v2.3.1`)

```bash
# Point to a specific version
ln -s /opt/node-v20.0.0 /opt/node-current
# Now /opt/node-current/bin/node works, and you can switch versions
# by re-pointing the symlink

# Link to a directory
ln -s /var/log/nginx /home/paul/nginx-logs
# Easy access to logs from your home directory
```

> 🧭 **DevOps use case:** Symlinks are how `nvm`, `pyenv`, `rbenv` and version managers work — `current` → `v20.0.0`, and switching versions just re-points the symlink. They're also how `/etc/init.d` scripts often work, and how you make a binary available system-wide: `ln -s /opt/myapp/bin/myapp /usr/local/bin/myapp`.

---

## 6️⃣ Practical Examples

### 📋 System Symlinks You'll See

```bash
# /usr/bin/python is often a symlink to a specific version
$ ls -l /usr/bin/python
lrwxrwxrwx 1 root root 9 Oct 5 /usr/bin/python -> python3.11

# /etc/rc.local might be a symlink
$ ls -l /etc/rc.local
lrwxrwxrwx 1 root root 21 Oct 5 /etc/rc.local -> /etc/init.d/rc.local

# A user's .bashrc might source a shared config
$ ls -l ~/.bashrc
lrwxrwxrwx 1 paul paul 25 Oct 5 .bashrc -> /shared/configs/bashrc
```

### 📋 Finding Broken Symlinks

```bash
# Find all broken symlinks in a directory
$ find /var/www -type l ! -exec test -e {} \; -print

# Or:
$ find /var/www -xtype l
```

### 📋 Removing Symlinks (Safely)

```bash
# Remove a symlink (does NOT affect the target)
$ rm symlink.txt
# OR
$ unlink symlink.txt

# ⚠️ DO NOT use rm -f on a symlink to a directory
# rm -rf symlink_to_dir/  ← the trailing slash makes rm follow the link
#                           and delete the TARGET directory's contents!
# Use: rm symlink_to_dir   (no trailing slash)
```

> ⚠️ **The trailing slash trap:** `rm -rf mylink/` (with slash) follows the symlink and deletes the **target's** contents. `rm -rf mylink` (no slash) removes the symlink itself. Always omit the trailing slash when removing symlinks to directories.

---

## 7️⃣ Quick Reference Cheat Sheet

| Command | What it does |
|---|---|
| **`ln original.txt hardlink.txt`** | Create a hard link (same inode) |
| **`ln -s original.txt symlink.txt`** | Create a symbolic link (path shortcut) |
| **`ls -li`** | List files with inode numbers |
| **`stat file.txt`** | Show detailed inode info |
| **`unlink symlink.txt`** | Remove a symlink (safe) |
| **`find /path -type l`** | Find all symlinks |
| **`find /path -xtype l`** | Find broken symlinks |

### 🧠 Memory Hooks

- **`ln`** = link (hard, no flag)
- **`ln -s`** = link **s**ymbolic (soft, with `-s` flag)
- **Hard link** = "hard" because it's solidly tied to the inode — survives target deletion
- **Symlink** = "soft" because it's fragile — breaks if target moves or is deleted
- **Inode** = **i**ndex **node** (metadata + data pointers, no filename)

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **Inode** | Index node — stores file metadata (owner, permissions, data pointers) but NOT the filename. |
| **`ls -li`** | Show files with their inode numbers (first column). |
| **`stat file`** | Detailed inode info — size, timestamps, link count, all metadata. |
| **Link count** | The number after permissions in `ls -l` — how many filenames point to this inode. |
| **Hard link** | `ln original.txt hardlink.txt` — shares the same inode. Both names are equally "the file." |
| **Hard link properties** | Shares inode, true sync, data survives until link count hits 0, same filesystem only, files only. |
| **Symbolic link** | `ln -s original.txt symlink.txt` — own inode, data is just the target path string. |
| **Symlink properties** | Own inode (type `l`), points to PATH not inode, breaks if target deleted/moved, can cross filesystems, can link to dirs. |
| **If target deleted** | Hard link: data survives. Symlink: becomes broken (dangling). |
| **If target renamed/moved** | Hard link: still works. Symlink: breaks. |
| **Cross filesystems** | Hard link: ❌ no. Symlink: ✅ yes. |
| **Link to directories** | Hard link: ❌ no. Symlink: ✅ yes. |
| **`unlink symlink`** | Safe way to remove a symlink (doesn't affect target). |
| **Trailing slash trap** | `rm -rf mylink/` follows the symlink and deletes target contents. `rm mylink` removes the symlink. Always omit trailing slash. |
| **Use hard links for** | Space-efficient backups, redundancy (data survives deletion of one name). |
| **Use symlinks for** | Version managers (`current → v2.3.1`), cross-filesystem links, directory links, system-wide binary access. |
| **DevOps relevance** | Symlinks = how version managers, init scripts, and config management work. Hard links = how backup snapshots save space. |

---

← Back to [Permissions & Access Control](./README.md) | Previous: [ACLs](./3-acls.md) | Next: [Environments](./5-environments.md) →
