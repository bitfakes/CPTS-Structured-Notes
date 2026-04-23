---
title: Virtual Hosts (VHosts)
tags: [vhosts, subdomains, active-recon, http, tools]
---

# Virtual Hosts (VHosts)
> Virtual hosting lets a single web server (same IP) serve multiple websites by distinguishing them via the HTTP `Host` header.

## VHosts vs Subdomains

| | Subdomains | Virtual Hosts |
|--|-----------|--------------|
| DNS record required | Yes — points to an IP | Not necessarily |
| Discovery via DNS | Yes | Not always — may have no DNS record |
| How distinguished | DNS resolution | `Host` header in HTTP request |

!!! tip
    A VHost without a DNS record is only reachable by adding an entry to your `/etc/hosts` file. VHost fuzzing finds these hidden hosts that won't appear in DNS enumeration.

---

## How VHost Routing Works

1. Browser sends HTTP request with `Host: www.example.com` header.
2. Web server checks its virtual host config for a matching entry.
3. Server serves the corresponding site's files.

The `Host` header acts as a switch — same IP, different sites.

---

## Types of Virtual Hosting

| Type | How it Works | Notes |
|------|-------------|-------|
| **Name-Based** | Uses `Host` header | Most common, cost-effective, needs name-based VHost support |
| **IP-Based** | Unique IP per site | No reliance on `Host` header; expensive (requires multiple IPs) |
| **Port-Based** | Different ports on same IP | Less common, users must specify port in URL |

---

## VHost Fuzzing Tools

| Tool | Description |
|------|-------------|
| [gobuster](https://github.com/OJ/gobuster) | Multi-purpose; fast VHost brute-forcing |
| [Feroxbuster](https://github.com/epi052/feroxbuster) | Rust-based; recursion, wildcard detection |
| [ffuf](https://github.com/ffuf/ffuf) | Web fuzzer; fuzzes `Host` header directly |

---

## Key Commands

```bash
# VHost brute force with gobuster
gobuster vhost -u http://<target_IP> -w <wordlist> --append-domain

# VHost brute force with ffuf (fuzzes the Host header)
ffuf -w <wordlist> -u http://<target_IP> -H "Host: FUZZ.<domain.com>"
```

### Manual VHost Testing (no DNS)
```
# Add to /etc/hosts:
<target_IP>    <vhost.domain.com>
```
