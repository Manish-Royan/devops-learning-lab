# 🐧 DevOps Learning Journey — Day 16
## Linux Networking & My First Linux Project

📅 **Date:** November 23, 2026

Today was an important milestone in my Linux and DevOps learning journey. I spent time strengthening my understanding of Linux networking and also worked on documenting and building my first hands-on Linux project.

Rather than just learning individual commands, my focus was on understanding how these concepts connect to real-world infrastructure troubleshooting.

---

## 🌐 Part 1: [Linux Networking](../../notes/linux/13-linux-networking/)

I studied Linux networking from the perspective of a DevOps engineer, focusing on how to inspect, understand, and troubleshoot network-related problems on Linux systems.

### Topics covered

**1. Network Fundamentals and the `ip` Command**
- IP addresses and network interfaces
- Loopback, Ethernet, wireless, and virtual interfaces
- MAC addresses
- Subnets and subnet masks
- Private and public IP addresses
- Ports and their purpose
- Difference between `0.0.0.0` and `127.0.0.1`
- Using `ip addr`, `ip link`, and `ip route`

**2. Network Diagnostic Tools**
- `ping` — reachability, latency, and packet loss
- `traceroute`, `tracepath`, and `mtr` — understanding network paths
- `ifconfig`, `arp`, and `ifplugstatus`
- `curl` and `wget` — testing APIs and downloading resources
- `jq` — processing JSON responses
- `watch` — repeatedly monitoring command output

**3. DNS Tools**
- Understanding DNS and domain resolution
- DNS record types: A, AAAA, CNAME, MX, TXT, and NS
- TTL (Time To Live)
- `nslookup`, `dig`, and `whois`

**4. Socket Inspection**
- Understanding sockets
- Using `ss` to inspect network connections
- Identifying listening ports and associated processes
- Understanding common socket states

**5. TCP and Packet Analysis**
- TCP three-way handshake: SYN → SYN-ACK → ACK
- Understanding TCP connection establishment
- Using `tcpdump` to capture and inspect packets
- Recognizing basic TCP flags in packet output

**6. Firewalls and Network Security**
- Understanding Linux host firewalls
- UFW and iptables
- Cloud Security Groups
- How firewall rules affect incoming connections
- Understanding reverse-proxy patterns

### The troubleshooting mindset

One important takeaway was learning to approach networking problems systematically.

For example, an application might work through `localhost` but fail when accessed remotely. Troubleshooting that problem requires checking more than just the application.

My notes organize the investigation around this flow:

`Interfaces → Listening Sockets → Packet Capture → Firewalls → DNS`

This helped me connect individual networking commands to a practical troubleshooting process.

---

## 🛠️ Part 2: My First Linux Project

I also worked on my first hands-on Linux project:

### [System Health & Performance Monitoring on Linux](../../projects/linux-projects/Project%201:%20System%20Health%20&%20Performance%20Monitoring/README.md)

The goal was to build a small Linux monitoring and performance-testing lab using **Vagrant** and a Rocky Linux 9 virtual machine.

### Technologies and tools used
- Linux — Rocky Linux 9
- Vagrant — virtual machine lifecycle management
- libvirt/KVM — virtualization provider on Fedora
- Bash — monitoring and automation scripts
- Python — memory-leak simulation
- `df`, `awk`, and `cut` — disk usage monitoring
- `sar` and `iostat` — system performance analysis
- `dd` — generating disk I/O activity
- Slack Webhooks — alert notifications

### What I implemented

**1. Virtual Machine Provisioning**
- Created a Vagrantfile to define the virtual machine.
- Configured a Rocky Linux 9 environment.
- Configured CPU and memory resources.
- Set up a private network and synced folders.
- Automated swap-space creation through shell provisioning.

**2. Disk Monitoring and Alerting**
- Created a Bash script to check disk usage.
- Used `df`, `awk`, and `cut` to extract disk usage information.
- Added threshold-based alerting.
- Explored sending notifications through a Slack webhook.

**3. Memory-Leak Simulation**
- Created a Python script that continuously allocates memory.
- Used the experiment to observe memory consumption, swap behavior, and OOM-killer-related behavior.

**4. Disk I/O Stress Testing**
- Used `dd` to generate disk I/O activity.
- Explored how disk activity can be monitored using Linux performance tools.

**5. System Performance Reports**
- Used `sar` to collect CPU, memory, and disk activity information.
- Created a Bash script to generate daily system activity reports.

This project gave me an opportunity to connect Linux commands, scripting, virtualization, monitoring, and system administration in one practical environment.

---

## 💡 Key Takeaways

- Linux networking is not just about knowing commands; it is about understanding how packets travel and where communication can fail.
- `ip`, `ss`, `tcpdump`, `dig`, and firewall tools answer different questions during troubleshooting.
- A service listening on localhost is not necessarily accessible from another machine.
- Vagrant makes it easier to create reproducible Linux lab environments.
- Monitoring CPU, memory, disk usage, and I/O helps us understand system behavior.
- Automation scripts can turn repetitive administrative tasks into repeatable workflows.

---

## 🎯 What's Next?

I'll continue strengthening my understanding of Linux networking and infrastructure. I'm also moving forward with deeper DNS Server learning, which I want to document separately once my notes are ready.

This is still a learning journey, and every lab gives me another opportunity to understand how Linux systems behave in practice.

**Learning in public, one step at a time.** 🚀