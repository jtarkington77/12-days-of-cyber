# 🎄 12 Days of Cyber – Saving Christmas at the North Pole ❄️

**12 days of challenges. 1 Christmas crisis. Can you help the elves save the holiday?**

Welcome to the **North Pole Security Incident** - a holiday CTF series running from **December 12th through December 23rd, 2025**, with a special Christmas Eve bonus challenge on **December 24th**.

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
1. Solve **Day 1** → Find flag: `FROST{sl31gh_d@shb0@rd_pwn3d}`
2. Go to `day02/` and extract `day02_artifacts.zip`
3. Use the exact **Day 1 flag** as the password (case sensitive!)
4. Solve Day 2 → Use that flag to unlock Day 3, and so on...
5. Day 12's flag unlocks the **Christmas Eve Bonus** on December 24th!

This creates a progression chain - you must solve each day to continue helping the elves!

---

## 🛡️ Skills You'll Learn

Across the 12 days, you'll help the elves by learning:

- 🔤 **Encoding & Decoding** - Uncover hidden messages (base64, hex, ROT13)
- 🔐 **Password Cracking** - Recover compromised elf accounts (hashcat, custom wordlists)
- 📋 **Log Analysis** - Track down attackers in system logs (PowerShell, pattern recognition)
- 📧 **Phishing Detection** - Identify fake "Toy Recall" emails targeting elves
- 💻 **Malware Analysis** - Deobfuscate malicious scripts left by Jack Frost
- 🌐 **Web Security** - Analyze web server logs for reconnaissance
- 📡 **Network Forensics** - Catch data exfiltration (Wireshark, PCAP analysis)
- 🗄️ **Database Forensics** - Investigate Nice List database breaches
- ☁️ **Cloud Security** - Find misconfigurations in North Pole Cloud infrastructure
- 🎯 **Incident Response** - Build the complete attack timeline and save Christmas!

Each challenge builds on previous skills - by Day 12, you're using techniques from across the whole series!

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

## Tools you may want

You can do a lot of this with just a browser and a text editor, but the following can help:

- **Text editor:** [VSCode](https://code.visualstudio.com/)
- **Encodings / transformations:** [CyberChef](https://gchq.github.io/CyberChef/)
- **Hash cracking:** [John the Ripper](https://www.openwall.com/john/)
- **Network analysis:** [Wireshark](https://www.wireshark.org/)
- **HTML inspection:** [Browser DevTools](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/What_are_browser_developer_tools)
- **Command line (Linux/WSL or PowerShell):** For grepping and parsing logs

Use whatever fits your workflow. The focus is on the thinking, not the specific tools.

---

## Difficulty progression

- **Days 1-3 (Dec 12-14):** Easy warm-up (encodings, hash cracking, log analysis)
- **Days 4-6 (Dec 15-17):** Medium difficulty (phishing, script analysis, web logs)
- **Days 7-9 (Dec 18-20):** Hard challenges (PCAP analysis, database forensics, API logs)
- **Days 10-12 (Dec 21-23):** Very Hard (Windows forensics, cloud security, lateral movement)
- **Bonus Day (Dec 24):** Synthesis Challenge (Complete incident timeline, MITRE ATT&CK mapping)

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
- [Day 1 (Dec 12) – Sleigh Dashboard Sabotage](./day01/) – Encoded messages, multi-layer decoding
- [Day 2 (Dec 13) – Nice List Backup Breach](./day02/) – Password hash cracking, elf account recovery
- [Day 3 (Dec 14) – Workshop Portal Attack](./day03/) – Log analysis, brute force detection
- [Day 4 (Dec 15) – Phishing the Elves](./day04/) – Email analysis, fake toy recalls
- [Day 5 (Dec 16) – Malicious Toy Scripts](./day05/) – PowerShell deobfuscation, malware analysis
- [Day 6 (Dec 17) – Reindeer Stable Reconnaissance](./day06/) – Web server log analysis
- [Day 7 (Dec 18) – Stealing the Nice List](./day07/) – Network forensics, data exfiltration

**Week 2: Deep Investigation - The Plot Thickens!**
- [Day 8 (Dec 19) – Nice List Database Heist](./day08/) – Database forensics, SQL injection
- [Day 9 (Dec 20) – Gift API Compromise](./day09/) – API security, leaked secrets
- [Day 10 (Dec 21) – Persistent Frost Biters](./day10/) – Windows persistence mechanisms
- [Day 11 (Dec 22) – North Pole Cloud Chaos](./day11/) – Cloud misconfigurations, S3 buckets
- [Day 12 (Dec 23) – Jack Frost's Path](./day12/) – Lateral movement, attack chain mapping

### 🎁 Christmas Eve Bonus (December 24)
- [Bonus Day (Dec 24) – Saving Christmas!](./day13/) – Complete incident response, timeline synthesis, final report
