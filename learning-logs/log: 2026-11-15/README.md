# 🐧 DevOps Learning Journey — Day 8

Today was a little different from my usual learning days. I explored Linux Cron jobs and, more importantly, restarted my Computer Networking studies with a renewed perspective.

## 1. Linux: Cron Jobs — Scheduling Automation

Today I learned the fundamentals of Cron, Linux's time-based job scheduler.

Topics explored:

- What Cron jobs are and why scheduled automation matters
- The role of the `crond` daemon
- Checking and managing the Cron service with `systemctl`
- Managing user crontabs with `crontab -e`, `crontab -l`, and `crontab -r`
- Understanding the five-field Cron expression:
  `minute hour day-of-month month day-of-week`
- Common scheduling patterns such as `*/15`, ranges, and lists
- Special scheduling strings such as `@reboot`, `@daily`, and `@weekly`
- Redirecting standard output and errors to log files
- Basic Cron troubleshooting and the importance of absolute paths

### Why this matters for DevOps

A script defines *what* should happen; a scheduler defines *when* it should happen.

Cron is useful for automating repetitive server tasks such as backups, health checks, cleanup jobs, and scheduled reports. It introduced me to another important part of Linux automation.

I explored the fundamentals today rather than going deeply into production scheduling.

## 2. Restarting Computer Networking — This Time with a DevOps Perspective

This is the part that made today special.

I had already studied Computer Networking during my academic journey. However, I have decided to revisit the subject and learn it more deeply, this time with a practical DevOps and infrastructure perspective.

My goal is not simply to memorize networking definitions and protocols. I want to understand how machines communicate, how networks are structured, how traffic moves, and how to troubleshoot real infrastructure problems.

### Topics revisited today

- What a computer network is
- Why devices need addressing, communication rules, and protocols
- Network types: LAN, WAN, MAN, and WLAN
- Specialized network concepts: mobile networks, IoT, SDN, and air-gapped networks
- Software-Defined Networking (SDN)
- Air-gapped networks and network isolation
- The Internet versus the World Wide Web
- What network protocols are and why they matter
- An introductory view of network protocol layers
- Familiar protocols: TCP, HTTP/HTTPS, SMTP, FTP, and Telnet
- Email communication basics: SMTP and POP3

### Why I am revisiting networking for DevOps

Modern infrastructure is distributed. Applications, databases, containers, virtual machines, cloud services, and monitoring systems all communicate over networks.

Understanding networking will help me reason about problems such as:

- Why one service cannot communicate with another
- DNS resolution and connectivity failures
- Ports, firewalls, routing, and network boundaries
- Communication between containers and Kubernetes workloads
- Latency and reliability in distributed applications

I have only restarted the fundamentals today. There is much more to explore, and I intend to build that understanding step by step.

## Today's Reflection

One thing I am realizing is that learning DevOps is not just about learning tools such as Docker, Kubernetes, Jenkins, or Terraform.

It is also about understanding the foundations beneath those tools.

Linux teaches me how systems operate. Networking teaches me how those systems communicate. Automation teaches me how to operate them efficiently and consistently.

Today was a small step, but an important one.

**Learning is not always about covering more topics. Sometimes it is about returning to an important foundation and understanding why it matters in the real world.**

---

*This is Day 8 of my public DevOps learning journey. I am documenting what I study, what I understand, and what I still need to improve — one day at a time.*
