# 12 Days of Cyber – The Breaches of FrostNet ❄️

**13 days. 1 major breach. 13 hands-on security challenges.**

Welcome to the FrostNet Incident - a holiday CTF series running from **December 12th through December 24th, 2024**.

## The Story

It's peak holiday season at **FrostNet Global Solutions**, a logistics and communications platform processing 10 million packages daily. The Security Operations Center is already stretched thin when Senior SOC Analyst Maya Chen notices something odd on the public status page...

What starts as a minor anomaly quickly spirals into a full-scale security incident. Over the next 13 days, you'll step into the shoes of the FrostNet security team as they uncover a sophisticated breach by an attacker group called the "Frost Giants."

Each day reveals a new piece of the puzzle:
- **Week 1 (Dec 12-18):** Discovery and investigation of the initial compromise
- **Week 2 (Dec 19-24):** Deep forensics, lateral movement analysis, and the complete incident timeline

This isn't just a CTF - it's a realistic incident response scenario built for learning.

Each day unlocks:

- A short **story beat** from the FrostNet incident  
- A focused **lesson** (logs, encodings, phishing, PCAPs, etc.)  
- One or more **artifacts** you can actually work with (JSON, logs, scripts, PCAPs, HTML, configs)  
- A **flag** in the format:

```text
FROST{something_here}
```

The idea is " 12 Days of Cyber": one small challenge per day up to Christmas, each slightly harder than the last.

---

## How it works

- Each day lives in its own folder: `day01`, `day02`,...`day12`.
- Inside each folder you'll find:
    - `README.md` - story, lesson, instructions, and hints.
    - One or more artifact files (for example: `status.json`, `hashes.txt`, `traffic.pcap`), `newsletter.html`, ect.)

## Gated progression with flags

To make this feel like a proper chain:
- **Day 1 (Dec 12)** artifacts are open.
- **Days 2-13 (Dec 13-24)** artifacts live in **password-protected ZIP files**.
- The password for a day's ZIP is always the previous day's flag, including `FROST{}`.

**Example flow:**
1. Solve **Day 1 (Dec 12)** → you get a flag that looks like `FROST{3x@mpl3_fl@g}`.
2. Go to `day02/` and download `day02_artifacts.zip`.
3. Use the exact **Day 1 flag** (case sensitive, including `FROST{}`) as the password to open that ZIP.
4. Solve Day 2 → that flag unlocks **Day 3**, and so on through December 24th.

---

## What you'll practice

Across the 12 days, you'll touch things like:
- Recognizing and decoding **base64 / simple encodings**
- Basic **hash identification** and cracking fundamentals
- Reading **authentication logs** and spotting brute-force behavior
- Inspecting **phishing / HTML /email content**
- Light **code review** for suspicious scripts
- Basic **network foresniscs** on small PCAPs
- Spotting weird accounts, misconfigurations, and API abuse
- Building a simple **incident timeline** from scattered clues

Each changlenge is intentionally small. The goal is daily reps, not a marathon exam.

---

## How to "submit" and follow along

There is no central scoreboard for this series.

Treat the flags as self-checks:

- If your flag matches the expected format and successfully unlocks the next day's ZIP, you're good!
- After the full series is done, I may publish official write-ups/solutions so you can compare.

If you're following along from Linkedin or other socials:
- I'll post when the new day goes live.
-Please don't paste full flags publicly (avoid spoilers)
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
- **Command line: (Linux/WSL or PowerShell):** For grepping and parsing logs

Use whatever fits your workflow. The focus is on the thinking, not the specific tools.

---

## Difficulty progression

- **Days 1-3 (Dec 12-14):** Easy warm-up (encodings, hash cracking, log analysis)
- **Days 4-6 (Dec 15-17):** Medium difficulty (phishing, script analysis, web logs)
- **Days 7-9 (Dec 18-20):** Hard challenges (PCAP analysis, database forensics, API logs)
- **Days 10-13 (Dec 21-24):** Very Hard (Windows forensics, cloud security, incident response)

Each `dayxx/README.md` includes:
- A story beat for the FrostNet incident
- The learning goal for that day
- Your mission - exactly what to find or extract
- A flag format reminder
- A couple of hints so you're not hard-stuck

---

## Day index

Links will go live each day from December 12-24:

### Week 1: Discovery & Investigation
- [Day 1 (Dec 12) – Status Page Anomaly](./day01/) – Double encoding, reconnaissance detection
- [Day 2 (Dec 13) – Shadow Backup Files](./day02/) – Hash cracking, password analysis
- [Day 3 (Dec 14) – Brute Force Blues](./day03/) – Log parsing, attack pattern recognition
- [Day 4 (Dec 15) – Phishing Season](./day04/) – Email analysis, HTML deobfuscation
- [Day 5 (Dec 16) – PowerShell Malware](./day05/) – Script deobfuscation, malware analysis
- [Day 6 (Dec 17) – Web Recon](./day06/) – Web server logs, reconnaissance detection
- [Day 7 (Dec 18) – Network Exfiltration](./day07/) – PCAP analysis, DNS tunneling

### Week 2: Deep Forensics & Response
- [Day 8 (Dec 19) – Database Breach](./day08/) – SQL log analysis, injection detection
- [Day 9 (Dec 20) – API Key Theft](./day09/) – API log analysis, secret exposure
- [Day 10 (Dec 21) – Persistence Mechanisms](./day10/) – Windows forensics, persistence detection
- [Day 11 (Dec 22) – Cloud Misconfigurations](./day11/) – AWS security, policy analysis
- [Day 12 (Dec 23) – Lateral Movement](./day12/) – Event log analysis, attack path mapping
- [Day 13 (Dec 24) – Incident Timeline](./day13/) – Complete IR, MITRE ATT&CK mapping
