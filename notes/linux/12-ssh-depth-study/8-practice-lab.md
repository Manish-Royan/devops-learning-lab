# 8. Practice lab on Fedora

A safe way to practise everything in this chapter without a cloud server: the local Fedora machine acts as **both the SSH client and the SSH server**, and the connection is made to `localhost` (the machine connecting to itself). Key generation, key copying, login and configuration changes can all be tried this way.

---

## 8.1 Step 1: install and start the SSH server

Fedora comes with the SSH client. The server daemon (`sshd`) may not be installed or running.

```bash
sudo dnf install -y openssh-server        # install the server package
sudo systemctl enable --now sshd          # start now and start at every boot
sudo systemctl status sshd                # check the state
```

Expected result: green text saying `active (running)`.

## 8.2 Step 2: generate a key pair

```bash
ssh-keygen -t rsa -b 4096 -C "my-local-fedora-key"
```

Prompts and answers used in the lab:

| Prompt | Answer in the lab |
|---|---|
| File to save the key | Enter, to accept the default location in `~/.ssh/` |
| Passphrase | Enter twice for an empty passphrase, only to keep the practice quick |

> **Risk:** an empty passphrase leaves the private key usable by anyone who gets the file.
> **Safer way:** acceptable for a local throwaway lab only. On real keys, set a passphrase and use `ssh-add` so it is typed once per session (see [5. SSH key authentication](5-ssh-key-authentication.md)).

Check the files:

```bash
ls -la ~/.ssh/
```

Expected result: two new files, `id_rsa` (private key) and `id_rsa.pub` (public key). The comment `my-local-fedora-key` sits at the end of the line when the public key is displayed with `cat ~/.ssh/id_rsa.pub`.

## 8.3 Step 3: copy the public key and log in

```bash
ssh-copy-id localhost      # asks for the Linux account password one last time
ssh localhost              # logs in with no password prompt
exit                       # return to the normal terminal
```

On the first connection, SSH may show the host fingerprint and ask whether to trust it. Answering yes saves it in `known_hosts` (see [4. Connection handshake](4-connection-handshake.md)).

Expected result: a login to the same Fedora machine through an encrypted SSH loopback tunnel, with no password asked.

If `ssh localhost` fails or is refused, the first thing to check is the service: `systemctl status sshd` (see [2. Setup and first connection](2-setup-and-first-connection.md)).

## 8.4 Step 4: practise hardening the server config

Step 3 has to work before this step. The settings below turn off password login, so key login must already be proven.

1. Back up the configuration file first:

   ```bash
   sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
   ```

2. Open it:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

3. Find and change two settings:
   - `PasswordAuthentication`: uncomment it if needed and set it to `no`, which forces key authentication only.
   - `PermitRootLogin`: make sure it is `no`, which prevents direct logins as `root`.
4. Save and exit Nano with `Ctrl+O`, `Enter`, `Ctrl+X`.
5. Test the syntax:

   ```bash
   sudo sshd -t
   ```

   No output means the syntax is clean.

6. Restart the daemon to apply the change:

   ```bash
   sudo systemctl restart sshd
   ```

7. Open a second terminal and run `ssh localhost` again to confirm key login still works.

> **Risk:** a wrong setting, or turning off passwords before key login works, can lock the user out. On a remote server this can be unrecoverable.
> **Safer way:** keep the backup, run `sshd -t` before every restart, and keep the first terminal open until the second one logs in. To undo, copy the backup back over `sshd_config` and restart `sshd`.

## 8.5 How the lab maps to real work

| Lab action | Real-world situation | Why it matters |
|---|---|---|
| `sudo systemctl enable --now sshd` | Setting up a fresh cloud virtual machine, for example an EC2 instance | The SSH daemon must start automatically at boot |
| `ssh-copy-id localhost` | Automated provisioning | Putting public keys into `authorized_keys` so tools and scripts can connect securely |
| Editing `sshd_config` | Server hardening before a machine goes live | Closing off brute-force routes by disabling password and root logins |

## 8.6 What the lab proves

- A machine can be both an SSH client and an SSH server.
- Key login replaces the password after one `ssh-copy-id`.
- The server's behaviour is controlled in `/etc/ssh/sshd_config`, and a restart is needed to apply changes.
- A safe change always includes a backup, a syntax test and a second session.

---

Previous: [7. File transfer and tunneling](7-file-transfer-and-tunneling.md) | [Back to chapter overview](README.md)
