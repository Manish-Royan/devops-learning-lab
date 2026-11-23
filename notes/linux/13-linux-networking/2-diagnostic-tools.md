# 🔧 Diagnostic Tools — `ping`, `traceroute`, `curl`, `wget`, `jq`, `watch`

> **Goal:** Master the everyday network diagnostic toolkit — `ping` (reachability + latency), `traceroute`/`tracepath`/`mtr` (hop-by-hop path tracing), `ifconfig`/`arp`/`ifplugstatus` (interface tools), `curl`/`wget` (HTTP + downloads), `jq` (JSON processing), and `watch` (live monitoring).

---

## 📍 The DevOps Diagnostic Mindset

When something is "slow" or "not working," you need tools that answer specific questions:
- **"Is the server alive?"** → `ping`
- **"Where do packets get lost?"** → `traceroute` / `mtr`
- **"Is the network cable plugged in?"** → `ifplugstatus`
- **"Does the API respond?"** → `curl`
- **"Can I download that file?"** → `wget`
- **"Is the JSON response valid?"** → `jq`
- **"Is the situation changing over time?"** → `watch`

Many of these tools are **not pre-installed** on minimal server images. You may need to install them:

```bash
# Debian/Ubuntu
$ sudo apt install net-tools traceroute dnsutils mtr nmap jq whois

# RHEL/Fedora
$ sudo dnf install net-tools traceroute bind-utils mtr nmap jq whois
```

---

## 1️⃣ `ping` — Reachability & Latency

**`ping`** stands for **Packet Internet Groper**. It sends **ICMP** (Internet Control Message Protocol) echo request packets to a target and measures:
- **Latency** — how long replies take (milliseconds)
- **Packet loss** — percentage of packets that didn't return

### 📋 What `ping` Checks

| Check | What it tells you |
|---|---|
| **Network connectivity** | Can you reach the target at all? |
| **Internet connection** | Is your internet working? (`ping 8.8.8.8`) |
| **NIC health** | Is your network interface working? (`ping localhost`) |
| **Latency** | How fast is the connection? |
| **DNS resolution** | Does the domain resolve? (`ping google.com`) |

### 📝 Basic Usage

```bash
$ ping -c 4 example.com
PING example.com (93.184.216.34) 56(84) bytes of data.
64 bytes from 93.184.216.34: icmp_seq=1 ttl=56 time=14.2 ms
64 bytes from 93.184.216.34: icmp_seq=2 ttl=56 time=13.9 ms
64 bytes from 93.184.216.34: icmp_seq=3 ttl=56 time=14.5 ms
64 bytes from 93.184.216.34: icmp_seq=4 ttl=56 time=14.1 ms

--- example.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 13.920/14.175/14.510/0.225 ms
```

### 📊 Interpreting Packet Loss

| Loss | What it usually means |
|---|---|
| **0%** | Clean reachability ✅ |
| **Partial (e.g., 30%)** | Network congestion, bandwidth pressure, intermittent link drops |
| **100%** | Host is down, unreachable, blocked by a firewall, or local connectivity is broken |

### 📋 Useful Flags

| Flag | What it does |
|---|---|
| **`-c N`** | Send N packets then stop (otherwise runs forever) |
| **`-i N`** | Wait N seconds between packets (default 1s) |
| **`-W N`** | Timeout after N seconds for each reply |

> 🧭 **DevOps tip:** Always use `-c` in scripts — `ping` without it runs forever. `ping -c 4 google.com` sends 4 packets and exits.

---

## 2️⃣ `traceroute` / `tracepath` / `mtr` — Hop-by-Hop Path Tracing

Traffic rarely travels in a straight line. Routers forward packets **hop by hop** toward the destination. These tools show you the path.

### 📝 `traceroute`

Traces the hops packets take to reach a destination, showing per-hop timing:

