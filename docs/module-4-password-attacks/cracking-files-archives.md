---
title: Cracking Files & Archives
tags: [password-attacks, john, hashcat, ssh, zip, pdf, docx, bitlocker, archives]
---

# Cracking Files & Archives

> Use the `*2john` helper scripts to extract crackable hashes from protected files, then feed them to John or Hashcat.

## Finding Protected Files

```bash
# Hunt for common document extensions
for ext in $(echo ".xls .xls* .xltx .od* .doc .doc* .pdf .pot .pot* .pp*"); do
  echo -e "\nFile extension: " $ext
  find / -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core"
done

# See all available *2john converters
locate '*2john*'
```

---

## SSH Keys

```bash
# Find private keys on the filesystem
grep -rnE '^\-{5}BEGIN [A-Z0-9]+ PRIVATE KEY\-{5}$' /* 2>/dev/null

# Check if a key is encrypted (will prompt for passphrase if yes)
ssh-keygen -yf ~/.ssh/id_ed25519

# Crack encrypted SSH key
ssh2john.py SSH.private > ssh.hash
john --wordlist=rockyou.txt ssh.hash
john ssh.hash --show
```

---

## Office Documents

```bash
# Word/DOCX
office2john.py Protected.docx > protected-docx.hash
john --wordlist=rockyou.txt protected-docx.hash
john protected-docx.hash --show

# PDF
pdf2john.py PDF.pdf > pdf.hash
john --wordlist=rockyou.txt pdf.hash
john pdf.hash --show
```

---

## Archives

### ZIP

```bash
zip2john ZIP.zip > zip.hash
cat zip.hash
john --wordlist=rockyou.txt zip.hash
```

### OpenSSL-encrypted GZIP

```bash
file GZIP.gzip
for i in $(cat rockyou.txt); do
  openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null | tar xz
done
```

!!! tip
    The loop decrypts silently and only `tar xz` succeeds when the key is right — watch for files appearing in the current directory.

---

## BitLocker-Encrypted Drives

### Extract and Crack

```bash
bitlocker2john -i Backup.vhd > backup.hashes
grep "bitlocker\$0" backup.hashes > backup.hash
cat backup.hash

hashcat -a 0 -m 22100 '<bitlocker_hash>' /usr/share/wordlists/rockyou.txt
```

### Mount After Cracking (Linux)

```bash
sudo apt-get install dislocker

sudo mkdir -p /media/bitlocker
sudo mkdir -p /media/bitlockermount

# Attach VHD as loop device
sudo losetup -f -P <name>.vhd
lsblk
# or
sudo fdisk -l /dev/loop0

# Decrypt and mount
sudo dislocker /dev/loop0p2 -u<CrackedPassword> -- /media/bitlocker
sudo mount -o loop /media/bitlocker/dislocker-file /media/bitlockermount

# Browse files
cd /media/bitlockermount && ls -la

# Unmount when done
sudo umount /media/bitlockermount
sudo umount /media/bitlocker
```
