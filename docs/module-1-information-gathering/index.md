---
title: Module 1 - Information Gathering (Web Edition)
tags: [passive-recon, active-recon, dns, subdomains, fingerprinting, crawling, osint, cheatsheet]
---

# Module 1 - Information Gathering (Web Edition)
> Quick-reference cheatsheet for all recon techniques covered in this module.

---

## Summary

Web reconnaissance using both **passive** (no direct target interaction) and **active** (direct queries) techniques. The goal is to map the target's infrastructure — domains, subdomains, technologies, exposed files, and historical data — before any exploitation attempt.

| Phase | Techniques |
|-------|-----------|
| **Passive** | WHOIS, DNS records, CT logs, Google dorking, web archives, OSINT |
| **Active** | DNS queries, subdomain brute-force, zone transfers, VHost fuzzing, crawling, fingerprinting |

**Key tools:** `whois`, `dig`, `dnsenum`, `gobuster`, `ffuf`, `curl`, `wafw00f`, `nikto`, `scrapy/ReconSpider`, `FinalRecon`

---

## Reconnaissance / Information Gathering

### WHOIS
```bash
whois <domain.com>
```

### DNS Tools
```bash
# A record (default)
dig <domain.com>

# Query specific record type
dig <domain.com> <RECORD_TYPE>

# Reverse lookup
dig -x <x.x.x.x>

# Zone transfer attempt
dig axfr @<nameserver> <domain.com>
```

**DNS Tools:** `dig`, `nslookup`, `host`, `dnsenum`, `fierce`, `dnsrecon`, `theHarvester`

---

### Subdomain Enumeration

#### Passive
- Certificate Transparency Logs (`crt.sh`, Censys)
- Search engine operators (`site:example.com -www`)
- WHOIS / DNS records

#### Active — Brute Force
```bash
dnsenum --enum <domain.com> -f /usr/share/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -r
```
**Tools:** `dnsenum`, `fierce`, `dnsrecon`, `amass`, `assetfinder`, `puredns`

#### Active — DNS Zone Transfer
```bash
dig axfr @<nameserver> <domain.com>
# Test server: dig axfr @nsztm1.digi.ninja zonetransfer.me
```

#### Active — Virtual Host Fuzzing
```bash
gobuster vhost -u http://<target_IP> -w <wordlist> --append-domain
```
**Tools:** `gobuster`, `feroxbuster`, `ffuf`

#### Certificate Transparency Logs
```bash
curl -s "https://crt.sh/?q=<domain.com>&output=json" | jq -r '.[] | select(.name_value | contains("<domain>")) | .name_value' | sort -u
```

---

## Fingerprinting

**Techniques:** Banner Grabbing, HTTP Header Analysis, WAF Detection, CMS Detection

```bash
# Banner grab
curl -I https://<domain.com>

# WAF detection
wafw00f <domain.com>

# Install wafw00f
sudo pip3 install git+https://github.com/EnableSecurity/wafw00f

# Nikto fingerprinting scan
nikto -h <domain.com> -Tuning b
```

**Tools:** `Wappalyzer`, `BuiltWith`, `WhatWeb`, `Nmap`, `Netcraft`, `wafw00f`, `Nikto`

**CMS Detection:** [CMSeeK](https://github.com/Tuhinshubhra/CMSeeK)

---

## Crawling

```bash
# Install Scrapy
sudo pip3 install scrapy

# Download and run ReconSpider
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
unzip ReconSpider.zip
python3 ReconSpider.py http://<target.com>
```

**Tools:** Burp Suite Spider, OWASP ZAP, Scrapy, Apache Nutch

---

## Search Engine Discovery (Google Dorking)

| Operator | Purpose | Example |
|----------|---------|---------|
| `site:` | Limit to domain | `site:example.com` |
| `inurl:` | Term in URL | `inurl:login` |
| `filetype:` | File type | `filetype:pdf` |
| `intitle:` | Term in title | `intitle:"confidential"` |
| `intext:` | Term in body | `intext:"password reset"` |
| `cache:` | Cached version | `cache:example.com` |

**Reference:** [Google Hacking Database](https://www.exploit-db.com/google-hacking-database)

---

## Web Archives

**Tool:** [Wayback Machine](https://web.archive.org/) — view historical snapshots of any website.

---

## Automated Reconnaissance Frameworks

| Tool | Description |
|------|-------------|
| [FinalRecon](https://github.com/thewhiteh4t/FinalRecon) | All-in-one: headers, WHOIS, SSL, crawl, DNS, subdomains, wayback |
| [ReconFTW](https://github.com/six2dez/reconftw) | Full-automated recon framework |
| [Argus](https://github.com/jasonxtn/Argus) | Recon automation |
| [Recon-ng](https://github.com/lanmaster53/recon-ng) | Modular recon framework |
| [SpiderFoot](https://github.com/smicallef/spiderfoot) | OSINT automation |

```bash
# FinalRecon - headers + WHOIS
./finalrecon.py --headers --whois --url http://<target.com>

# FinalRecon - full scan
./finalrecon.py --full --url http://<target.com>
```

---

## All Commands

### WHOIS
```bash
whois <domain.com>
```

### DNS
```bash
# Default A record lookup
dig <domain.com>

# Specific record types
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

# Reverse lookup (IP → hostname)
dig -x <192.168.1.1>

# Concise output
dig +short <domain.com>

# Answer section only
dig +noall +answer <domain.com>

# All record types
dig <domain.com> ANY

# Zone transfer
dig axfr @<nameserver> <domain.com>
# Test server: dig axfr @nsztm1.digi.ninja zonetransfer.me
```

### Subdomain Brute Force
```bash
# dnsenum with SecLists wordlist
dnsenum --enum <domain.com> -f /usr/share/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -r

# dnsenum with specific nameserver
dnsenum --enum <domain.com> -f <wordlist> --dnsserver <nameserver>
```

### Virtual Host Fuzzing
```bash
# gobuster vhost
gobuster vhost -u http://<target_IP> -w <wordlist> --append-domain

# ffuf — fuzzes Host header directly
ffuf -w <wordlist> -u http://<target_IP> -H "Host: FUZZ.<domain.com>"
```

### Certificate Transparency Logs
```bash
curl -s "https://crt.sh/?q=<domain.com>&output=json" \
  | jq -r '.[] | select(.name_value | contains("<domain>")) | .name_value' \
  | sort -u
```

### Fingerprinting
```bash
# Banner grab — fetch HTTP headers only
curl -I http://<domain.com>
curl -I https://<domain.com>

# Install wafw00f
sudo pip3 install git+https://github.com/EnableSecurity/wafw00f

# WAF detection
wafw00f <domain.com>

# Install Nikto
sudo apt update && sudo apt install -y perl
git clone https://github.com/sullo/nikto
cd nikto/program && chmod +x ./nikto.pl

# Nikto fingerprinting scan (Software Identification)
nikto -h <domain.com> -Tuning b
```

### Crawling
```bash
# Install Scrapy
sudo pip3 install scrapy

# Download and run ReconSpider
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
unzip ReconSpider.zip
python3 ReconSpider.py http://<target.com>
```

### Automated Recon
```bash
# FinalRecon — headers + WHOIS only
./finalrecon.py --headers --whois --url http://<target.com>

# FinalRecon — full scan
./finalrecon.py --full --url http://<target.com>
```