```bash
$ traceroute example.com
traceroute to example.com (93.184.216.34), 30 hops max, 60 byte packets
 1  172.31.16.1   (172.31.16.1)   1.120 ms  1.045 ms  0.998 ms
 2  100.64.0.1    (100.64.0.1)    2.410 ms  2.380 ms  2.355 ms
 3  * * *
 4  93.184.216.34 (93.184.216.34) 14.180 ms 14.110 ms 14.095 ms
```

- Each line is a **hop** (a router along the way)
- Three time samples per hop (in ms)
- `* * *` means that hop didn't respond (common — many routers block traceroute)

### 📝 `tracepath`

Similar to `traceroute` but also traces **path MTU** (maximum packet size):

```bash
$ tracepath example.com
 1?: [LOCALHOST]           pmtu 1500
 1:  172.31.16.1           1.080ms
 2:  100.64.0.1            2.390ms
 3:  no reply
 4:  93.184.216.34         14.150ms reached
    Resume: pmtu 1500 hops 4 back 4
```

### 📝 `mtr` (My Traceroute) — The Best of Both

Combines **continuous `ping`** with hop-by-hop path visibility. Runs live and updates in real time:

```bash
# Interactive live mode
$ mtr example.com

# Report mode (for scripts/logs)
$ mtr -r -c 3 example.com
Start: 2026-11-23T14:09:00
HOST: app-server-01    Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- 172.31.16.1      0.0%     3    1.1   1.1   1.0   1.2   0.1
  2.|-- 100.64.0.1       0.0%     3    2.4   2.4   2.3   2.5   0.1
  3.|-- ???            100.0%     3    0.0   0.0   0.0   0.0   0.0
  4.|-- 93.184.216.34    0.0%     3   14.1  14.2  14.0  14.3   0.2
```

> 🧭 **DevOps tip:** `mtr` is the go-to tool for diagnosing "where is the latency coming from?" If hop 3 shows 100% loss but hop 4 responds, the issue is just that hop 3 blocks ICMP — traffic still flows through it. But if latency spikes at hop 5, that's where your bottleneck is.

---

## 3️⃣ Interface Tools — `ifconfig`, `arp`, `ifplugstatus`

### 📝 `ifconfig` (Legacy)

From the `net-tools` suite. Shows interface info — still around but superseded by `ip a`:

```bash
$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
      inet 172.31.18.42  netmask 255.255.240.0  broadcast 172.31.31.255
      ether 06:a2:15:8c:91:0a  txqueuelen 1000  (Ethernet)
      RX packets 14820  bytes 19450120 (19.4 MB)
      TX packets 9310  bytes 1420890 (1.4 MB)

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
    inet 127.0.0.1  netmask 255.0.0.0
```

### 📝 `arp` — Address Resolution Protocol

Maps IPv4 addresses on the local network segment to MAC addresses:

```bash
$ arp -n
Address       HWaddress         Flags  Iface
172.31.16.1   06:11:22:33:44:55 C      eth0
```

> 💡 `arp` is how your machine knows which MAC address to send a packet to for a given local IP. If ARP fails, local communication fails.

### 📝 `ifplugstatus` — Link State Check

Checks whether network interfaces have an active **link/carrier** (cable plugged in):

```bash
$ sudo ifplugstatus
eth0: link beat detected         ← ✅ connected
lo: link beat detected           ← ✅ (always, it's loopback)
docker0: link beat not detected  ← no cable (virtual interface, no containers running)
```

---

## 4️⃣ `curl` — HTTP Requests & API Testing

**`curl`** (Client URL) transfers data from or to a server. It's the DevOps workhorse for testing APIs and HTTP endpoints.

### 📝 Basic GET Request

```bash
$ curl http://localhost:5000
Hello from Flask!
```

### 📝 Verbose Mode (See What's Happening)

```bash
$ curl -v http://localhost:5000
*   Trying 127.0.0.1:5000...
* Connected to localhost (127.0.0.1) port 5000
> GET / HTTP/1.1
> Host: localhost:5000
< HTTP/1.1 200 OK
< Content-Type: text/html
<
Hello from Flask!
```

