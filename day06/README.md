[← Previous Day](../day05/README.md) | [Main README](../README.md) | **Day 6** | [Next Day →](../day07/README.md)

---

![Day 6](../Images/Day06.png)

# 🎄 Day 6 (December 17) - The Reindeer Stable Reconnaissance

## ⚡ QUICK START

**Before you begin, install the required tools:**

```bash
# Install Gobuster
sudo apt install gobuster -y   # Linux/WSL
brew install gobuster           # macOS

# Install Flask (REQUIRED!)
pip3 install flask

# Verify installations
gobuster version
python3 -c "import flask; print('Flask installed!')"
```

**Then extract artifacts:**
```bash
7z x day06_artifacts.zip -p"YOUR_DAY5_FLAG"
```

---

## 🎅 The Story

**December 17, 2024 - 10:15 AM (North Pole Time)**

Dekker Frostbeard is reviewing firewall logs when Sarah Hollycode, the North Pole's Web Application Security Engineer, rushes into the SOC with her laptop open, concern etched across her face.

"Dekker, we have a problem." Sarah sets her laptop on his desk, showing a terminal window filled with scrolling log entries. "The Reindeer Stable Management Portal - the web application that tracks reindeer assignments, flight schedules, and delivery routes - got absolutely hammered last night with suspicious traffic."

Dekker looks up from his screen, his Santa hat slightly askew from a long morning. "Define 'hammered.'"

Sarah pulls up a graph showing a massive spike in traffic. "Over 800 requests in 30 minutes from a single IP address. Systematic enumeration of every possible directory, endpoint scanning, vulnerability testing - this wasn't a customer browsing. This was reconnaissance."

Tom Icicle approaches with his ever-present coffee mug, having overheard the conversation. "Let me guess - 203.0.113 subnet?"

"Bingo." Sarah highlights a section of the access log. "203.0.113.99 - the same C2 server from yesterday's PowerShell malware."

Tom sets down his coffee, now fully alert. "The attacker's escalating. Malware deployment yesterday, web reconnaissance today. What was he looking for?"

Sarah scrolls through the logs, highlighting entries. "Look at the User-Agent strings: `Gobuster/3.6`, `Nikto/2.5.0`. These are automated security scanning tools. He's mapping our entire web application infrastructure."

Merry Tinselcode wheels her chair over. "What kinds of paths was he testing?"

"Everything," Sarah explains. "Admin panels, backup directories, configuration files, database interfaces, API endpoints. He's brute forcing every common vulnerable path he can think of - `/admin`, `/backup`, `/.env`, `/api/*`. Classic directory enumeration."

Dekker leans forward, studying the HTTP status codes. "Most of these are returning 404 Not Found or 403 Forbidden. That's good, right?"

Sarah's expression darkens. "Mostly. But there's one directory..." She filters for HTTP 200 responses. "Here. `/admin/reports/`. It returned 200 OK. That directory shouldn't be publicly accessible."

Tom's face goes pale. "What's in there?"

"Confidential incident reports. Security assessments. Vulnerability findings." Sarah pulls up the endpoint. "I thought we had authentication on the entire `/admin` tree, but it looks like `/admin/reports/` was accidentally left open during last week's deployment."

"So Jack Frost read our security incident reports?" Merry asks.

"Worse," Sarah says. "He also discovered our internal `/api/v2/delivery-routes` endpoint. He knows exactly what data we have and how our systems are structured."

Tom stands up. "We need to understand exactly what he did. Merry, I want you to replicate Jack Frost's reconnaissance attack in our test environment. Use the same tools he used - Gobuster for directory enumeration. Find every endpoint he discovered."

Merry nods. "I'll spin up a copy of the Reindeer Portal in our lab. If I can successfully enumerate the same directories Jack did, I'll understand his methodology."

"Good," Dekker says. "This isn't just about fixing the misconfiguration - we need to learn how attackers map our infrastructure so we can defend better."

Sarah adds, "Once you find what he found, document it. I want to know every path he tested, every tool signature he left, and especially what was in those incident reports he accessed."

