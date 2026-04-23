---
title: Web Crawlers (Tools)
tags: [crawling, active-recon, tools]
---

# Web Crawlers (Tools)
> Crawling tools automate the process of mapping a web application's structure and extracting data.

## Popular Web Crawlers

| Tool | Type | Best For |
|------|------|---------|
| **Burp Suite Spider** | GUI (part of Burp Suite) | Web app testing, finding hidden content, integrated with other Burp tools |
| **OWASP ZAP Spider** | GUI / automated | Open-source, integrated security scanning |
| **Scrapy** | Python framework | Custom crawlers, structured data extraction, scalable recon tasks |
| **Apache Nutch** | Java, scalable | Large-scale crawls across entire domains; requires technical setup |

!!! warning
    Always get permission before crawling a site. Be mindful of server load — excessive requests can trigger rate limits or take down a server.

---

## Scrapy + ReconSpider

`ReconSpider` is a custom Scrapy spider built for web reconnaissance — extracts emails, links, files, JS files, images, and HTML comments.

### Setup

```bash
# Install Scrapy
pip3 install scrapy

# Download ReconSpider
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
unzip ReconSpider.zip

# Run against target
python3 ReconSpider.py http://<target.com>
```

### Output — `results.json`

```json
{
    "emails": ["user@example.com"],
    "links": ["https://example.com/about"],
    "external_files": ["https://example.com/docs/manual.pdf"],
    "js_files": ["https://example.com/app.min.js"],
    "form_fields": [],
    "images": ["https://example.com/logo.png"],
    "comments": ["<!-- TODO: remove debug endpoint -->"]
}
```

!!! tip
    HTML comments (`comments` key) are frequently overlooked by developers — they often contain TODO notes, debug endpoints, or internal path references that are valuable during recon.
