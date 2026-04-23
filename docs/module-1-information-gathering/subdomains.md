---
title: Subdomains & Brute Force Enumeration
tags: [subdomains, active-recon, passive-recon, dns, tools]
---

# Subdomains & Brute Force Enumeration
> Subdomains are extensions of the main domain (e.g., `blog.example.com`). They often host valuable — and less-secured — content.

## Why Subdomains Matter

- **Dev/Staging Environments** — Often have relaxed security, may expose vulnerabilities or sensitive data.
- **Hidden Login Portals** — Admin panels not linked from the main site.
- **Legacy Applications** — Forgotten apps running outdated software with known CVEs.
- **Sensitive Information** — Config files, internal docs accidentally exposed.

---

## Subdomain Enumeration Methods

### Passive Enumeration
No direct interaction with the target's DNS servers.

- **Certificate Transparency Logs** — SSL certs list subdomains in their SAN field (see [CT Logs](certificate-transparency.md)).
- **Search Engine Operators** — `site:example.com -www` discovers indexed subdomains.
- **DNS/WHOIS Aggregators** — Online databases that aggregate DNS data.

### Active Enumeration
Directly queries the target's DNS infrastructure.

- **DNS Zone Transfer** — Misconfigured servers may leak the full zone (see [Zone Transfers](dns-zone-transfers.md)).
- **Brute Force** — Systematically test a wordlist of potential subdomain names.

!!! tip
    Combining passive and active approaches gives the most complete coverage. Start passive to stay stealthy, then go active.

---

## Subdomain Brute Force — How It Works

1. **Wordlist Selection** — General-purpose (`dev`, `staging`, `admin`, `mail`, `test`) or targeted/custom lists.
2. **Iteration** — Tool appends each word to the domain: `dev.example.com`, `staging.example.com`, etc.
3. **DNS Lookup** — Queries A/AAAA records for each potential subdomain.
4. **Validation** — Resolving subdomains are added to the results list.

---

## Brute Force Tools

| Tool | Description |
|------|-------------|
| [dnsenum](https://github.com/fwaeytens/dnsenum) | DNS enumeration, dictionary attacks, zone transfers, Google scraping |
| [fierce](https://github.com/mschwager/fierce) | Recursive subdomain discovery, wildcard detection |
| [dnsrecon](https://github.com/darkoperator/dnsrecon) | Multiple techniques, customisable output |
| [amass](https://github.com/owasp-amass/amass) | Actively maintained, integrates many data sources |
| [assetfinder](https://github.com/tomnomnom/assetfinder) | Lightweight, quick scans |
| [puredns](https://github.com/d3mondev/puredns) | High-performance DNS brute-forcing with filtering |

---

## Key Commands

```bash
# dnsenum brute force
dnsenum --enum <domain.com> -f /usr/share/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -r

# dnsenum with specific nameserver
dnsenum --enum <domain.com> -f <wordlist> --dnsserver <nameserver>
```

!!! tip
    SecLists (`/usr/share/SecLists/Discovery/DNS/`) contains several wordlists of varying sizes. Start with `subdomains-top1million-20000.txt` for balanced coverage and speed.