### 📝 POST Requests (Common Pitfall)

```bash
# ❌ WRONG — single hyphen, bash interprets as URL
$ curl -X POST -data "p1=value1&p2=value2" http://example.com
curl: (6) Could not resolve host: p1=value1&p2=value2

# ✅ CORRECT — double hyphen
$ curl -X POST --data "p1=value1&p2=value2" http://example.com

# ✅ Also correct — -d for multiple parameters
$ curl -X POST -d "parm1=value" -d "parm2=value" http://example.com
```

### 📝 Saving Output to a File

```bash
$ curl -o response.txt http://example.com
  % Total    % Received % Xferd
100  1256  100  1256    0     0  45200      0 --:--:-- --:--:--
# Saves the response to 'response.txt' instead of printing to terminal
```

### 📝 Silent Mode + JSON API

```bash
$ curl -s -X GET https://jsonplaceholder.typicode.com/todos/1 | jq
{
  "userId": 1,
  "id": 1,
  "title": "delectus aut autem",
  "completed": false
}
```

### 📋 Useful Flags

| Flag | What it does |
|---|---|
| **`-X METHOD`** | Specify HTTP method (GET, POST, PUT, DELETE) |
| **`-d` / `--data`** | Send data in the request body (POST) |
| **`-H`** | Add a header (`-H "Content-Type: application/json"`) |
| **`-o file`** | Save output to a file |
| **`-s`** | Silent mode (no progress bar) |
| **`-v`** | Verbose (show headers, connection details) |
| **`-L`** | Follow redirects |
| **`-k`** | Skip SSL certificate verification |

> 🧭 **DevOps use case:** `curl` is how you test if an API endpoint works. `curl -s -o /dev/null -w "%{http_code}" http://api.example.com/health` returns just the HTTP status code — perfect for health check scripts.

---

## 5️⃣ `wget` — File Downloads

Downloads files from HTTP/HTTPS URLs to disk:

```bash
$ wget https://example.com/sample.txt
--2026-11-23 14:09:10--  https://example.com/sample.txt
Resolving example.com... 93.184.216.34
Connecting to example.com|93.184.216.34|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1256 (1.2K) [text/plain]
Saving to: 'sample.txt'

sample.txt  100%[==========>]  1.23K  --.-KB/s  in 0s
```

### 📋 Useful Flags

| Flag | What it does |
|---|---|
| **`-O file`** | Save to a specific filename |
| **`-P dir`** | Save to a specific directory |
| **`-c`** | Continue/resume an interrupted download |
| **`-q`** | Quiet mode (no output) |
| **`--recursive`** | Download an entire website (mirroring) |

### 🆚 `curl` vs `wget`

| | `curl` | `wget` |
|---|---|---|
| **Primary use** | Testing APIs, making requests | Downloading files |
| **Output** | Prints to stdout by default | Saves to file by default |
| **Resume** | Limited support | ✅ `-c` flag |
| **Recursive** | ❌ No | ✅ Can mirror sites |

---

## 6️⃣ `jq` — JSON Processing

**`jq`** is a command-line JSON processor. Pipe it after `curl` to pretty-print and extract data from JSON responses:

```bash
# Pretty-print JSON
$ curl -s https://api.example.com/data | jq
{
  "userId": 1,
  "id": 1,
  "title": "delectus aut autem",
  "completed": false
}

# Extract a specific field
$ curl -s https://api.example.com/data | jq '.title'
"delectus aut autem"

# Extract from an array
$ curl -s https://api.example.com/users | jq '.[0].name'
"Alice"

# Filter with conditions
$ curl -s https://api.example.com/todos | jq '.[] | select(.completed == true) | .title'
```

> 🧭 **DevOps tip:** `curl | jq` is the standard pattern for API debugging. Without `jq`, JSON comes back as one long unreadable line. With `jq`, it's instantly formatted and searchable.

