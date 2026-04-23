---
title: Search Engine Discovery & Google Dorking
tags: [osint, passive-recon, google-dorking]
---

# Search Engine Discovery & Google Dorking
> Using search engines as recon tools — legally and passively surfacing sensitive data, exposed files, login pages, and misconfigurations that are publicly indexed.

## Search Operators Reference

| Operator | Description | Example |
|----------|-------------|---------|
| `site:` | Limit results to a specific domain | `site:example.com` |
| `inurl:` | Term in the URL | `inurl:login` |
| `filetype:` | Specific file type | `filetype:pdf` |
| `intitle:` | Term in the page title | `intitle:"confidential report"` |
| `intext:` / `inbody:` | Term in the body text | `intext:"password reset"` |
| `cache:` | Cached version of a page | `cache:example.com` |
| `link:` | Pages linking to a URL | `link:example.com` |
| `related:` | Websites similar to a URL | `related:example.com` |
| `info:` | Summary info about a page | `info:example.com` |
| `define:` | Definition of a term | `define:phishing` |
| `numrange:` | Numbers within a range | `site:example.com numrange:1000-2000` |
| `allintext:` | All terms in body text | `allintext:admin password reset` |
| `allinurl:` | All terms in URL | `allinurl:admin panel` |
| `allintitle:` | All terms in title | `allintitle:confidential report 2023` |
| `AND` | Require all terms | `site:example.com AND (inurl:admin OR inurl:login)` |
| `OR` | Any of the terms | `"linux" OR "ubuntu" OR "debian"` |
| `NOT` / `-` | Exclude a term | `site:bank.com NOT inurl:login` |
| `*` | Wildcard | `site:example.com filetype:pdf user* manual` |
| `..` | Number range | `site:ecommerce.com "price" 100..500` |
| `" "` | Exact phrase | `"information security policy"` |

---

## Google Dorking — Common Attack Patterns

### Find Login Pages
```
site:example.com inurl:login
site:example.com (inurl:login OR inurl:admin)
```

### Exposed Files
```
site:example.com filetype:pdf
site:example.com (filetype:xls OR filetype:docx)
```

### Configuration Files
```
site:example.com inurl:config.php
site:example.com (ext:conf OR ext:cnf)
```

### Database Backups
```
site:example.com inurl:backup
site:example.com filetype:sql
```

### Exposed Credentials / Sensitive Data
```
site:example.com filetype:env
site:example.com inurl:.git
site:example.com "index of" "password"
```

!!! tip
    **Reference:** [Google Hacking Database (GHDB)](https://www.exploit-db.com/google-hacking-database) — community-maintained repository of proven Google dorks organised by category.

!!! warning
    Search engine discovery relies on what's already indexed. Not all data is indexed, and some is deliberately excluded via `robots.txt` or `noindex` meta tags.
