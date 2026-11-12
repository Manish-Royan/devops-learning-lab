# Learning Log — 12 November 2026 🔐

**Learning Focus:** Linux Permissions & Access Control

**Learning Phase:** Linux Fundamentals → System Administration & Security

---

## 📍 Today's Learning Experience

Today's Linux learning session focused on one of the most important aspects of Linux administration: understanding users, permissions, ownership, and access control.

I explored how Linux manages access to files and directories, how different users interact with system resources, and how administrators control privileges without compromising the security of the system.

A central question throughout today's learning was:

**When Linux says "Permission denied", what exactly is preventing access?**

Rather than treating permission errors as something to fix with `chmod`, I wanted to understand the underlying access-control mechanisms.

## 📚 What I Explored Today

### 1. Users, Groups & Superuser

Explored Linux's multi-user security model and how identities are managed.

- Understanding Linux users and groups.
- Identifying users with `whoami` and `id`.
- Exploring `/etc/passwd` and user account information.
- Understanding UID and GID.
- Creating and managing users with `useradd`, `passwd`, and `userdel`.
- Managing group membership using `groupadd`, `groups`, and `usermod`.
- Understanding the root user and its privileges.
- Exploring `sudo`, `su`, and `sudo -i`.
- Understanding the Principle of Least Privilege.

### 2. File Permissions & Ownership

Explored how Linux controls access to files and directories.

- Understanding the 10-character permission and file-type string.
- Decoding owner, group, and others permissions.
- Understanding the different meanings of read, write, and execute for files vs directories.
- Modifying permissions using `chmod`.
- Exploring symbolic and octal permission modes.
- Understanding common permission values: `400`, `600`, `644`, `700`, and `755`.
- Understanding why `chmod 777` is generally an unsafe troubleshooting practice.
- Changing ownership using `chown`.
- Understanding the difference between `chmod`, `chown`, and `sudo`.

### 3. Access Control Lists (ACLs)

Explored how Linux provides more granular access control beyond the traditional owner/group/others model.

- Understanding ACLs and their purpose.
- Managing ACL entries using `setfacl`.
- Inspecting permissions using `getfacl`.
- Granting access to specific users and groups.
- Understanding recursive ACL operations.
- Exploring default ACLs for directory inheritance.
- Understanding the difference between standard permissions and extended ACLs.
- Exploring ACL management in onboarding and offboarding scenarios.

### 4. Inodes & File Links

Explored what happens underneath a filename and how Linux represents files.

- Understanding inodes and their role in filesystem metadata.
- Exploring directory entries and inode numbers.
- Inspecting inode information using `ls -li` and `stat`.
- Understanding hard links and how multiple filenames can reference the same inode.
- Exploring symbolic (soft) links and how they reference target paths.
- Understanding the differences between hard links and symbolic links.
- Exploring dangling symbolic links.
- Learning practical uses of symbolic links in version management and system configuration.

### 5. Environment Variables & Shell Configuration

Explored how processes receive environment information and how Bash configuration is managed.

- Understanding environment variables such as `PATH`, `HOME`, `USER`, and `SHELL`.
- Inspecting variables using `printenv`.
- Setting variables using `export`.
- Understanding temporary environment assignments.
- Exploring `/etc/environment`.
- Understanding `.bashrc` and `.bash_profile`.
- Exploring login and non-login shell behavior.
- Understanding environment-variable inheritance between parent and child processes.
- Exploring how environment configuration relates to development tools and automation.

---

## 💡 My Biggest Takeaway

One concept that particularly stood out to me was understanding that Linux access control is not simply about changing file permissions.

A file's access depends on several factors, including the identity of the process accessing it, ownership, group membership, permission bits, and potentially additional ACL rules.

I also explored how ACLs provide more granular access control when the traditional owner/group/others model isn't sufficient.

**Understanding why access is denied is much more important than blindly changing permissions until something works.**

---

## 🎯 Why This Matters in My DevOps Journey

Permissions and access control are fundamental to operating secure Linux infrastructure.

As I move further into DevOps, I will need to understand:

- How deployment users and service accounts should be configured.
- How applications obtain access to files and directories.
- How to troubleshoot ownership and permission errors.
- How to grant access without unnecessarily exposing resources.
- How to manage credentials and configuration securely.
- How to apply the Principle of Least Privilege.

These concepts will become especially important when working with application deployment, SSH, Docker, CI/CD runners, and cloud infrastructure.

---

## 🔍 What Comes Next?

My next step is to reinforce these concepts through practical Linux exercises.

I want to experiment with user and group creation, file permissions, ownership, ACLs, symbolic links, and environment variables.

The objective is to move beyond understanding the commands and become comfortable diagnosing real access-control problems.

---

## 📖 Detailed Technical Notes

Today's exploration is documented in the following technical chapter:

➡️ [Linux Permissions & Access Control](../../notes/linux/08-permissions-access-control/README.md)

The chapter includes detailed explanations, command examples, permission tables, diagrams, and practical DevOps scenarios.

---

**Learning in Public | Understanding security, access, and control — one concept at a time.** 🚀