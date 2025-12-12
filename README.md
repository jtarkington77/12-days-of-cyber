![12 Days Of Cyber](Images/12-days-of-cyber.png)

# 🎄 12 Days of Cyber – Saving Christmas at the North Pole ❄️

**12 days of challenges. 1 Christmas crisis. Can you help the elves save the holiday?**

Welcome to the **North Pole Security Incident** - a holiday CTF series running from **December 12th through December 23rd, 2024**, with a special Christmas Eve bonus challenge on **December 24th**.

## 🎅 The Story

It's December at the **North Pole Operations Center (NPOC)**, and everything should be running smoothly. Santa's elves are working overtime preparing for Christmas Eve - the Nice/Naughty List is being finalized, toys are being manufactured, and the sleigh's navigation systems are being tested.

But something's wrong...

**Merry Tinselcode**, Senior Security Elf, is monitoring the Sleigh Status Dashboard when she notices strange encoded messages where there should be system updates. "That's... odd," she mutters, setting down her peppermint cocoa.

What starts as a minor glitch quickly escalates into a full-blown Christmas crisis. Someone is attacking the North Pole's digital infrastructure! The Nice List database, gift tracking systems, reindeer navigation, and workshop operations are all at risk.

The attacker? **Jack Frost and his crew, "The Frost Giants"** - a mischievous group bent on ruining Christmas!

Over 12 days, you'll help the North Pole Security Operations team investigate the breach, learn cybersecurity skills, and race against time to save Christmas:

- **Week 1 (Dec 12-18):** Discovery – Something's attacking our systems!
- **Week 2 (Dec 19-23):** Deep Investigation – Understanding the full scope of the attack
- **Christmas Eve Bonus (Dec 24):** The Final Push – Save Christmas before it's too late!

This isn't just a CTF - it's a whimsical journey through real incident response, wrapped in holiday magic. 🎁

## 🎁 What You'll Experience

Each day unlocks:

- 🎭 **A Story Chapter** - Follow the North Pole Security Team as they investigate
- 📚 **A Cybersecurity Lesson** - Learn real skills (beginner-friendly, assumes zero experience)
- 🔍 **Investigation Artifacts** - Real files to analyze (logs, scripts, network captures, configs)
- 🏴 **A Flag** - Solve the challenge to get the flag in format: `FROST{l33t_sp34k_fl@g}`
- 🎄 **Progressive Difficulty** - Each day builds on previous skills, just like TryHackMe!

The goal: Learn real cybersecurity skills while saving Christmas! One challenge per day leading up to Christmas Eve.

---

## 🎮 How It Works

- Each day lives in its own folder: `day01`, `day02`, ... `day12`, plus `day13` (Christmas Eve Bonus)
- Inside each folder you'll find:
    - `README.md` - Story chapter, challenge description, and hints
    - `lesson_*.md` - Comprehensive beginner-friendly lesson on the day's technique
    - `artifacts/` folder - Investigation files (logs, configs, network captures, etc.)

## 🔐 Gated Progression with Flags

Each day's flag unlocks the next day's investigation files:

- **Day 1 (Dec 12)** artifacts are open - start here!
- **Days 2-12 + Bonus (Dec 13-24)** artifacts are in **password-protected ZIP files**
- The password is always the previous day's flag (including `FROST{}`)

**Example flow:**
1. Solve **Day 1** → Find flag: `FROST{example_flag_here}`
2. Go to `day02/` and extract `day02_artifacts.zip`
3. Use the exact **Day 1 flag** as the password (case sensitive!)
4. Solve Day 2 → Use that flag to unlock Day 3, and so on...
5. Day 12's flag unlocks the **Christmas Eve Bonus** on December 24th!

This creates a progression chain - you must solve each day to continue helping the elves!

---

## 🛡️ Skills You'll Learn

Across the 12 days (+bonus), you'll help the elves by learning:

- 🔤 **Encoding & Decoding** - Uncover hidden messages (Base64, Hex, Binary, Base32, ROT13)
- 🔐 **Password Cracking** - Recover compromised elf accounts (Hashcat/John, custom wordlists)
- 📋 **Log Analysis** - Track down attackers in authentication logs
- 💻 **JavaScript Deobfuscation** - Analyze malicious phishing scripts
- 🔧 **PowerShell Malware** - Deobfuscate malicious PowerShell scripts
- 🌐 **Web Reconnaissance** - Use Gobuster for directory enumeration
- 📡 **Network Forensics** - Analyze DNS tunneling attacks (Wireshark, PCAP analysis)
- 🗄️ **SQL Injection** - Hands-on exploitation of vulnerable web applications
- 🔑 **API Security** - Detect leaked secrets and authorization bypass vulnerabilities
- 💾 **Windows Persistence** - Hunt Registry, Scheduled Tasks, WMI, and Service persistence
- 🖼️ **Steganography** - Extract hidden data from image files with steghide
- 📄 **PDF Malware Analysis** - Static analysis of weaponized documents
- 🎯 **Incident Response** - Build the complete attack timeline and save Christmas!