Outside the SOC window, reindeer practice their flight formations against a bright blue sky. Inside, the elves prepare to think like an attacker to defend their systems.

---

## 🛠️ Prerequisites & Setup

**This is your first web reconnaissance challenge!** You'll use **Gobuster** - a professional directory/file enumeration tool used by penetration testers worldwide.

### Tools You'll Need

**Gobuster (Required)**
- **What it is:** Fast directory/file brute forcing tool
- **Used by:** Penetration testers, bug bounty hunters, red teams
- **Why it's important:** Discovers hidden web content that isn't linked publicly

**Installation:**

**Kali Linux / Parrot OS:**
```bash
# Already pre-installed!
gobuster version
```

**Ubuntu / Debian / WSL:**
```bash
sudo apt update
sudo apt install gobuster -y
```

**macOS:**
```bash
brew install gobuster
```

**Verify installation:**
```bash
gobuster version
# Should show: Gobuster v3.x.x
```

**Python 3 & Flask (Required)**
- **What it is:** Web framework for running the Reindeer Portal
- **Installation:**
```bash
pip3 install flask
```

### Getting Started Checklist

Before you begin:
- [ ] Install Gobuster (`gobuster version` to verify)
- [ ] Install Flask (`pip3 install flask`)
- [ ] Extract `day06_artifacts.zip` (password: Day 5's flag)
- [ ] Read the [Web Reconnaissance lesson](./lesson_web_reconnaissance.md)
- [ ] Understand you'll scan a LOCAL test portal, not a real website

---

## 📚 Lesson: Learn the Technique

**Before starting this challenge, read the lesson:**
📖 [Web Application Reconnaissance - Learning to Think Like an Attacker](./lesson_web_reconnaissance.md)

This lesson covers:
- What web reconnaissance is and why attackers do it
- How directory enumeration tools work (Gobuster, Nikto, dirb)
- Understanding HTTP status codes (200, 301, 403, 404)
- What attackers look for (admin panels, APIs, backups, configs)
- How to interpret Gobuster output
- Defense strategies against web recon
- Legal and ethical considerations (ONLY scan systems you own!)

**New to web security?** Start with the lesson - it explains everything from scratch!

---

## 🎯 **YOUR MISSION: Replicate Jack Frost's Reconnaissance!**

You are **Merry Tinselcode**, Senior Security Elf. Your mission is to replicate Jack Frost's web reconnaissance attack in a safe test environment to understand what he discovered.

### Mission Objectives:

1. **Start the Reindeer Portal** (test environment)
   - Run the web application locally
   - Verify it's accessible at `http://127.0.0.1:8000`

2. **Use Gobuster** to enumerate directories
   - Discover hidden endpoints and admin interfaces
   - Identify which paths return 200 OK vs 403 Forbidden

3. **Check robots.txt** for clues
   - Attackers always check robots.txt
   - It might reveal hidden paths

4. **Discover the hidden directory**
   - Find the accidentally exposed admin reports
   - Access the incident report

5. **Extract the flag**
   - The flag is in the incident report
   - Proves you understand web reconnaissance

---

## 🔍 Artifacts

**Password required:** Use Day 5's flag to extract `day06_artifacts.zip`

In the `artifacts/` folder you'll find:

| File | Description |
|------|-------------|
| `reindeer_portal.py` | Flask web application - the Reindeer Stable Portal |
| `web_directories.txt` | Wordlist for Gobuster (common directory names) |

---

## 📖 **Step-by-Step Challenge Guide**

### Step 1: Start the Reindeer Portal

First, run the web application:

```bash
cd day06/artifacts/
python3 reindeer_portal.py
```

**You should see:**
```
🦌 REINDEER STABLE MANAGEMENT PORTAL 🦌
Portal is running at: http://127.0.0.1:8000
```

**Keep this terminal open!** The portal must stay running.

**Test it:** Open browser → `http://127.0.0.1:8000` → You should see the Reindeer Portal home page.

---

### Step 2: Understanding the Target

Before scanning, explore manually:

**Visit the home page:**
```
http://127.0.0.1:8000
```

You'll see a public-facing portal. Most functionality requires authentication.

**Try some common paths manually:**
```
http://127.0.0.1:8000/admin     → 403 Forbidden (exists but blocked)
http://127.0.0.1:8000/api       → 200 OK (works!)
http://127.0.0.1:8000/test      → 404 Not Found (doesn't exist)
```

**Manual testing is slow. Gobuster can test hundreds of paths in seconds!**

---

### Step 3: Your First Gobuster Scan

Open a **new terminal** (keep the portal running in the first one).

**Navigate to artifacts:**
```bash
cd day06/artifacts/
```

**Run Gobuster:**
```bash
gobuster dir -u http://127.0.0.1:8000 -w web_directories.txt
```

**Command breakdown:**
- `gobuster` = The tool
- `dir` = Directory enumeration mode
- `-u http://127.0.0.1:8000` = Target URL
- `-w web_directories.txt` = Wordlist to use

**What you'll see:**
```
===============================================================
Gobuster v3.6
===============================================================
[+] Url:                     http://127.0.0.1:8000
[+] Wordlist:                web_directories.txt
===============================================================
Starting gobuster...
===============================================================
/admin                (Status: 403) [Size: 45]
/api                  (Status: 200) [Size: 156]
/images               (Status: 301)
/robots.txt           (Status: 200) [Size: 183]
===============================================================
Finished
===============================================================
```

**Gobuster found:**
- `/admin` → 403 Forbidden (exists but can't access)
- `/api` → 200 OK (accessible!)
- `/images` → 301 Redirect
- `/robots.txt` → 200 OK (exists!)

---

### Step 4: Check robots.txt (IMPORTANT!)

Attackers always check `robots.txt` - it often reveals hidden paths!

**Check it:**
```bash
curl http://127.0.0.1:8000/robots.txt
```

Or browser: `http://127.0.0.1:8000/robots.txt`

**You'll find:**
```
User-agent: *
Disallow: /admin
Disallow: /api/v2/
Disallow: /admin/reports/
```

**What this reveals:**
- `/admin` - Already knew that
- `/api/v2/` - Versioned API!
- `/admin/reports/` - Hidden reports directory!

**robots.txt is supposed to tell search engines what NOT to index. But it tells attackers exactly where the sensitive stuff is!**

---

### Step 5: Investigate the Hidden Directory

From robots.txt, you know `/admin/reports/` exists. Access it:

**Option 1: Direct URL**
```bash
curl http://127.0.0.1:8000/admin/reports/
```

**Option 2: Browser**
```
http://127.0.0.1:8000/admin/reports/
```

**Option 3: Scan the subdirectory**
```bash
gobuster dir -u http://127.0.0.1:8000/admin -w web_directories.txt
```

---

### Step 6: Find the Flag

Once you access `/admin/reports/`, you'll see a **Security Incident Report** in green terminal-style text.

This report documents Jack Frost's reconnaissance attack!

**The flag is in the report** in a section labeled:
```
🏁 Investigation Flag
```

The flag format: `FROST{...}`

**Read the entire report** - it explains what Jack Frost discovered and what security mistakes allowed him to find it.

---

## 🧪 **What You're Learning**

### Technical Skills:
- ✅ Using Gobuster for directory enumeration
- ✅ Understanding HTTP status codes (200, 301, 403, 404)
- ✅ Reading robots.txt for reconnaissance
- ✅ Discovering hidden endpoints and admin interfaces
- ✅ Thinking like an attacker to find vulnerabilities

### Security Concepts:
- ✅ Why "security through obscurity" doesn't work
- ✅ How misconfigurations expose sensitive endpoints
- ✅ The importance of proper access controls
- ✅ Why robots.txt shouldn't contain sensitive paths
- ✅ How attackers map web application structure

### MITRE ATT&CK:
- **Tactic:** TA0043 - Reconnaissance
- **Technique:** T1595 - Active Scanning
- **Sub-technique:** T1595.002 - Vulnerability Scanning

---

## 🎓 **Bonus Challenges (Optional)**

Want to learn more? Try these:

### Challenge 1: Find All Hidden Paths

Can you discover:
- The `/api/v2/delivery-routes` endpoint (mentioned in robots.txt)
- Any other hidden directories?

**Hint:** Try scanning subdirectories:
```bash
gobuster dir -u http://127.0.0.1:8000/api -w web_directories.txt
```

### Challenge 2: Speed Up Your Scan

Make Gobuster faster:
```bash
gobuster dir -u http://127.0.0.1:8000 -w web_directories.txt -t 50
# -t 50 = 50 parallel threads (default is 10)
```

How much faster is it?

### Challenge 3: Test File Extensions

Look for specific file types:
```bash
gobuster dir -u http://127.0.0.1:8000 -w web_directories.txt -x php,html,txt,bak
# Tests admin.php, admin.html, admin.txt, admin.bak, etc.
```

### Challenge 4: Use a Bigger Wordlist

If you have a larger wordlist (like on Kali Linux):
```bash
gobuster dir -u http://127.0.0.1:8000 -w /usr/share/wordlists/dirb/common.txt
```

What additional endpoints do you find?

---

## 🛡️ **Defense Strategies**

Now that you understand web reconnaissance, here's how to defend:

### 1. Proper Access Controls
```python
# Protect ALL admin routes
@app.route('/admin/<path:subpath>')
@require_authentication
def admin_area(subpath):
    # Enforce auth on entire /admin tree
```

### 2. Don't Leak Paths in robots.txt
```
# BAD - Reveals sensitive paths
Disallow: /admin
Disallow: /api/v2/secret
Disallow: /backup

# BETTER - Generic
Disallow: /private/
```

### 3. Consistent Error Responses
```
# Return 404 for everything unauthorized
# Don't distinguish between "doesn't exist" and "not allowed"
```

### 4. Rate Limiting
```python
# Detect and block directory brute forcing
if requests_per_minute > 100:
    return "Rate limit exceeded", 429
```

### 5. Web Application Firewall (WAF)
- Detects Gobuster/Nikto user-agent strings
- Blocks rapid sequential requests
- Alerts on suspicious patterns

### 6. Security Headers
```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Strict-Transport-Security: max-age=31536000
```

---

## ❓ Need Help?

- **Can't install Gobuster?** See [hints.md](./hints.md) - Hint 1
- **Portal won't start?** See [hints.md](./hints.md) - Hint 2
- **Don't understand Gobuster output?** See [hints.md](./hints.md) - Hint 5
- **Can't find robots.txt clues?** See [hints.md](./hints.md) - Hint 6
- **Can't find the hidden directory?** See [hints.md](./hints.md) - Hint 7
- **Need the full solution?** See [hints.md](./hints.md) - Hint 10

**Progressive hints available** - from installation to complete solution.

---

## 📚 **Additional Resources**

**Web Reconnaissance Tools:**
- Gobuster GitHub: https://github.com/OJ/gobuster
- Dirb Tutorial: https://tools.kali.org/web-applications/dirb
- Nikto Scanner: https://github.com/sullo/nikto

**Wordlists:**
- SecLists: https://github.com/danielmiessler/SecLists
- FuzzDB: https://github.com/fuzzdb-project/fuzzdb

**Web Security:**
- OWASP Top 10: https://owasp.org/www-project-top-ten/
- PortSwigger Web Security Academy: https://portswigger.net/web-security

---

## 🏁 **Next Steps**

Once you have the flag (it will be in `FROST{...}` format), you're ready for Day 7!

**Use your Day 6 flag to unlock:**
```bash
7z x day07_artifacts.zip -p"YOUR_DAY6_FLAG"
```

**What's Next?**
Day 7: Jack Frost exfiltrates data via DNS tunneling. You'll analyze network traffic to decode his hidden communications!

---

**Challenge Status:** 🟢 Ready to solve!
**Difficulty:** ⭐⭐ Intermediate
**Time Estimate:** 30-45 minutes
**Tools Required:** Gobuster, Python/Flask, browser or curl

---

[← Previous Day](../day05/README.md) | [Main README](../README.md) | **Day 6** | [Next Day →](../day07/README.md)
