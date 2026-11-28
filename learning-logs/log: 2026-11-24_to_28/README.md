# 🐧 DevOps Learning Journey — Days 21
## Deep Dive into Linux DNS, Apache Web Server & Building a DNS Resolver in Go

📅 **Learning period:** November 23–28, 2026

This was one of the most intensive learning periods in my DevOps journey so far.

For almost six days, I dedicated my learning time to understanding Linux DNS in depth. I didn't want to stop at memorizing DNS commands or understanding the theory. I wanted to configure an actual DNS server, connect it to a web server, test it from different devices, and finally implement the resolution process in code.

Here's what I explored and built.

---

## 🌐 Part 1: Understanding DNS

I started by strengthening my understanding of the Domain Name System (DNS), the system that translates human-readable domain names into IP addresses.

### Topics covered

- What DNS is and why it exists
- The `/etc/hosts` file and its purpose
- DNS messages and port 53
- Domain names, domain hierarchy, and TLDs
- Forward and reverse DNS lookups
- Recursive and iterative queries
- DNS resolvers and the resolution process
- Root, TLD, and authoritative name servers
- DNS caching, TTL, and anycast
- DNS zones and delegation
- Resource records:
  - A and AAAA
  - CNAME
  - MX
  - NS
  - SOA
  - PTR
  - TXT
  - SRV
  - CAA

I also explored DNS troubleshooting using tools such as `dig`, `nslookup`, and `resolvectl`.

One important realization was that DNS resolution is not always as straightforward as running `nslookup`. Different applications can use different name-resolution paths, and browser settings, system resolvers, and caching can affect the result.

---

## 🖥️ Part 2: Building a Custom Domain Web Server

After understanding the concepts, I wanted to see DNS working in a real environment.

I used my Fedora Linux machine to build a small local infrastructure lab.

### Technologies used

- **Apache HTTP Server (`httpd`)** — Web server
- **BIND (`named`)** — DNS server
- **firewalld** — Firewall management
- **Fedora Linux** — Host environment

### What I implemented

1. Installed and configured Apache.
2. Hosted a custom static web page.
3. Installed and configured BIND.
4. Created a DNS zone and configured its records.
5. Validated DNS configuration using `named-checkconf` and `named-checkzone`.
6. Opened the required DNS and HTTP firewall services.
7. Tested DNS resolution using `dig` and `nslookup`.
8. Configured my laptop to use the local DNS server.
9. Configured an Android phone connected to the same Wi-Fi to use that DNS server.
10. Successfully opened the custom-domain website from both devices.

### Understanding how DNS and HTTP work together

One of my biggest takeaways was understanding that DNS does not forward a visitor to Apache.

DNS resolves a domain name to an IP address. The browser then independently connects to the web server using HTTP.

For my lab:

- BIND answered DNS queries on port `53`.
- Apache served the website on port `80`.
- Both services ran on the same machine but served different purposes.

I also encountered real troubleshooting challenges involving Fedora's local resolver, `.local` and multicast DNS, browser Secure DNS, Android Private DNS, and DNS settings.

Those problems taught me that successful DNS resolution from `dig` does not automatically guarantee that a browser will resolve the same name.

---

## 🚀 Part 3: Building My Own DNS Resolver in Go

After spending so much time understanding DNS conceptually and configuring BIND, I wanted to take the next step.

**Why not implement a small DNS resolver myself?**

That idea led to my Go project:

**Repository:** https://github.com/Manish-Bishwakarma/go-dns-resolver

I built a small recursive DNS resolver in Go that listens for client queries over UDP and performs the resolution process by walking the DNS hierarchy itself.

### What the resolver does

1. Receives a DNS question from a client.
2. Sends a query to a root DNS server.
3. Follows referrals to the appropriate TLD name servers.
4. Follows referrals to the domain's authoritative name servers.
5. Handles the resulting answer, including following CNAME records.
6. Sends the final response back to the original client.

### Concepts explored while implementing it

- DNS message structure and parsing
- DNS transaction IDs
- Root, TLD, and authoritative server referrals
- Glue records
- CNAME resolution
- UDP communication
- TCP fallback when a UDP response is truncated
- DNS response validation
- Context-based timeouts
- Graceful server shutdown
- Unit and integration testing

I also documented the project architecture, server implementation, resolver traversal, DNS exchanges, and tests.

The project currently includes **26 tests**, helping me verify important parts of the resolver's behavior.

This project helped me connect the DNS concepts I had studied with actual networking and Go programming.

---

## 💡 What I learned from this journey

- DNS is more than simply translating a domain name into an IP address.
- Recursive and iterative resolution are important concepts to understand when working with DNS infrastructure.
- A DNS server and a web server are separate services, even when hosted on the same machine.
- DNS troubleshooting requires understanding the client, operating system, resolver, browser, and network configuration.
- Building a DNS resolver made concepts like referrals, glue records, CNAMEs, UDP, and TCP much more concrete.
- Learning a technology becomes more meaningful when I can implement a small version of its underlying behavior.

---

## 📂 Related Work

**Linux DNS and Web Server Notes:**  
https://github.com/Manish-Royan/devops-learning-lab/tree/main/notes/linux/14-dns-and-web-server

**Go DNS Resolver:**  
https://github.com/Manish-Bishwakarma/go-dns-resolver

---

## 🎯 What's Next?

After nearly six days of intensive learning and hands-on experimentation with DNS, I'm taking a short break to recharge.

When I return, I'll continue strengthening my Linux knowledge, explore **Nginx**, and then take my next big step into the **world of containerization**.

**Learning in public, experimenting in practice, and growing one step at a time.** 🚀