---
title: Robots.txt
tags: [crawling, passive-recon, osint]
---

# Robots.txt
> A plain text file at the root of a website that tells web crawlers which areas to access or avoid. From a recon perspective, the "do not enter" signs often point directly to interesting content.

## Location

```
https://www.example.com/robots.txt
```

## Structure

Each record targets a user-agent with a set of directives:

```
User-agent: *
Disallow: /admin/
Disallow: /private/
Allow: /public/

User-agent: Googlebot
Crawl-delay: 10

Sitemap: https://www.example.com/sitemap.xml
```

## Directives

| Directive | Description | Example |
|-----------|-------------|---------|
| `Disallow` | Paths the bot should NOT crawl | `Disallow: /admin/` |
| `Allow` | Explicitly permitted paths (overrides Disallow) | `Allow: /public/` |
| `Crawl-delay` | Seconds between requests (prevents overload) | `Crawl-delay: 10` |
| `Sitemap` | URL of the XML sitemap | `Sitemap: https://example.com/sitemap.xml` |

---

## Recon Value

| What You Find | What It Implies |
|---------------|----------------|
| `Disallow: /admin/` | Admin panel exists at `/admin/` |
| `Disallow: /private/` | Private content worth investigating |
| `Disallow: /backup/` | Backup files may be present |
| Sitemap URL | Full site structure — all indexed pages |
| Honeypot paths | Target is security-aware; tread carefully |

!!! warning
    `robots.txt` is not a security control — it's an etiquette guide. A rogue bot (or you during a pentest) can ignore it entirely. Its value is as an **intelligence source**, not a barrier.

!!! tip
    The sitemap URL in robots.txt gives you a complete structured map of the site's public content without any crawling needed.
