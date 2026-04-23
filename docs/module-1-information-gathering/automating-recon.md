---
title: Automating Recon
tags: [active-recon, passive-recon, tools, osint, automation]
---

# Automating Recon
> Automation frameworks combine multiple recon techniques into a single tool — DNS enumeration, subdomain discovery, crawling, port scanning, WHOIS, and more — reducing time and human error.

## Why Automate?

- **Efficiency** — Automated tools execute repetitive tasks far faster than manual work.
- **Scalability** — Cover many targets or domains without proportional effort increase.
- **Consistency** — Predefined rules produce reproducible results.
- **Comprehensive Coverage** — Single run covers DNS, subdomains, crawling, port scanning, certificates, and more.

---

## Reconnaissance Frameworks

| Tool | Description |
|------|-------------|
| [FinalRecon](https://github.com/thewhiteh4t/FinalRecon) | Python-based all-in-one: SSL, WHOIS, headers, crawl, DNS, subdomains, wayback, port scan |
| [ReconFTW](https://github.com/six2dez/reconftw) | Full-automated recon framework covering a wide attack surface |
| [Argus](https://github.com/jasonxtn/Argus) | Recon automation |
| [Recon-ng](https://github.com/lanmaster53/recon-ng) | Modular Python framework — DNS, subdomains, port scanning, known CVE exploitation |
| [theHarvester](https://github.com/laramies/theHarvester) | Emails, subdomains, hosts, employee names, open ports from search engines and Shodan |
| [SpiderFoot](https://github.com/smicallef/spiderfoot) | OSINT automation — IPs, domains, emails, social profiles via multiple data sources |
| [OSINT Framework](https://osintframework.com/) | Collection of OSINT tools and resources (social media, search engines, public records) |

---

## FinalRecon

FinalRecon covers the full recon workflow in a single tool:

- Headers, WHOIS, SSL certificate details
- Web crawling (HTML, CSS, JS, internal/external links, robots.txt, sitemaps)
- DNS enumeration (40+ record types including DMARC)
- Subdomain enumeration (crt.sh, AnubisDB, ThreatMiner, CertSpotter, VirusTotal, Shodan, BeVigil)
- Directory enumeration
- Wayback Machine URLs (last 5 years)

### Setup

```bash
git clone https://github.com/thewhiteh4t/FinalRecon.git
cd FinalRecon
pip3 install -r requirements.txt
chmod +x ./finalrecon.py
./finalrecon.py --help
```

### Key Commands

```bash
# Headers + WHOIS
./finalrecon.py --headers --whois --url http://<target.com>

# Full recon
./finalrecon.py --full --url http://<target.com>

# Specific modules
./finalrecon.py --sslinfo --url http://<target.com>
./finalrecon.py --dns --url http://<target.com>
./finalrecon.py --sub --url http://<target.com>
./finalrecon.py --crawl --url http://<target.com>
./finalrecon.py --wayback --url http://<target.com>
```

### FinalRecon Options

| Flag | Description |
|------|-------------|
| `--url` | Target URL |
| `--headers` | HTTP header info |
| `--sslinfo` | SSL certificate details |
| `--whois` | WHOIS lookup |
| `--crawl` | Crawl the target |
| `--dns` | DNS enumeration |
| `--sub` | Subdomain enumeration |
| `--dir` | Directory search |
| `--wayback` | Wayback Machine URLs |
| `--ps` | Fast port scan |
| `--full` | Full recon (all modules) |
| `-w` | Custom wordlist path |
| `-e` | File extensions (e.g., `txt,php,xml`) |
| `-o` | Export format (default: txt) |
| `-k` | API key (e.g., `shodan@<key>`) |

!!! tip
    Use `--full` for an initial broad sweep, then follow up with targeted manual techniques based on what the automated scan surfaces.
