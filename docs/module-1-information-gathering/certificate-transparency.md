---
title: Certificate Transparency Logs
tags: [ssl, subdomains, passive-recon, ct-logs]
---

# Certificate Transparency Logs
> Public, append-only ledgers recording every SSL/TLS certificate issued. Certificates list subdomains in their SAN (Subject Alternative Name) field — making CT logs a reliable passive subdomain discovery method.

## Why CT Logs Are Useful for Recon

- **No guessing** — Unlike brute force, CT logs give a definitive record of certificates issued for a domain and its subdomains.
- **Historical coverage** — Includes old and expired certificates, which may point to outdated, vulnerable systems.
- **Passive** — No interaction with the target required.

---

## How CT Logs Work

1. **Certificate Issuance** — CA verifies domain ownership and issues a pre-certificate.
2. **Log Submission** — CA submits the pre-certificate to multiple CT logs (append-only, cannot be modified/deleted).
3. **SCT Issued** — Each log returns a Signed Certificate Timestamp (cryptographic proof of submission time).
4. **Browser Verification** — Browser checks the certificate's SCTs against public CT logs on connection.
5. **Monitoring** — Security researchers continuously monitor logs for rogue or suspicious certificates.

CT logs use a **Merkle tree** structure — any tampering changes the root hash, making modification detectable.

---

## Search Tools

| Tool | Features | Notes |
|------|----------|-------|
| [crt.sh](https://crt.sh/) | Web UI + API, search by domain, shows SAN entries | Free, no registration |
| [Censys](https://search.censys.io/) | Advanced filtering, API access, extensive data | Registration required (free tier) |

---

## Key Commands

```bash
# Query crt.sh API for all subdomains of a domain
curl -s "https://crt.sh/?q=<domain.com>&output=json" \
  | jq -r '.[] | select(.name_value | contains("<domain>")) | .name_value' \
  | sort -u
```

### Example Output
```
*.payatu.com
payatu.com
studio.testlms.payatu.com
testlms.payatu.com
trail.payatu.com
www.payatu.com
```

!!! tip
    Wildcard entries like `*.payatu.com` mean any subdomain may be valid — worth fuzzing further with brute force tools.
