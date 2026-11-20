# 🐧 DevOps Learning Journey — Day 10 & 11

**Dates:** November 17–18, 2026  
**Topic:** Linux SSH — Deep Dive into Secure Shell

## A note about these two learning days

These two days were different from my usual learning routine. I was unwell and couldn't prepare my regular notes or publish my daily progress.

However, I continued studying SSH in depth. Rather than rushing through the concepts just to maintain a publishing schedule, I spent time understanding how SSH works internally, how it establishes secure connections, and how it is used in real infrastructure.

I'm documenting these two days together because the topics form one connected learning chapter.

## 1. SSH Fundamentals

- What SSH (Secure Shell) is and why it is used.
- Telnet vs SSH.
- SSH client and server architecture.
- OpenSSH and the `sshd` daemon.
- Default SSH port: TCP 22.
- SSH-1 vs SSH-2.
- Practical uses of SSH in Linux administration, cloud infrastructure, Git, and CI/CD.

## 2. SSH Setup and First Connection

- Installing OpenSSH client and server packages.
- Managing the SSH daemon with `systemctl`.
- Starting and enabling `sshd`.
- Connecting to a remote machine.
- Understanding the first-connection host fingerprint prompt.
- Practising SSH connections using Fedora.

## 3. Cryptography Behind SSH

I explored the cryptographic concepts that make SSH secure:

- Symmetric encryption.
- Asymmetric cryptography.
- Hybrid encryption.
- Key exchange and Diffie–Hellman.
- Hashing and message integrity.
- Digital signatures.
- Temporary key exchange and session-key derivation.

## 4. Understanding the SSH Connection Handshake

I studied the major stages involved in establishing an SSH connection:

- TCP connection establishment.
- SSH protocol identification.
- Algorithm negotiation.
- Key exchange.
- Server host-key verification.
- User authentication.
- Encrypted communication through SSH channels.

I also explored why `known_hosts` matters and how it helps detect unexpected changes to a server's identity.

## 5. SSH Key-Based Authentication

- Public and private key pairs.
- Generating keys with `ssh-keygen`.
- Understanding `authorized_keys`.
- Copying public keys using `ssh-copy-id`.
- Private-key permissions.
- Passphrases and SSH agents.
- Agent forwarding.
- SSH authentication in cloud environments and GitHub.

## 6. SSH Configuration and Security Hardening

- Client configuration: `~/.ssh/config`.
- Server configuration: `/etc/ssh/sshd_config`.
- Password authentication settings.
- Root-login restrictions.
- Restricting permitted users.
- Firewall rules and SSH access control.
- Fail2Ban.
- Safely modifying SSH server configuration.

One important operational lesson: always back up the configuration, validate it with `sshd -t`, and test a new connection while keeping an existing session open.

## 7. File Transfer and SSH Tunneling

- SCP.
- SFTP.
- Local port forwarding.
- Accessing private services through an SSH tunnel.
- Understanding the role of bastion hosts.

## 8. Practical Lab — Fedora as an SSH Server

I practised SSH locally by using my Fedora machine as both the SSH client and server.

The lab involved:

- Installing and starting `sshd`.
- Generating SSH keys.
- Authorizing public keys.
- Configuring file permissions.
- Managing `known_hosts`.
- Testing key-based login.
- Exploring SSH server hardening.

## 9. Connecting SSH Theory with Go

I also studied how SSH concepts translate into application code. This led to a separate hands-on project:

**Go SSH Client:**  
https://github.com/Manish-Bishwakarma/go-ssh-client

The project uses Go's `golang.org/x/crypto/ssh` package to:

- Establish an SSH connection.
- Authenticate using private keys.
- Verify the server's identity using `known_hosts`.
- Open an SSH session.
- Execute a remote command and display its output.

This exercise helped connect Linux administration, networking, cryptography, and Go programming.

## My Key Takeaways

SSH is much more than a command such as `ssh user@server`.

Understanding its connection handshake, cryptographic foundations, host-key verification, authentication mechanisms, and security configuration gives me a much better understanding of how engineers securely manage remote infrastructure.

The biggest lesson from these two days was that understanding the mechanism behind a tool is just as important as knowing how to use it.

**Learning status:** This is an ongoing learning chapter. I am documenting the concepts I studied and the practical experiments I performed, while identifying areas that need further exploration.

---

*Learning in public. Building understanding one concept at a time.*