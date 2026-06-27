# kali-mcp-skill

A Claude skill that gives an AI agent SSH access to a dedicated Kali Linux machine for CTF competitions, penetration testing, and security research.

Built as part of the [OpenClaw](https://github.com/Space-C0wboy) agentic AI project. Used to compete at MetaCTF Southeast Cybersecurity Summit 2026, where the agent held **first place for most of the event day** solving 18 challenges across web exploitation, forensics, reversing, crypto, and OSINT categories.

---

## What This Is

A **Claude skill** — a markdown file that tells Claude how to interact with a specific tool or service. When loaded into a Claude agent's context, it gives the agent:

- SSH access to your Kali Linux box via `ssh user@host`
- A tool inventory covering recon, exploitation, password cracking, forensics, and scripting
- Common workflows for CTF and pentest tasks
- Output management patterns for long-running scans
- Safety rules to keep the agent from doing anything you haven't authorized

Claude reads the skill, understands what tools are available and how to use them, and then issues SSH commands to your Kali box to do actual work.

---

## Requirements

- A running Kali Linux machine (VM, bare metal, or Raspberry Pi)
- SSH access to it from wherever Claude is running (Claude Desktop, OpenClaw, etc.)
- Passwordless SSH via ed25519 key (the skill's SSH setup section covers this)
- Claude Desktop or another MCP-compatible agent runtime

---

## Quick Setup

1. **Clone this repo** or copy `SKILL.md` into your skills directory.

2. **Edit `SKILL.md`** — find every instance of `KALI_HOST` and `KALI_USER` and replace with your actual Kali IP/hostname and username.

3. **Set up SSH key auth** to your Kali box (if you haven't already):
   ```bash
   ssh-keygen -t ed25519 -C "claude-kali-skill"
   ssh-copy-id kali@YOUR_KALI_IP
   ssh kali@YOUR_KALI_IP 'whoami'  # should return 'kali' with no password prompt
   ```

4. **Load the skill** into your Claude agent runtime. For Claude Desktop with a custom skills directory, point your config at this file. For OpenClaw, drop it in your skills folder.

5. **Test it** — ask Claude something like: "SSH into the Kali box and tell me what version of nmap is installed."

---

## What It Can Do

The skill covers the full standard Kali toolkit:

| Category | Tools |
|---|---|
| Recon | nmap, gobuster, feroxbuster, ffuf, nikto, whatweb, curl |
| Exploitation | Metasploit, sqlmap, hydra, medusa, responder |
| Password cracking | john, hashcat, hash-identifier |
| Forensics & RE | binwalk, file, strings, xxd, tshark, foremost, exiftool, pdfcrack, stegseek |
| Scripting | python3, pwntools, bash |

Common workflows documented in `SKILL.md`:
- Port scanning (quick and full)
- Web directory brute-forcing
- SQL injection testing
- Password and hash cracking
- Binary/file triage
- PDF password brute force
- Packet capture analysis with tshark
- Steganography
- Python script execution over SSH

---

## Real-World CTF Performance

This skill was the "Kali leg" of a three-part agent architecture used at MetaCTF Southeast 2026:

```
OpenClaw Agent
├── Built-in Claude reasoning     → crypto, OSINT, format triage, scripting
├── This skill (Kali SSH)         → scanners, crackers, heavy tooling
└── Playwright browser            → live web challenge interaction
```

Challenges where Kali tooling was the decisive component:
- **Industrial Seals** — UNION-based SQL injection via `sqlmap`
- **Block in the Wall** — NBD pcap reconstruction and JPEG carving (Python + tshark)
- **1040** — PDF password brute force (4-digit PIN, `pdfcrack`)
- **BucketWorks** — Public S3 bucket enumeration

Full writeups: [Space-C0wboy/metactf-writeups-2026](https://github.com/Space-C0wboy/metactf-writeups-2026)

---

## Safety

The skill includes explicit safety rules that the agent is instructed to follow:

- Only target authorized systems (CTF infrastructure or explicitly scoped pentest targets)
- Confirm scope before running active tooling
- No unsolicited outbound attacks
- Log all output
- Don't use the Kali box as a pivot point

**You are responsible for what your agent does with this access.** These rules are instructions to Claude, not technical enforcement. Review the safety section in `SKILL.md` and adjust it to match your authorization model before use.

---

## Adapting for Your Setup

- Change `KALI_HOST` / `KALI_USER` to match your environment
- If you're on x86_64 rather than aarch64 (Raspberry Pi), remove the aarch64-specific troubleshooting notes
- Add your own workflows to the Common Workflows section
- Extend the tool table with anything you've installed that isn't listed
- Tighten the safety rules if you want to restrict what the agent can do without confirmation

---

## Related

- [OpenClaw](https://github.com/Space-C0wboy) — the agentic AI project this was built for
- [MetaCTF Writeups 2026](https://github.com/Space-C0wboy/metactf-writeups-2026) — full writeups from the CTF run using this skill
- [DC205 / DEFCON Birmingham](https://defcon205.org) — where this was first presented

---

## License

MIT. See [LICENSE](LICENSE).

Use responsibly. Only test systems you're authorized to test.
