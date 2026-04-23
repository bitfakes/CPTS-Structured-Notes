---
title: DNS Zone Transfers
tags: [dns, subdomains, active-recon, zone-transfer]
---

# DNS Zone Transfers
> A mechanism for replicating DNS records between name servers. If misconfigured, it leaks the entire zone — all subdomains, IPs, and DNS records.

## What is a Zone Transfer?

A DNS zone transfer is a wholesale copy of all DNS records within a zone from one name server to another. Legitimate use: keeping secondary DNS servers in sync. Security risk: if not restricted, anyone can request it.

### Zone Transfer Process (AXFR)

1. **AXFR Request** — Secondary DNS server sends a full zone transfer request to the primary.
2. **SOA Transfer** — Primary responds with its Start of Authority record (contains serial number for version checking).
3. **Records Transmission** — Primary sends all DNS records (A, AAAA, MX, CNAME, NS, etc.) one by one.
4. **Completion Signal** — Primary signals end of transfer.
5. **ACK** — Secondary confirms receipt.

---

## What an Attacker Gets from a Successful Zone Transfer

- **All subdomains** — Including dev, staging, and admin subdomains not linked publicly.
- **IP addresses** — Direct targets for further scanning or attacks.
- **Name server records** — Reveals hosting provider and potential misconfigurations.

!!! warning
    Most modern DNS servers only allow zone transfers to trusted secondaries. However, misconfigurations still occur — always attempt this during recon (with authorisation).

---

## Key Commands

```bash
# Zone transfer attempt
dig axfr @<nameserver> <domain.com>

# Test against a public vulnerable server
dig axfr @nsztm1.digi.ninja zonetransfer.me
```

!!! tip
    `zonetransfer.me` is a deliberately vulnerable domain for practicing zone transfer attacks. Use it to see what a successful response looks like.