---

## 7️⃣ `watch` — Live Monitoring

**`watch`** repeatedly runs a command at a set interval (default 2 seconds) and shows the output live:

```bash
# Watch ping every 2 seconds (default)
$ watch "ping -c 1 127.0.0.1"

# Watch every 5 seconds
$ watch -n 5 "ping -c 1 127.0.0.1"

# Watch interface stats for dropped packets
$ watch "ip -s link show eth0"

# Watch listening ports
$ watch "ss -tln"
```

### 📋 Flags

| Flag | What it does |
|---|---|
| **`-n N`** | Run every N seconds (default 2) |
| **`-d`** | Highlight differences between updates |
| **`-t`** | Turn off header (title + timestamp) |

> 💡 Press `Ctrl+C` to exit `watch`. The `-d` (differences) flag is incredibly useful — it highlights what changed since the last refresh, making it easy to spot anomalies.

---

## 8️⃣ `route` — Legacy Routing Table

The old way to view the routing table (superseded by `ip route`):

```bash
$ route -n
Kernel IP routing table
Destination     Gateway       Genmask         Flags  Iface
0.0.0.0         172.31.16.1   0.0.0.0         UG     eth0
172.31.16.0     0.0.0.0       255.255.240.0   U      eth0
```

- `Destination 0.0.0.0` with `Gateway 172.31.16.1` = your default route (gateway)
- `Destination 172.31.16.0` = your local subnet (no gateway needed, direct via eth0)

> Use `ip route show` instead — it's the modern equivalent.

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **`ping`** | ICMP echo — checks reachability, latency, packet loss. Use `-c N` to limit packets. |
| **`ping -c 4`** | Send 4 packets then stop (don't run forever). |
| **0% packet loss** | Clean reachability. |
| **100% packet loss** | Host down, unreachable, or firewall blocking ICMP. |
| **`traceroute`** | Trace hop-by-hop path to a destination. Shows per-hop timing. |
| **`tracepath`** | Like traceroute + also traces path MTU. |
| **`mtr`** | Combines continuous ping + traceroute. Best for diagnosing latency. `mtr -r -c 3` for report mode. |
| **`* * *` in traceroute** | That hop didn't respond (blocks ICMP). Traffic still flows through. |
| **`ifconfig`** | Legacy interface viewer. Superseded by `ip a`. |
| **`arp -n`** | Maps local IPs to MAC addresses. |
| **`ifplugstatus`** | Checks if interfaces have an active link/carrier (cable plugged in). |
| **`curl`** | Transfer data to/from a server. DevOps workhorse for API testing. |
| **`curl -v`** | Verbose mode — see connection details and headers. |
| **`curl -s`** | Silent mode (no progress bar). |
| **`curl -o file`** | Save output to a file. |
| **`curl -X POST --data`** | Send a POST request with data. Use `--data` (double hyphen), not `-data`. |
| **`curl -L`** | Follow redirects. |
| **`wget`** | Download files from HTTP/HTTPS to disk. `-c` to resume, `--recursive` to mirror. |
| **`jq`** | JSON processor. Pipe after `curl` to pretty-print and extract fields. |
| **`jq '.field'`** | Extract a specific field from JSON. |
| **`watch`** | Repeatedly run a command (default 2s). `-n N` for custom interval, `-d` for differences. |
| **`route -n`** | Legacy routing table viewer. Superseded by `ip route show`. |
| **Package installation** | `apt install net-tools traceroute dnsutils mtr nmap jq whois` (Debian/Ubuntu). |
| **DevOps relevance** | These tools = your diagnostic toolkit. Every "is it working?" question has a specific tool to answer it. |

---

← Back to [Linux Networking](./README.md) | Previous: [Fundamentals](./1-fundamentals-ip-command.md) | Next: [DNS Tools](./3-dns-tools.md) →
