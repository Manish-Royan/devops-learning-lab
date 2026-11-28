# 5. Building the DNS Server with BIND

The DNS server software is **BIND**. On Fedora and Red Hat systems its service is called **`named`**. The goal is to make this machine the authoritative server for a private domain, `manishdevops.local`, so that `www.manishdevops.local` resolves to `192.168.110.86`.

```mermaid
graph LR
    A["/etc/named.conf (main configuration)"] -->|"zone block names the domain and the zone file"| B["/var/named/manishdevops.local.forwardzone (records)"]
    B --> C["named service answers on port 53"]
```

---

## Step 1: Install and start BIND

```bash
sudo dnf install -y bind bind-utils     # bind = the DNS server, bind-utils = dig and nslookup
rpm -qa | grep bind                     # list installed packages whose name contains "bind"
sudo systemctl start named
sudo systemctl status named
```

In this lab both packages were already installed (`bind-9.18.51...` and `bind-utils-9.18.51...`), so the install command was skipped.

Notes on the output:

- The `rpm -qa | grep bind` list also shows `rpcbind`. That is a different package whose name merely contains the word "bind". It is not part of BIND. For an exact check, `rpm -q bind bind-utils` queries just those two packages.
- The status line showed `named.service` as an `alias`, and enabling the service later created a link named `bind.service`. The service therefore has both names.
- The start-up log contained lines such as `network unreachable resolving './NS/IN'` with IPv6 addresses. These are normal when the network does not route IPv6 to the root DNS servers.

### Make it start at boot

```bash
sudo systemctl enable --now named       # enable at boot and make sure it is running now
```

---

## Step 2: Open the DNS door in the firewall

```bash
sudo firewall-cmd --add-service=dns --permanent
sudo firewall-cmd --reload
```

Firewall rules are per service or port. Opening `http` (TCP 80) only lets other devices reach the web page. DNS uses **UDP and TCP port 53**, so it needs its own rule. Without it, another device that uses `192.168.110.86` as its DNS server would time out or be refused.

The two servers work together like this:

1. The phone first asks door **53** (DNS) to translate `www.manishdevops.local` into an IP address.
2. The phone then knocks on door **80** (HTTP) to load the page from Apache.

If either door stays locked, the visit fails. Both rules use the same pattern: `--permanent` saves the rule so it survives a reboot, and `--reload` applies the saved rules right away. When only the machine itself is queried, the rule is not required.

---

## Step 3: Configure `/etc/named.conf`

This is the main BIND configuration file.

### Back it up first

```bash
sudo cp /etc/named.conf /etc/named.conf.bak
ls -l /etc/named.conf.bak
```

A backup file named `.bak` does no harm: BIND reads only `/etc/named.conf` and the files it explicitly includes. It is a restore point if an edit breaks the setup. To restore it:

```bash
sudo cp /etc/named.conf.bak /etc/named.conf
sudo systemctl restart named
sudo named-checkconf /etc/named.conf
```

### Reading and editing the file

The file is not readable by a normal user (the backup copy of it shows permissions `rw-r-----`), so a plain `cat /etc/named.conf` ends with `Permission denied`. Prefixing the command with `sudo` fixes that. In this lab the previous command was re-run with sudo using `sudo !!`.

Two attempts to edit the file with `vi` were suspended instead of closed (the terminal printed `[1]+ Stopped sudo vi /etc/named.conf`, then `[2]+ Stopped`), so the edit was done with `nano` instead (save with `Ctrl+O`, `Enter`, exit with `Ctrl+X`).

### The `options` block

As shipped, the file configures BIND as a caching resolver for the local machine only. Two lines matter for this lab:

```text
options {
    listen-on port 53 { 127.0.0.1; 192.168.110.86; };
    listen-on-v6 port 53 { ::1; };
    directory "/var/named";
    allow-query { localhost; };
    recursion yes;
    ...
};
```

| Line | Meaning |
|---|---|
| `listen-on port 53 { 127.0.0.1; 192.168.110.86; };` | The addresses on which BIND answers DNS questions. By default only `127.0.0.1` (the machine itself). Adding the Wi-Fi address `192.168.110.86` lets other devices on the network ask as well. |
| `directory "/var/named";` | The base folder. File names in the zone blocks are looked up here. |
| `allow-query { localhost; };` | Who may ask general questions. It was left as `localhost`. To let other devices use the server for everything, it would have to be widened, for example to `{ localhost; 192.168.110.0/24; }` (or `any` for a short lab test). |
| `recursion yes;` | Lets the server look names up on behalf of clients. The comments in the shipped file warn that a recursive server with a public IP **must** restrict who may query it, otherwise it can be abused in large-scale DNS amplification attacks. |

### The zone block

A zone block is added near the bottom of the file, above the `include` lines:

```text
zone "manishdevops.local" IN {
    type master;
    file "manishdevops.local.forwardzone";
    allow-query { any; };
};
```

