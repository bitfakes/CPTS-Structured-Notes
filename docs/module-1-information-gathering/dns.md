---
title: DNS & Digging DNS
tags: [dns, passive-recon, tools]
---

# DNS & Digging DNS
> The Domain Name System translates human-readable domain names (e.g., `www.example.com`) into IP addresses (e.g., `192.0.2.1`) that computers use to communicate.

## How DNS Resolution Works

1. **DNS Query** — Your computer checks its local cache first. If no result, it queries the DNS resolver (usually your ISP).
2. **Recursive Lookup** — The resolver checks its own cache, then contacts a root name server.
3. **Root Name Server** — Doesn't know the answer but points to the correct TLD name server (e.g., `.com`).
4. **TLD Name Server** — Points to the authoritative name server for the specific domain.
5. **Authoritative Name Server** — Holds the actual IP address and returns it.
6. **Response Cached** — The resolver caches the answer and returns it to your computer.
7. **Connection** — Your computer connects to the web server at the resolved IP.

---

## The Hosts File

Bypasses DNS entirely — maps hostnames to IPs locally.

| OS | Location |
|----|---------|
| Windows | `C:\Windows\System32\drivers\etc\hosts` |
| Linux/macOS | `/etc/hosts` |

**Format:**
```
<IP Address>    <Hostname> [<Alias>]
```

**Common uses:**
```
127.0.0.1       myapp.local          # redirect to local dev server
192.168.1.20    testserver.local     # test connectivity
0.0.0.0         unwanted-site.com    # block a site
```

---

## DNS Record Types

| Record | Full Name | Description | Example |
|--------|-----------|-------------|---------|
| `A` | Address Record | IPv4 address for a hostname | `www.example.com → 192.0.2.1` |
| `AAAA` | IPv6 Address Record | IPv6 address for a hostname | `www.example.com → 2001:db8::1` |
| `CNAME` | Canonical Name | Alias pointing to another hostname | `blog.example.com → webserver.net` |
| `MX` | Mail Exchange | Mail server for the domain | `example.com → mail.example.com` |
| `NS` | Name Server | Authoritative name server for the zone | `example.com → ns1.example.com` |
| `TXT` | Text Record | Arbitrary text; used for SPF, DKIM, verification | `"v=spf1 mx -all"` |
| `SOA` | Start of Authority | Admin info about the zone | Serial, refresh, retry, expire |
| `SRV` | Service Record | Hostname and port for specific services | `_sip._udp.example.com` |
| `PTR` | Pointer Record | Reverse DNS lookup (IP → hostname) | `1.2.0.192.in-addr.arpa → www.example.com` |

!!! tip
    `TXT` records often leak tech stack info — a value like `_1password=...` reveals the org uses 1Password, useful for targeted social engineering.

---

## Why DNS Matters for Recon

- **Uncovering Assets** — `CNAME` records pointing to old servers can expose vulnerable systems.
- **Mapping Infrastructure** — `NS` records reveal hosting providers; `A` records for `loadbalancer.example.com` reveal architecture.
- **Monitoring for Changes** — New subdomains (`vpn.example.com`) may indicate new entry points.

---

## DNS Tools

| Tool | Best For |
|------|---------|
| `dig` | Versatile, detailed output, supports all record types |
| `nslookup` | Simple lookups, quick A/AAAA/MX checks |
| `host` | Concise output, quick lookups |
| `dnsenum` | Automated enumeration, zone transfers, brute force |
| `fierce` | Recursive subdomain discovery, wildcard detection |
| `dnsrecon` | Comprehensive enumeration, multiple output formats |
| `theHarvester` | OSINT: emails + DNS data from multiple sources |

---

## Key Commands

```bash
# Default A record lookup
dig <domain.com>

# Specific record type
dig <domain.com> A
dig <domain.com> AAAA
dig <domain.com> MX
dig <domain.com> NS
dig <domain.com> TXT
dig <domain.com> CNAME
dig <domain.com> SOA

# Query a specific name server
dig @1.1.1.1 <domain.com>

# Trace full resolution path
dig +trace <domain.com>

# Reverse lookup
dig -x <192.168.1.1>

# Concise output
dig +short <domain.com>

# Answer section only
dig +noall +answer <domain.com>

# All records (note: many servers block ANY queries per RFC 8482)
dig <domain.com> ANY
```

### Reading dig Output

```
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20123
;; flags: qr rd ra
```

| Flag | Meaning |
|------|---------|
| `qr` | This is a query response |
| `rd` | Recursion was requested |
| `ra` | Recursion is available |
| `ad` | Resolver considers data authentic |

```
;; ANSWER SECTION:
facebook.com.    1    IN    A    163.70.144.35
```
- `1` = TTL in seconds
- `IN` = Internet protocol class (standard for all modern DNS)
- `A` = Record type
