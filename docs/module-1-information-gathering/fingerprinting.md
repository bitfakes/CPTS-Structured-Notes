---
title: Fingerprinting
tags: [fingerprinting, active-recon, http, tools, waf, cms]
---

# Fingerprinting
> Extracting technical details about the technologies powering a target — web server, OS, CMS, frameworks, WAF — to find exploitable weaknesses specific to those systems.

## Why Fingerprinting Matters

- **Targeted Attacks** — Knowing the stack lets you focus on known CVEs for those specific versions.
- **Identifying Misconfigurations** — Outdated software, default settings, weak headers.
- **Prioritising Targets** — Focus effort on systems more likely to be vulnerable.
- **Building a Profile** — Combined with other recon, reveals the full attack surface.

---

## Fingerprinting Techniques

| Technique | What It Reveals |
|-----------|----------------|
| **Banner Grabbing** | Web server software, version numbers |
| **HTTP Header Analysis** | `Server`, `X-Powered-By`, custom headers revealing tech stack |
| **Probing Specific Responses** | Unique error messages or behaviours characteristic of specific software |
| **Page Content Analysis** | Copyright headers, paths (e.g., `/wp-content/` = WordPress), meta tags |

---

## Tools

| Tool | Description |
|------|-------------|
| `Wappalyzer` | Browser extension — identifies CMS, frameworks, analytics |
| `BuiltWith` | Web tech profiler with detailed stack reports |
| `WhatWeb` | CLI tool with a vast signature database |
| `Nmap` | Network scanner with NSE scripts for service/OS fingerprinting |
| `Netcraft` | Website fingerprinting + hosting/security reports |
| `wafw00f` | WAF detection — identifies type and configuration |
| `Nikto` | Web server scanner with fingerprinting modules |
| [CMSeeK](https://github.com/Tuhinshubhra/CMSeeK) | CMS-specific detection |

---

## Key Commands

### Banner Grabbing
```bash
# Fetch HTTP headers only
curl -I http://<domain.com>
curl -I https://<domain.com>
```

**What to look for:**
- `Server: Apache/2.4.41 (Ubuntu)` — web server and OS
- `X-Powered-By: PHP/8.1.0` — language/framework
- `X-Redirect-By: WordPress` — CMS
- Paths like `wp-json`, `wp-content` in `Link` headers = WordPress

### WAF Detection
```bash
# Install
sudo pip3 install git+https://github.com/EnableSecurity/wafw00f

# Detect WAF
wafw00f <domain.com>
```

### Nikto — Fingerprinting Scan
```bash
# Install
sudo apt update && sudo apt install -y perl
sudo apt install libjson-perl libjson-pp-perl
git clone https://github.com/sullo/nikto
cd nikto/program
chmod +x ./nikto.pl

# Run fingerprinting modules only (-Tuning b = Software Identification)
nikto -h <domain.com> -Tuning b
```

**Nikto `-Tuning b` reveals:**
- Web server version and OS
- CMS presence (WordPress, etc.)
- Missing security headers (`Strict-Transport-Security`, `X-Content-Type-Options`)
- Exposed files (`license.txt`, `readme.html`)
- Login pages (`/wp-login.php`)

---

## Example: Identifying a WordPress Site

```bash
$ curl -I inlanefreight.com
# → Server: Apache/2.4.41 (Ubuntu) + X-Redirect-By: WordPress

$ curl -I https://www.inlanefreight.com
# → Link header contains /wp-json/ (WordPress REST API)
```

!!! warning
    WAFs can interfere with fingerprinting probes and may block requests. Always check for WAF presence with `wafw00f` before running aggressive scans.
