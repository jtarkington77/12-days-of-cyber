# 12 Days of Cyber – The Breaches of FrostNet ❄️

**12 days. 12 breaches. 12 small, hands-on security challenges.**

This repo is a holiday mini–CTF series built around a fictional company called **FrostNet** – a global logistics and messaging platform that gets hammered during the holiday season.

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
- **Day 1** artifacts are open.
- **Days 2-12** articats live in **password-protected ZIP files**.
- The password for a day's ZIP is always the previous day's flag, including `FROST{}`.

**Example flow:**
1. Solve **Day 1** -> you get a flag that looks like `FROST{example_flag}`.
2. Go to `day02/` and download `day02_artifacts.zip`.
3. Use the exact **Day 1 flag** (case sensitive, including `FROST{}`) as the password to open that ZIP.
4. Solve Day 2 -> that flag unlocks **Day 3**, and so on.

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

- **Days 1-3:** Very easy -> easy (encodings, basic hashes, simple logs)
- **Days 4-6:** Low-mid -> mid (phishing, HTML, basic script review)
- **Days 7-9:** Mid -> upper-mid (PCAPs, DB/API-style artifacts)
- **Days 10-12:** Harder thinking (persistence, misconfig, IR timeline)

Each `dayxx/README.md` includes:
- A story beat for the FrostNet incident
- The learning goal for that day
- Your mission - exactly what to find or extract
- A flag format reminder
- A couple of hints so you're not hard-stuck

---

## Day index

Links will fill in as days go live:

- [Day 1 – Status Page Glitch](./day01/) – encodings & base64 warm-up
- [Day 2 – TBD](./day02/) – coming soon
- [Day 3 – TBD](./day03/) – coming soon
- [Day 4 – TBD](./day04/) – coming soon
- [Day 5 – TBD](./day05/) – coming soon
- [Day 6 – TBD](./day06/) – coming soon
- [Day 7 – TBD](./day07/) – coming soon
- [Day 8 – TBD](./day08/) – coming soon
- [Day 9 – TBD](./day09/) – coming soon
- [Day 10 – TBD](./day10/) – coming soon
- [Day 11 – TBD](./day11/) – coming soon
- [Day 12 – TBD](./day12/) – coming soon
