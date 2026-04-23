---
title: Introduction to Web Reconnaissance
tags: [passive-recon, active-recon, osint]
---

# Introduction to Web Reconnaissance
> Foundation of a security assessment — systematically collecting information about a target before analysis or exploitation.

## Goals of Web Reconnaissance

- **Identifying Assets** — Uncover all publicly accessible components: web pages, subdomains, IP addresses, technologies.
- **Discovering Hidden Information** — Locate sensitive data inadvertently exposed: backup files, config files, internal docs.
- **Analysing the Attack Surface** — Examine technologies, configurations, and potential entry points for exploitation.
- **Gathering Intelligence** — Collect information for further exploitation or social engineering: key personnel, email addresses, behavioural patterns.

---

## Active Reconnaissance
> Directly interacts with the target. Higher detection risk.

| Technique | Description | Tools | Detection Risk |
|-----------|-------------|-------|---------------|
| Port Scanning | Identify open ports and services | Nmap, Masscan, Unicornscan | High |
| Vulnerability Scanning | Probe for known vulnerabilities | Nessus, OpenVAS, Nikto | High |
| Network Mapping | Map network topology | Traceroute, Nmap | Medium–High |
| Banner Grabbing | Retrieve service banners | Netcat, curl | Low |
| OS Fingerprinting | Identify the target OS | Nmap (`-O`), Xprobe2 | Low |
| Service Enumeration | Determine service versions on open ports | Nmap (`-sV`) | Low |
| Web Spidering | Crawl the target website for pages/files | Burp Suite Spider, OWASP ZAP, Scrapy | Low–Medium |

---

## Passive Reconnaissance
> No direct interaction with the target. Relies on public information. Very low detection risk.

| Technique | Description | Tools |
|-----------|-------------|-------|
| Search Engine Queries | Find info via Google/Bing/Shodan | Google, DuckDuckGo, Shodan |
| WHOIS Lookups | Domain registration details | `whois`, online WHOIS services |
| DNS | Analyse DNS records for subdomains, mail servers | `dig`, `nslookup`, `dnsenum`, `fierce` |
| Web Archive Analysis | View historical snapshots of the target | Wayback Machine |
| Social Media Analysis | Gather employee and org info | LinkedIn, Twitter, OSINT tools |
| Code Repositories | Find exposed credentials or vulnerabilities | GitHub, GitLab |

!!! tip
    Passive recon is stealthier but may yield less than active. Combining both gives the most thorough results.