| Line | Meaning |
|---|---|
| `zone "manishdevops.local" IN { ... };` | Declares that this server handles the domain `manishdevops.local`, in the `IN` (Internet) class. Without this block BIND does not know it owns the domain and cannot serve its records. |
| `type master;` | This server is the **primary, authoritative** source for the zone. It reads the zone file itself instead of copying it from another DNS server. |
| `file "manishdevops.local.forwardzone";` | The zone file that holds the name-to-IP mappings. The path is relative to `/var/named`. The name here must match the real file name exactly. |
| `allow-query { any; };` | Any client that reaches the server may ask about this zone. This keeps a phone or another device from being blocked for this zone's records. |

> **Caution:** `allow-query { any; }` is acceptable on a private home network for a lab. It should not be used on a server that is reachable from the internet.

### Check the syntax before using it

```bash
sudo named-checkconf /etc/named.conf
```

No output means the syntax is valid.

---

## Step 4: Create the zone file

The zone file goes in `/var/named/` and is named exactly as in the zone block.

```bash
sudo tee /var/named/manishdevops.local.forwardzone > /dev/null <<'EOF'
$TTL 86400
@       IN SOA  ns1.manishdevops.local. admin.manishdevops.local. (
                2026112601 ; Serial
                3600       ; Refresh
                1800       ; Retry
                604800     ; Expire
                86400 )    ; Minimum TTL
@       IN NS   ns1.manishdevops.local.
ns1     IN A    192.168.110.86
@       IN A    192.168.110.86
www     IN A    192.168.110.86
EOF
```

### Line by line

| Line | Meaning |
|---|---|
| `$TTL 86400` | Default TTL for the records: 86,400 seconds, which is one day. Caches may keep these records for that long. |
| `@ IN SOA ns1.manishdevops.local. admin.manishdevops.local. (...)` | The Start of Authority record. `@` stands for the domain itself (the apex). The first name is the primary name server. The second name is the administrator's email address, with the first dot standing for `@` (`admin@manishdevops.local`). |
| `2026112601 ; Serial` | The zone's version number. It has to be increased every time the file is edited. The value here follows a date-plus-counter pattern (2026-11-26, edit 01). |
| `3600 ; Refresh`, `1800 ; Retry`, `604800 ; Expire`, `86400 ; Minimum TTL` | The timing values of the SOA record, in seconds. They were used as written in this lab. |
| `@ IN NS ns1.manishdevops.local.` | Declares `ns1.manishdevops.local.` as the authoritative name server of the domain. |
| `ns1 IN A 192.168.110.86` | Gives the name server itself an IPv4 address. |
| `@ IN A 192.168.110.86` | The bare domain `manishdevops.local` resolves to the server. |
| `www IN A 192.168.110.86` | `www.manishdevops.local` resolves to the server. |

The names that end in a dot are written out in full, including the hidden root dot described in [1-dns-basics.md](1-dns-basics.md). The plain labels `ns1` and `www` are the host parts inside the zone.

An `A` record only maps a name to an IP address. It does not choose a port and it does not redirect anywhere. The port comes from the URL the browser opens (`http://` means port 80).

### Group ownership and SELinux label

```bash
sudo chgrp named /var/named/manishdevops.local.forwardzone     # group becomes "named", the BIND service's group, so BIND may read it
sudo restorecon -v /var/named/manishdevops.local.forwardzone   # resets the SELinux security label so named is allowed to open the file
```

Both commands act on this one file only. `restorecon -v` printed nothing here, so the label was already correct.

### Validate the zone file

```bash
sudo named-checkzone manishdevops.local /var/named/manishdevops.local.forwardzone
```

Result in the lab:

```text
zone manishdevops.local/IN: loaded serial 2026112601
OK
```

### Restart BIND so it loads the zone

```bash
sudo systemctl restart named
systemctl is-active named        # active
```

---

## Step 5: Test the DNS server directly

Three tests, each aimed at a different thing:

```bash
dig @192.168.110.86 www.manishdevops.local +short
# 192.168.110.86

nslookup www.manishdevops.local 192.168.110.86
# Server:   192.168.110.86
# Name:     www.manishdevops.local
# Address:  192.168.110.86

curl -s --resolve www.manishdevops.local:80:192.168.110.86 http://www.manishdevops.local | head -n 10
# prints the first 10 lines of index.html
```

- `dig @<server>` and `nslookup <name> <server>` send the question **straight to the BIND server**, so they test BIND alone and do not depend on any system setting. `+short` makes `dig` print only the answer.
- `curl --resolve name:80:IP` tells curl which IP belongs to the name, so curl does no DNS lookup at all. It proves that Apache answers correctly when the page is requested by that host name. It does not test BIND.

### What a plain lookup does before the system is pointed at BIND

```bash
nslookup www.manishdevops.local
# Server:   127.0.0.53
# ** server can't find www.manishdevops.local: REFUSED
```

Without a server name, `nslookup` uses `127.0.0.53`, Fedora's local stub resolver, and not the BIND server on `192.168.110.86`. That resolver does not know the private domain and answers `REFUSED`. The next step is to tell the system and the clients to use the BIND server, which is covered in [6-client-setup-and-testing.md](6-client-setup-and-testing.md).
