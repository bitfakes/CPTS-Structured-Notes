---
title: Password Cracking (JtR & Hashcat)
tags: [password-attacks, john, hashcat, wordlists, brute-force, rules, cewl]
---

# Password Cracking

> John the Ripper and Hashcat are the two primary offline cracking tools. JtR handles many formats automatically; Hashcat is faster on GPU and supports hundreds of hash modes.

## How Passwords Are Stored

```bash
# MD5 hash (no salt)
echo -n Soccer06! | md5sum
# 40291c1d19ee11a7df8495c4cccefdfa

# Salted hash (prevents rainbow table attacks)
echo -n Th1sIsTh3S@lt_Soccer06! | md5sum
# 90a10ba83c04e7996bc53373170b5474
```

- **Rainbow tables** — pre-compiled hash→password maps; defeated by salting.
- **Dictionary attack** — test passwords from a wordlist.
- **Brute-force attack** — try every possible combination; 100% effective given enough time.

---

## John the Ripper

### Identify Hash Format

```bash
hashid -j 193069ceb0461e1d40d216e32c79c704
locate '*2john*'   # see all format converters bundled with John
```

### Cracking Modes

```bash
# Wordlist mode (most common)
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Wordlist + rules (mutations like leetspeak, append numbers)
john --wordlist=rockyou.txt --rules hash.txt

# Single crack mode (derives candidates from username / GECOS)
john --single passwd

# Incremental mode (statistical brute-force via Markov chains)
john --incremental hash.txt

# Show cracked passwords
john hash.txt --show

# View incremental mode config
grep '# Incremental modes' -A 100 /etc/john/john.conf
```

---

## Hashcat

### Identify Hash Type

```bash
hashid -m '$1$FNr44XZC$wQxY6HHLrgrGX0e1195k.1'   # returns hashcat mode
hashcat -hh                                          # list all modes
# Reference: https://hashcat.net/wiki/doku.php?id=example_hashes
```

### Common Hash Modes

| Mode | Hash Type |
|------|-----------|
| `0` | MD5 |
| `100` | SHA1 |
| `1400` | SHA2-256 |
| `1000` | NTLM |
| `1800` | sha512crypt (Linux shadow) |
| `2100` | DCC2 (domain cached credentials) |
| `22100` | BitLocker |
| `13100` | Kerberoast (TGS-REP) |
| `18200` | AS-REP Roast |

### Attack Modes

```bash
# -a 0: Dictionary attack
hashcat -a 0 -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt

# -a 0 with rules
hashcat -a 0 -m 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule
ls /usr/share/hashcat/rules    # view available rulesets

# -a 3: Mask (brute-force) attack
hashcat -h | grep -A 12 "Built-in Charsets"
# ?u=uppercase, ?l=lowercase, ?d=digit, ?s=special
hashcat -a 3 -m 0 hash.txt '?u?l?l?l?l?d?s'
```

!!! tip
    Use `-a 0` with `rockyou.txt` first. If that fails, try adding `--rules best64.rule`. Fall back to mask mode only when you have good intel on the password pattern (e.g., from a policy document).

---

## Custom Wordlists and Rules

```bash
# Generate a wordlist from a target website with CeWL
cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
wc -l inlane.wordlist
```

!!! warning
    Encoders (Hashcat/JtR rules) are not AV evasion — they generate password candidates from patterns. Keep them separate from evasion techniques.
