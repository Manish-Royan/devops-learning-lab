# dns-resolver: Project Documentation

A small recursive DNS resolver written in Go. It listens on a local UDP port and answers DNS questions by walking the DNS tree itself. It asks a root server first, then the server for the top-level domain (for example `.com`), and then the domain's own name server.

I wrote it to understand how a DNS lookup really works, after setting up Apache for a domain I own. These notes go through the code file by file and function by function, so each behaviour can be traced back to the line that causes it.

---

## What the program does

A client such as `dig` sends one question over UDP. The server checks the packet, asks the resolver to find the answer, and sends back one reply.

The resolver does not know the answer at the start. It asks a root server, which points it to the `.com` servers. The `.com` servers point it to the name servers for the domain, and those give the answer. If the answer is an alias (a CNAME record), the resolver looks up the target as well.

---

## Reading order

| Order | File | What it covers |
|---|---|---|
| 1 | [01-architecture.md](01-architecture.md) | The packages, what each one knows, and where each function lives |
| 2 | [02-server.md](02-server.md) | The client side: UDP listener, packet checks, reply codes, EDNS, truncation, shutdown |
| 3 | [03-resolver-walk.md](03-resolver-walk.md) | The walk: referrals, glue, CNAMEs, limits |
| 4 | [04-exchange-and-trust.md](04-exchange-and-trust.md) | One query to one server: IDs, UDP and TCP, reply checks |
| 5 | [05-tests.md](05-tests.md) | What the 26 tests check, and how the fake DNS server works |

---

## Project layout

```
dns-resolver/
├── cmd/dns-resolver/main.go          flags, logger, shutdown on Ctrl+C, starts the server
├── internal/server/
│   ├── server.go                     UDP listener, packet checks, reply building
│   └── server_test.go                tests for the client side
├── internal/resolver/
│   ├── resolver.go                   the walk: referrals, CNAMEs, limits
│   ├── exchange.go                   one query to one server (UDP, TCP fallback, reply checks)
│   ├── roots.go                      the 13 root server addresses
│   └── resolver_test.go              tests for the walk, using a fake DNS server
├── go.mod                            module path, Go version, dependencies
└── go.sum
```

`internal/` makes the two packages private to this module, so other Go projects cannot import them.

The only external dependency is `golang.org/x/net/dns/dnsmessage`. It turns DNS messages into bytes and back.

---

## Running it

```bash
go run ./cmd/dns-resolver                            # listens on 127.0.0.1:1053
dig @127.0.0.1 -p 1053 www.amazon.com                # in a second terminal
go run ./cmd/dns-resolver -v                         # print every query sent upstream
go run ./cmd/dns-resolver -upstream 1.1.1.1,8.8.8.8  # forwarding mode
```

Port 1053 is used because ports below 1024 need root. On Fedora, `systemd-resolved` already listens on port 53 at `127.0.0.53` and `127.0.0.54`.

You need Go 1.22 or newer, because the code uses range over integers and the built-in `min` and `max`.

---

## Tests

```bash
go test ./...
go test -race ./...
```

All 26 tests pass with both commands. None of them uses the Internet. Each one starts a fake DNS server on localhost and checks how the code reacts to it.

---

> ### ⚠️ NOTE:This is for learning, not a replacement for `systemd-resolved`.

---
