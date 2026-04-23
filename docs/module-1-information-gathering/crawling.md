---
title: Crawling
tags: [crawling, active-recon, tools]
---

# Crawling
> Automated process of systematically browsing a website by following links — like a spider navigating its web. Differs from fuzzing: crawling follows existing links, fuzzing guesses potential ones.

## How Web Crawlers Work

Starting from a seed URL, a crawler:
1. Fetches the page
2. Parses content and extracts all links
3. Adds new links to a queue
4. Repeats for each link in the queue

---

## Crawling Strategies

### Breadth-First Crawling
Explores all links at the current depth before going deeper. Good for a broad overview of site structure.

```
Seed URL → Page 1 → (Page 2, Page 3) → (Page 4, Page 5, Page 6, Page 7)
```

### Depth-First Crawling
Follows one path as deep as possible before backtracking. Useful for reaching deeply nested content.

```
Seed URL → Page 1 → Page 2 → Page 3 → Page 4
```

---

## What Crawlers Extract

| Data Type | Recon Value |
|-----------|------------|
| **Internal Links** | Map site structure, find unlisted pages |
| **External Links** | Identify third-party dependencies |
| **Comments** | May reveal internal details, version info, or dev notes |
| **Metadata** | Page titles, author names, keywords, dates |
| **Backup/Config Files** | `.bak`, `.old`, `web.config`, `settings.php`, `error_log` — may contain credentials |
| **Emails** | Contact addresses for social engineering |
| **JavaScript Files** | Hidden endpoints, API keys, hardcoded credentials |

!!! tip
    Don't analyse data points in isolation. A single comment mentioning a "file server" combined with a discovered `/files/` directory with open browsing = critical finding. Context multiplies value.

---

## Key Commands

```bash
# Install Scrapy
sudo pip3 install scrapy

# Download ReconSpider (custom Scrapy spider for recon)
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
unzip ReconSpider.zip

# Run against target
python3 ReconSpider.py http://<target.com>
```

### ReconSpider Output (`results.json`)

| Key | Data |
|-----|------|
| `emails` | Email addresses found on the domain |
| `links` | All URLs found |
| `external_files` | PDFs and other external files |
| `js_files` | JavaScript files |
| `form_fields` | HTML forms |
| `images` | Image URLs |
| `comments` | HTML comments in source code |
