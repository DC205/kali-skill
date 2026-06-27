---
name: kali
description: "Use when given a security or CTF task requiring real tooling: port scanning, web exploitation, SQL injection, password cracking, forensic file analysis, packet capture parsing, binary triage, or steganography. Triggers on: 'scan this target', 'run nmap', 'crack this hash', 'analyze this pcap', 'brute force the login', 'enumerate the service', 'run sqlmap', 'run gobuster', CTF challenge categories requiring tools (web, forensics, reversing, crypto), or any request needing Kali Linux tools executed against an authorized target. Connects via SSH to a dedicated Kali Linux box and runs commands directly. Requires explicit authorization before targeting any host."
license: MIT
---

# Kali Linux Remote Access Skill

Give Claude SSH access to a dedicated Kali Linux machine for CTF competitions, penetration testing, and security research. Claude can enumerate targets, run tooling, crack hashes, parse artifacts, and manage output files — all via SSH commands issued from your agent's shell.

This skill was developed as part of the [OpenClaw](https://github.com/Space-C0wboy) agentic AI project and used to compete in MetaCTF Southeast 2026, where the agent held first place for most of the event day.

---

## Configuration

**Before using this skill, update the connection details below to match your environment.**

```
KALI_HOST = <your-kali-ip-or-hostname>
KALI_USER = kali                          # default Kali user; change if different
KEY_PATH  = ~/.ssh/id_ed25519             # path to your SSH private key
```

All example commands in this skill use the pattern:
```bash
ssh KALI_USER@KALI_HOST '<command>'
```

Substitute your actual host and user throughout, or set shell aliases:
```bash
alias kssh='ssh kali@YOUR_KALI_IP'
alias kscp='scp kali@YOUR_KALI_IP'
```

---

## SSH Setup

If you haven't set up passwordless SSH to your Kali box yet:

```bash
# Generate a key (if you don't have one)
ssh-keygen -t ed25519 -C "claude-kali-skill" -f ~/.ssh/id_ed25519_kali

# Copy the public key to Kali
ssh-copy-id -i ~/.ssh/id_ed25519_kali.pub kali@YOUR_KALI_IP

# Test passwordless login
ssh -i ~/.ssh/id_ed25519_kali kali@YOUR_KALI_IP 'whoami'
```

Confirm SSH is running on Kali:
```bash
sudo systemctl enable ssh && sudo systemctl start ssh
```

---

## Quick Reference

Run a command:
```bash
ssh kali@KALI_HOST '<command>'
```

Run with TTY (for interactive/ncurses tools):
```bash
ssh -t kali@KALI_HOST '<command>'
```

Background a long-running task and poll later:
```bash
# Start
ssh kali@KALI_HOST 'nohup <command> > /tmp/output.log 2>&1 &'

# Check progress
ssh kali@KALI_HOST 'cat /tmp/output.log'
ssh kali@KALI_HOST 'tail -f /tmp/output.log'  # live tail (with TTY)
```

Transfer files:
```bash
# To Kali
scp /local/path kali@KALI_HOST:/remote/path

# From Kali
scp kali@KALI_HOST:/remote/path /local/path

# Whole directory
scp -r kali@KALI_HOST:/tmp/results/ ./results/
```

---

## Available Tools

### Reconnaissance

| Tool | Path | Purpose |
|------|------|---------|
| nmap | /usr/bin/nmap | Network discovery, port scanning, service detection |
| gobuster | /usr/bin/gobuster | Directory, DNS, and vhost brute-forcing |
| feroxbuster | /usr/bin/feroxbuster | Fast recursive content discovery (often preferred over gobuster) |
| ffuf | /usr/bin/ffuf | Web fuzzer — parameters, headers, vhosts |
| enum4linux | /usr/bin/enum4linux | SMB/NetBIOS enumeration |
| nikto | /usr/bin/nikto | Web server vulnerability scanner |
| whatweb | /usr/bin/whatweb | Web technology fingerprinting |
| curl | /usr/bin/curl | HTTP inspection and manual request crafting |

### Exploitation

| Tool | Path | Purpose |
|------|------|---------|
| msfconsole | /usr/bin/msfconsole | Metasploit Framework |
| sqlmap | /usr/bin/sqlmap | SQL injection automation |
| hydra | /usr/bin/hydra | Online password brute-forcing |
| medusa | /usr/bin/medusa | Parallel network login brute-forcer |
| responder | /usr/sbin/responder | LLMNR/NBT-NS/MDNS poisoner |

### Password Cracking

| Tool | Path | Purpose |
|------|------|---------|
| john | /usr/sbin/john | John the Ripper — broad hash support |
| hashcat | /usr/bin/hashcat | GPU-accelerated hash cracking |
| hash-identifier | /usr/bin/hash-identifier | Identify unknown hash types |

### Forensics & Reverse Engineering

| Tool | Path | Purpose |
|------|------|---------|
| binwalk | /usr/bin/binwalk | Firmware/binary extraction and analysis |
| file | /usr/bin/file | File type identification via magic bytes |
| strings | /usr/bin/strings | Extract printable strings from binaries |
| xxd | /usr/bin/xxd | Hex dump for binary inspection |
| tshark | /usr/bin/tshark | CLI packet capture and analysis (Wireshark backend) |
| foremost | /usr/sbin/foremost | File carving from raw data/images |
| exiftool | /usr/bin/exiftool | Metadata extraction |
| pdfcrack | /usr/bin/pdfcrack | PDF password brute-forcing |
| stegseek | /usr/bin/stegseek | Fast steganography tool (rockyou-backed) |

### Scripting & Automation

| Tool | Path | Purpose |
|------|------|---------|
| python3 | /usr/bin/python3 | Quick parsers, decoders, and exploit scripts |
| pip3 | /usr/bin/pip3 | Install Python packages (pwntools, pycryptodome, etc.) |
| pwntools | (pip) | CTF scripting framework — `from pwn import *` |
| ruby | /usr/bin/ruby | For tools and scripts that require it |

### Not Installed by Default (install as needed)

```bash
sudo apt install -y bloodhound netexec seclists impacket-scripts
pip3 install pwntools pycryptodome requests
```

> **Note:** Burp Suite is GUI-only and not practical for headless/agent use. Use `curl`, `ffuf`, or `sqlmap` for web work instead.

---

## Wordlists

Standard Kali wordlists live at `/usr/share/wordlists/`:

```
/usr/share/wordlists/rockyou.txt          — classic password list
/usr/share/wordlists/rockyou.txt.gz       — compressed (gunzip before use)
/usr/share/wordlists/dirb/common.txt      — common web directories
/usr/share/wordlists/dirb/big.txt         — larger directory list
/usr/share/wordlists/dirbuster/           — even larger lists
/usr/share/wordlists/wfuzz/               — fuzzing wordlists
/usr/share/seclists/                      — if seclists is installed
```

Decompress rockyou if needed:
```bash
ssh kali@KALI_HOST 'ls /usr/share/wordlists/rockyou.txt 2>/dev/null || gunzip /usr/share/wordlists/rockyou.txt.gz'
```

---

## Common Workflows

### Port Scan

```bash
# Quick scan — top ports
ssh kali@KALI_HOST 'nmap -sC -sV -oN /tmp/scan.txt TARGET'

# Full port scan (slow — use background task)
ssh kali@KALI_HOST 'nohup nmap -sC -sV -p- TARGET -oA /tmp/fullscan > /dev/null 2>&1 &'
ssh kali@KALI_HOST 'cat /tmp/fullscan.nmap'
```

### Web Directory Brute Force

```bash
# gobuster
ssh kali@KALI_HOST 'gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt -o /tmp/gobuster.txt'

# feroxbuster (recursive, often better)
ssh kali@KALI_HOST 'feroxbuster -u http://TARGET -w /usr/share/wordlists/dirb/common.txt -o /tmp/ferox.txt'
```

### SQL Injection

```bash
# Basic test
ssh kali@KALI_HOST 'sqlmap -u "http://TARGET/page?id=1" --batch --dbs'

# Targeted extraction
ssh kali@KALI_HOST 'sqlmap -u "http://TARGET/page?id=1" --batch -D dbname -T tablename --dump'
```

### Password Cracking

```bash
# John
ssh kali@KALI_HOST 'john --wordlist=/usr/share/wordlists/rockyou.txt /tmp/hashes.txt'
ssh kali@KALI_HOST 'john --show /tmp/hashes.txt'

# Hashcat (needs GPU support or falls back to CPU)
ssh kali@KALI_HOST 'hashcat -m 0 /tmp/hashes.txt /usr/share/wordlists/rockyou.txt'

# Identify hash type first
ssh kali@KALI_HOST 'hash-identifier <<< "HASH_STRING"'
```

### Hydra Login Brute Force

```bash
# SSH
ssh kali@KALI_HOST 'hydra -l USERNAME -P /usr/share/wordlists/rockyou.txt TARGET ssh'

# HTTP form
ssh kali@KALI_HOST 'hydra -l admin -P /usr/share/wordlists/rockyou.txt TARGET http-post-form "/login:user=^USER^&pass=^PASS^:F=incorrect"'
```

### Binary / File Triage

```bash
# Fast first-pass (always start here)
ssh kali@KALI_HOST 'file /tmp/artifact && strings /tmp/artifact | head -100'

# Extract embedded files
ssh kali@KALI_HOST 'binwalk -e /tmp/artifact -C /tmp/extracted/'

# Hex inspection
ssh kali@KALI_HOST 'xxd /tmp/artifact | head -40'
```

### PDF Password Brute Force

```bash
# Upload the PDF first
scp ./target.pdf kali@KALI_HOST:/tmp/target.pdf

# Brute force (numeric PIN example — 4-digit)
ssh kali@KALI_HOST 'for i in $(seq -w 0000 9999); do pdfcrack -q -n "$i" /tmp/target.pdf 2>/dev/null | grep -q "found" && echo "PIN: $i" && break; done'

# Or with pdfcrack wordlist mode
ssh kali@KALI_HOST 'pdfcrack -w /usr/share/wordlists/rockyou.txt /tmp/target.pdf'
```

### Packet Capture Analysis (tshark)

```bash
# Upload pcap
scp ./capture.pcap kali@KALI_HOST:/tmp/capture.pcap

# List protocols
ssh kali@KALI_HOST 'tshark -r /tmp/capture.pcap -z io,phs -q'

# Follow TCP stream (stream index 0)
ssh kali@KALI_HOST 'tshark -r /tmp/capture.pcap -q -z follow,tcp,ascii,0'

# Extract HTTP objects
ssh kali@KALI_HOST 'tshark -r /tmp/capture.pcap --export-objects http,/tmp/http_export/'

# Filter by protocol
ssh kali@KALI_HOST 'tshark -r /tmp/capture.pcap -Y "http" -T fields -e http.request.uri'
```

### Quick Python Script on Kali

```bash
# Copy a local script and run it
scp ./solve.py kali@KALI_HOST:/tmp/solve.py
ssh kali@KALI_HOST 'python3 /tmp/solve.py'
```

### Steganography

```bash
# stegseek (rockyou-backed, very fast)
scp ./image.jpg kali@KALI_HOST:/tmp/image.jpg
ssh kali@KALI_HOST 'stegseek /tmp/image.jpg /usr/share/wordlists/rockyou.txt'

# steghide (if password is known)
ssh kali@KALI_HOST 'steghide extract -sf /tmp/image.jpg -p PASSWORD'
```

### Metasploit (interactive)

```bash
# Interactive session — requires TTY
ssh -t kali@KALI_HOST 'msfconsole'

# Non-interactive one-liner
ssh kali@KALI_HOST 'msfconsole -q -x "use exploit/multi/handler; set PAYLOAD linux/x64/shell_reverse_tcp; set LHOST 0.0.0.0; set LPORT 4444; run"'
```

---

## Output Management

Always save output to files on Kali for long-running tasks. Don't rely on keeping the SSH session open.

```bash
# Start scan with output to file
ssh kali@KALI_HOST 'nohup nmap -sC -sV TARGET -oA /tmp/scan > /dev/null 2>&1 &'

# Check if it's still running
ssh kali@KALI_HOST 'pgrep -la nmap'

# Retrieve results when done
scp kali@KALI_HOST:/tmp/scan.* ./results/

# Clean up after retrieval
ssh kali@KALI_HOST 'rm -f /tmp/scan.*'
```

Keep working files organized:
```bash
ssh kali@KALI_HOST 'mkdir -p ~/ctf/$(date +%Y%m%d)/'
```

---

## Safety Rules

**This skill gives Claude real access to real tooling. These rules must be enforced by the system prompt and skill context.**

1. **Only target authorized systems.** CTF infrastructure and explicitly scoped pentest targets only. Never scan or probe systems you don't have permission to test.
2. **Confirm scope before each scan or exploitation attempt.** When in doubt, ask the operator before running active tooling.
3. **No unsolicited outbound attacks.** Do not initiate connections to external hosts unless the current task explicitly requires it and the operator has authorized it.
4. **Log everything.** Save all command output. Good logs are essential for writeups, debrief, and defending your actions.
5. **Store findings locally.** Use `~/ctf/` or `/tmp/` on the Kali box and retrieve them via `scp`. Don't leave sensitive output sitting on remote hosts longer than needed.
6. **Treat the Kali box as a trusted tool, not a launch pad.** It's for your authorized work, not a pivot point to other networks.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `Connection refused` | Check SSH is running: `sudo systemctl status ssh` |
| `Permission denied (publickey)` | Confirm key is deployed: `ssh-copy-id -i ~/.ssh/id_ed25519.pub kali@HOST` |
| Tool not found | Install it: `ssh kali@HOST 'sudo apt install -y TOOL'` |
| `Permission denied` running tool | Prefix with `sudo`: `ssh kali@HOST 'sudo TOOL ...'` |
| Slow scans | On aarch64 (Raspberry Pi), reduce parallelism: `nmap -T2`, `hashcat --opencl-device-types 1` |
| Hung interactive session | Use `Ctrl+C` locally or kill server-side: `ssh kali@HOST 'pkill TOOL'` |
| TTY errors in scripts | Remove `-t` flag; use non-interactive command forms |
| `rockyou.txt` missing | Decompress: `ssh kali@HOST 'gunzip /usr/share/wordlists/rockyou.txt.gz'` |

---

## Adapting This Skill

To use this skill with your own Kali setup:

1. **Update the connection details** — replace `KALI_HOST` and `KALI_USER` throughout with your actual values.
2. **Set up SSH key auth** — follow the [SSH Setup](#ssh-setup) section above.
3. **Verify your tool paths** — most tools listed here are at their default Kali paths, but confirm with `which TOOL` if any are missing.
4. **Adjust aarch64 notes if you're on x86_64** — hashcat GPU support and scan performance will differ. Remove aarch64-specific guidance from the troubleshooting section if it's not relevant.
5. **Customize the safety rules** — replace references to "the operator" with whatever authorization model your setup uses.
6. **Add your own workflows** — the Common Workflows section is a starting point. Extend it with techniques relevant to your target environment or CTF focus.

This skill is designed to work with Claude Desktop via MCP (Model Context Protocol) but the SSH patterns here are shell-agnostic and work with any agent that can run bash commands.
