---
title: WHOIS
tags: [whois, passive-recon, osint, dns]
---

# WHOIS
> Query protocol for accessing databases that store information about registered internet resources — domains, IP blocks, autonomous systems.

## What a WHOIS Record Contains

| Field | Description |
|-------|-------------|
| `Domain Name` | The registered domain (e.g., `example.com`) |
| `Registrar` | Where the domain was registered (e.g., GoDaddy, Namecheap) |
| `Registrant Contact` | Person/org that registered the domain |
| `Administrative Contact` | Person managing the domain |
| `Technical Contact` | Person handling technical issues |
| `Creation / Expiration Dates` | When registered and when it expires |
| `Name Servers` | Servers that resolve the domain to an IP |

## Why WHOIS Matters for Recon

- **Identifying Key Personnel** — Names, emails, phone numbers usable for social engineering or phishing.
- **Discovering Network Infrastructure** — Name servers and IPs reveal the hosting provider and potential entry points.
- **Historical Data Analysis** — Services like [WhoisFreaks](https://whoisfreaks.com/) reveal past ownership and infrastructure changes.

---

## Key Commands

```bash
whois <domain.com>
```

### Example Output
```
Domain Name: FACEBOOK.COM
Registrar: RegistrarSafe, LLC
Creation Date: 1997-03-29T05:00:00Z
Registry Expiry Date: 2034-03-30T04:00:00Z
Name Server: A.NS.FACEBOOK.COM
DNSSEC: unsigned
```

---

## Real-World Use Cases

### Phishing Investigation
A suspicious email links to a recently registered domain with hidden registrant info and bulletproof hosting name servers — strong indicators of a phishing campaign.

### Malware Analysis
WHOIS on a C2 domain reveals an anonymous email registrant, high-risk country, and a lax-policy registrar — suggesting a compromised or bulletproof server.

### Threat Intelligence
Analysing WHOIS across multiple attacker domains reveals registration clusters before major attacks, shared name servers, and repeated takedown history — used to build IOC profiles.

!!! tip
    A domain registered very recently + hidden registrant + suspicious name servers = strong phishing/malware indicator.
