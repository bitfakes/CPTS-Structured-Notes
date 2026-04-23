---
title: MSF Database & Workspaces
tags: [metasploit, database, postgresql, nmap, workspace]
---

# MSF Database & Workspaces
> Metasploit has built-in PostgreSQL support — store scan results, hosts, services, credentials, and loot across sessions. Workspaces keep engagements separated.

## Setup

```bash
# Install PostgreSQL
sudo apt install postgresql postgresql-contrib

# Check status
sudo service postgresql status

# Start PostgreSQL
sudo systemctl start postgresql

# Initialize MSF database
sudo msfdb init
sudo msfdb status

# Launch MSFConsole with DB
sudo msfdb run

# Or connect from within msfconsole
msf6 > db_status
# [*] Connected to msf. Connection type: postgresql.
```

### Reinitialise (if broken)
```bash
msfdb reinit
cp /usr/share/metasploit-framework/config/database.yml ~/.msf4/
sudo service postgresql restart
msfconsole -q
msf6 > db_status
```

---

## Workspaces

Separate different engagements or targets into isolated workspaces.

```bash
workspace           # list all workspaces
workspace -a Target_1   # create new workspace
workspace Target_1      # switch to workspace
```

---

## Storing & Querying Data

```bash
# Run Nmap through MSF (auto-stores results in DB)
msf6 > db_nmap -sV -sS <IP>

# Import external scan results (Nmap XML, Nexpose, etc.)
msf6 > db_import <scan.xml>

# Query stored data
msf6 > hosts       # discovered hosts
msf6 > services    # discovered services
msf6 > creds       # stored credentials
msf6 > loot        # stored loot

# Export data
msf6 > db_export -h
```

!!! tip
    Always use `db_nmap` instead of running Nmap separately — results go straight into the database and become queryable with `hosts` and `services`.