Each challenge builds on previous skills - by Day 13, you're combining techniques from ALL previous days!

---

## How to "submit" and follow along

There is no central scoreboard for this series.

Treat the flags as self-checks:

- If your flag matches the expected format and successfully unlocks the next day's ZIP, you're good!
- After the full series is done, I may publish official write-ups/solutions so you can compare.

If you're following along from Linkedin or other socials:
- I'll post when the new day goes live.
- Please don't paste full flags publicly (avoid spoilers)
- Comment something like "**Solved Day X**" and one thing you learned
- If you really want confirmation, you can DM me your flag and I'll tell you if it's correct when I can.

---

## 🛠️ Tools & Prerequisites

### Required for All Days
These tools are essential and you'll use them throughout the challenges:

- **Text Editor** - [VSCode](https://code.visualstudio.com/), Notepad++, or any text editor
- **Web Browser** - For accessing CyberChef and reading documentation
- **ZIP Extraction Tool** - Built into most operating systems (for password-protected artifacts)

### Recommended Tools by Day

Each day's README will remind you which tools you need, but here's the complete list so you can prepare:

| Day | Tools Needed | Challenge Type |
|-----|--------------|----------------|
| **Day 1** | [CyberChef](https://gchq.github.io/CyberChef/) (web-based, no install) | Multi-layer encoding (Base64, Hex, Binary) |
| **Day 2** | [Hashcat](https://hashcat.net/hashcat/) OR [John the Ripper](https://www.openwall.com/john/) | Password hash cracking (MD5) |
| **Day 3** | Text editor with search, `grep` or PowerShell | Log analysis, brute force detection |
| **Day 4** | Python 3, CyberChef | JavaScript deobfuscation |
| **Day 5** | PowerShell (Windows) or `pwsh` (Linux/Mac) | PowerShell malware analysis |
| **Day 6** | Python 3, Flask, [Gobuster](https://github.com/OJ/gobuster) | Web reconnaissance, directory enumeration |
| **Day 7** | [Wireshark](https://www.wireshark.org/) OR `tshark` OR Python 3 | DNS tunneling, PCAP analysis |
| **Day 8** | Python 3, Flask, web browser | SQL injection exploitation |
| **Day 9** | Python 3, Flask, `curl` or PowerShell | API security, secret scanning, authorization bypass |
| **Day 10** | Python 3, Windows OS | Windows persistence hunting (Registry, Tasks, WMI, Services) |
| **Day 11** | [Steghide](http://steghide.sourceforge.net/), WSL (Windows users) | Steganography extraction |
| **Day 12** | Python 3, PDF analysis tools (pdfid, pdf-parser) | PDF malware analysis |
| **Day 13** | All previous tools | Multi-layer final challenge (steganography, hash cracking, XOR, SQL, JS deobfuscation, ROT13) |

### Tool Installation Guides

**CyberChef** (Day 1)
- Web-based: https://gchq.github.io/CyberChef/
- No installation required - just open in browser!

**Hashcat** (Day 2)
- Windows: Download from https://hashcat.net/hashcat/
- Linux: `sudo apt install hashcat`
- Mac: `brew install hashcat`
- Alternative: John the Ripper (easier for beginners)

**John the Ripper** (Day 2 - Alternative to Hashcat)
- Windows/Mac/Linux: https://www.openwall.com/john/
- Linux: `sudo apt install john`
- Beginner-friendly option for password cracking

**Wireshark** (Day 7)
- Windows/Mac: Download from https://www.wireshark.org/
- Linux: `sudo apt install wireshark`
- Includes tshark (command-line version)
- **First-time setup guide included in Day 7!**

**PowerShell** (Day 5)
- Windows: Already installed
- Linux: https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-linux
- Mac: https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-macos

**Python 3** (Days 4, 6, 7, 10)
- Windows: https://www.python.org/downloads/
- Linux: `sudo apt install python3 python3-pip`
- Mac: `brew install python3`

**Flask** (Days 6, 8, 9)
- Install via pip: `pip install flask` or `pip3 install flask`
- Used for running local vulnerable web applications

**Gobuster** (Day 6)
- Linux: `sudo apt install gobuster`
- Mac: `brew install gobuster`
- Windows: Download from https://github.com/OJ/gobuster/releases

**PDF Analysis Tools** (Day 12)
- Download directly from https://github.com/DidierStevens/DidierStevensSuite
- `pdfid.py` - Identifies suspicious PDF elements
- `pdf-parser.py` - Extracts and analyzes PDF objects

**Steghide** (Day 11)
- Linux/WSL: `sudo apt install steghide`
- Mac: `brew install steghide`
- Windows: Use WSL (recommended) or download from http://steghide.sourceforge.net/

**Command Line Tools**
- Linux/Mac: `grep`, `awk`, `sed`, `cut` (usually pre-installed)
- Windows: Use PowerShell OR install [WSL](https://learn.microsoft.com/en-us/windows/wsl/install)

### Important Notes

- **No programming required!** - All challenges can be solved with the tools listed above
- **First-time tool users welcome!** - Each day includes beginner guides when a new tool is introduced
- **Alternative tools work!** - Use whatever you're comfortable with (e.g., Python instead of CyberChef, John instead of Hashcat)
- **Web-based options available** - Many tasks can be done with just CyberChef and a text editor

### Installation Verification

Before Day 1, verify you can:
1. Open a text file in your editor
2. Extract a ZIP file (try the `day01_artifacts` folder)
3. Access https://gchq.github.io/CyberChef/ in your browser

For other tools, install them as you reach each day - you'll get specific setup instructions!

---

## Difficulty progression

- **Days 1-3 (Dec 12-14):** Easy warm-up (multi-layer encoding, hash cracking, log analysis)
- **Days 4-6 (Dec 15-17):** Medium difficulty (JavaScript deobfuscation, PowerShell malware, web reconnaissance)
- **Days 7-9 (Dec 18-20):** Medium-Hard (DNS tunneling/PCAP, SQL injection, API security)
- **Days 10-11 (Dec 21-22):** Hard (Windows persistence hunting, steganography)
- **Day 12 (Dec 23):** Hard (PDF malware analysis)
- **Bonus Day 13 (Dec 24):** Expert (Multi-layer final challenge using ALL previous skills)

Each `dayxx/README.md` includes:
- A story beat for the Jack Frost incident
- The learning goal for that day
- Your mission - exactly what to find or extract
- A flag format reminder
- A couple of hints so you're not hard-stuck

---

## Day Index

### 🎄 The 12 Days (December 12-23)

**Week 1: Discovery - Something's Wrong at the North Pole!**
- [Day 1 (Dec 12) – Sleigh Dashboard Sabotage](./day01/) – Multi-layer encoding (Base64 → Hex → Binary)
- [Day 2 (Dec 13) – Nice List Backup Breach](./day02/) – Password hash cracking (MD5 with custom wordlist)
- [Day 3 (Dec 14) – Workshop Portal Attack](./day03/) – Log analysis, brute force detection
- [Day 4 (Dec 15) – Phishing the Elves](./day04/) – JavaScript deobfuscation, phishing analysis
- [Day 5 (Dec 16) – Malicious Toy Scripts](./day05/) – PowerShell malware deobfuscation
- [Day 6 (Dec 17) – Reindeer Stable Reconnaissance](./day06/) – Web reconnaissance with Gobuster, robots.txt analysis
- [Day 7 (Dec 18) – Stealing the Nice List](./day07/) – DNS tunneling, PCAP analysis, Base32 decoding

**Week 2: Deep Investigation - The Plot Thickens!**
- [Day 8 (Dec 19) – Nice List Database Heist](./day08/) – Hands-on SQL injection exploitation
- [Day 9 (Dec 20) – The GitHub Secret Leak](./day09/) – Secret scanning, API authorization bypass
- [Day 10 (Dec 21) – Persistent Frost Biters](./day10/) – Windows persistence hunting (Registry, Tasks, WMI, Services)
- [Day 11 (Dec 22) – Hidden in Plain Sight](./day11/) – Steganography extraction with steghide
- [Day 12 (Dec 23) – The Final Payload](./day12/) – PDF malware static analysis

### 🎁 Christmas Eve Bonus (December 24)
- [Day 13 (Dec 24) – Operation Blizzard: The Final Payload](./day13/) – Expert multi-layer challenge (7+ layers using ALL previous skills)
