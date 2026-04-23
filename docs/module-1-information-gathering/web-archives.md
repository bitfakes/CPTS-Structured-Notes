---
title: Web Archives (Wayback Machine)
tags: [osint, passive-recon, web-archives]
---

# Web Archives (Wayback Machine)
> The Wayback Machine archives snapshots of websites over time, letting you "go back in time" to see how a site looked — and what it exposed — in the past.

**URL:** [https://web.archive.org/](https://web.archive.org/)

## How It Works

1. **Crawling** — Automated bots browse the web, downloading copies of pages they encounter.
2. **Archiving** — Pages are stored with a timestamp. Frequency varies: popular sites may be archived daily; others only a few times per year.
3. **Accessing** — Enter a URL + select a date to view the archived snapshot.

!!! tip
    The Wayback Machine does not archive everything — it prioritises culturally or historically significant sites. Website owners can request exclusion, though it's not guaranteed.

---

## Recon Value

| Use Case | What You're Looking For |
|----------|------------------------|
| **Hidden Assets** | Old pages, directories, files, or subdomains removed from the current site |
| **Tracking Changes** | Compare snapshots over time — structural changes, removed features, old credentials |
| **OSINT** | Past marketing, employee names, technology choices, internal communications |
| **Stealthy Recon** | Accessing archives is fully passive — no interaction with the live target |

---

## Practical Tips

- Look for old login pages, admin panels, or API endpoints no longer linked but potentially still live.
- Old source code in archived JS files may contain hardcoded credentials or internal endpoints.
- Config changes between snapshots can reveal what was intentionally hidden and when.
- Combine with crt.sh — if an old subdomain appears in CT logs, check the Wayback Machine for its historical content.
