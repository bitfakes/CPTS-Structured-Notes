---
title: Well-Known URIs
tags: [passive-recon, http, osint, ssl]
---

# Well-Known URIs
> Standardised directory at `/.well-known/` on any web server (defined in RFC 8615) — a consistent location for metadata, config files, and security info. Often overlooked in recon.

## Common Well-Known URIs

| URI | Description | Status |
|-----|-------------|--------|
| `/.well-known/security.txt` | Contact info for reporting vulnerabilities | Permanent (RFC 9116) |
| `/.well-known/change-password` | URL for directing users to password change | Provisional |
| `/.well-known/openid-configuration` | OpenID Connect provider config (OAuth 2.0) | Permanent |
| `/.well-known/assetlinks.json` | Verify ownership of digital assets (apps) | Permanent |
| `/.well-known/mta-sts.txt` | SMTP MTA Strict Transport Security policy | Permanent |

Full registry: [IANA Well-Known URIs](https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml)

---

## Recon Value: `openid-configuration`

Accessing `https://example.com/.well-known/openid-configuration` returns a JSON document revealing:

```json
{
  "issuer": "https://example.com",
  "authorization_endpoint": "https://example.com/oauth2/authorize",
  "token_endpoint": "https://example.com/oauth2/token",
  "userinfo_endpoint": "https://example.com/oauth2/userinfo",
  "jwks_uri": "https://example.com/oauth2/jwks",
  "response_types_supported": ["code", "token", "id_token"],
  "id_token_signing_alg_values_supported": ["RS256"],
  "scopes_supported": ["openid", "profile", "email"]
}
```

**What this exposes:**

| Field | Recon Value |
|-------|------------|
| `authorization_endpoint` | Auth flow entry point |
| `token_endpoint` | Token issuance URL |
| `userinfo_endpoint` | User data endpoint |
| `jwks_uri` | Cryptographic keys used by the server |
| `scopes_supported` | What data the app accesses |
| `id_token_signing_alg_values_supported` | Signing algorithm (e.g., RS256 vs HS256) |

!!! tip
    Always check `/.well-known/openid-configuration` on any target using OAuth/OIDC — it maps the entire auth infrastructure without any authentication required.
