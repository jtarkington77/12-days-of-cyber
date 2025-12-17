# Day 6 Hints - Web Reconnaissance

**Stuck?** Here are progressive hints to help you complete the web reconnaissance challenge.

---

## Hint 1: Installing Gobuster

<details>
<summary>Click to reveal Hint 1</summary>

### How to Install Gobuster:

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

**Alternative: Use dirb or dirbuster**
If Gobuster won't install, you can use `dirb` (similar tool):
```bash
sudo apt install dirb
```

</details>

---

## Hint 2: Starting the Reindeer Portal

<details>
<summary>Click to reveal Hint 2</summary>

### Start the Web Application:

1. **Navigate to artifacts:**
```bash
cd day06/artifacts/
```

2. **Install Flask (if needed):**
```bash
pip3 install flask
```

3. **Run the portal:**
```bash
python3 reindeer_portal.py
```

**You should see:**
```
🦌 REINDEER STABLE MANAGEMENT PORTAL 🦌
Portal is running at: http://127.0.0.1:8000
```

**Keep this terminal open!** The portal must stay running.

**Test it:** Open browser → `http://127.0.0.1:8000` → You should see the Reindeer Portal home page.

</details>

---

## Hint 3: Basic Gobuster Syntax

<details>
<summary>Click to reveal Hint 3</summary>

### Understanding Gobuster Commands:

**Basic syntax:**
```bash
gobuster dir -u TARGET_URL -w WORDLIST
```

**Breaking it down:**
- `gobuster` - The tool
- `dir` - Directory/file enumeration mode
- `-u TARGET_URL` - Target website URL
- `-w WORDLIST` - Path to wordlist file

**For this challenge:**
```bash
gobuster dir -u http://127.0.0.1:8000 -w web_directories.txt
```

</details>

---

## Hint 4: Running Your First Scan

<details>
<summary>Click to reveal Hint 4</summary>

### Step-by-Step Gobuster Scan:

1. **Open a new terminal** (keep the portal running in the first one)

2. **Navigate to artifacts:**
```bash
cd day06/artifacts/
```

3. **Run Gobuster:**
```bash
gobuster dir -u http://127.0.0.1:8000 -w web_directories.txt
```

**What you'll see:**
```
===============================================================
Gobuster v3.6
===============================================================
[+] Url:                     http://127.0.0.1:8000
[+] Wordlist:                web_directories.txt
[+] Status codes:            200,204,301,302,307,401,403
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

</details>

---

## Hint 5: Understanding the Results

<details>
<summary>Click to reveal Hint 5</summary>

### Interpreting Gobuster Output:

**Status Codes:**
- **200 OK** - Page exists and is accessible → investigate!
- **301 Redirect** - Resource moved → follow the redirect
- **403 Forbidden** - Exists but you can't access → might be important
- **404 Not Found** - Doesn't exist → skip it

**What Gobuster found:**
```
/admin          (403) - Admin panel exists but is blocked
/api            (200) - API endpoint is accessible!
/images         (301) - Images directory
/robots.txt     (200) - Robots file exists - check it!
```

**Next steps:**
1. Check `/robots.txt` first - might reveal hidden paths
2. Investigate `/api` - what does it return?
3. Try to find subdirectories under `/admin`

</details>

---

## Hint 6: Checking robots.txt

<details>
<summary>Click to reveal Hint 6</summary>

### Why robots.txt Matters:

**Check it manually:**
```bash
curl http://127.0.0.1:8000/robots.txt
```

Or open in browser: `http://127.0.0.1:8000/robots.txt`

**What you'll find:**
```
User-agent: *
Disallow: /admin
Disallow: /api/v2/
Disallow: /admin/reports/
```

**What this tells you:**
- `/admin` exists (you already knew that)
- `/api/v2/` exists - a versioned API!
- `/admin/reports/` exists - hidden reports directory!

**The `Disallow` lines tell search engines not to index these paths, but attackers can read robots.txt to discover hidden directories!**

</details>

---

## Hint 7: Scanning Subdirectories

<details>
<summary>Click to reveal Hint 7</summary>

### How to Scan Subdirectories:

From robots.txt, you know `/admin/reports/` exists. Scan it!

**Scan the /admin directory:**
```bash
gobuster dir -u http://127.0.0.1:8000/admin -w web_directories.txt
```

**Results:**
```
/admin/reports        (Status: 200) [Size: ...]
```

**Found it!** The `/admin/reports/` directory is accessible.

**Alternative: Direct URL test**
```bash
curl http://127.0.0.1:8000/admin/reports/
```

Or open in browser: `http://127.0.0.1:8000/admin/reports/`

</details>

---

## Hint 8: Finding the Incident Report

<details>
<summary>Click to reveal Hint 8</summary>

### Accessing the Hidden Report:

Once you discover `/admin/reports/` is accessible, visit it:

**Option 1: Browser**
```
http://127.0.0.1:8000/admin/reports/
```

**Option 2: curl**
```bash
curl http://127.0.0.1:8000/admin/reports/
```

**You'll see:** A security incident report in green terminal-style text.

This report documents Jack Frost's reconnaissance attack!

</details>

---

## Hint 9: Finding the Flag

<details>
<summary>Click to reveal Hint 9</summary>

### Where is the Flag?

The flag is in the **Security Incident Report** page at `/admin/reports/`

**Look for the section:**
```
🏁 Investigation Flag
```

The flag will be displayed in a prominent green box in the format:
```
FROST{...}
```

**Read the entire report** - it explains what Jack Frost discovered during his reconnaissance and what security mistakes allowed it.

</details>

---

## Hint 10: Complete Solution (Last Resort)

<details>
<summary>Click to reveal FULL SOLUTION</summary>

### Step-by-Step Complete Solution:

**Step 1: Start the Reindeer Portal**
```bash
cd day06/artifacts/
python3 reindeer_portal.py
# Keep this running
```

**Step 2: Open new terminal and run Gobuster**
```bash
cd day06/artifacts/
gobuster dir -u http://127.0.0.1:8000 -w web_directories.txt
```

**Step 3: Check robots.txt**
```bash
curl http://127.0.0.1:8000/robots.txt
```

Output reveals:
```
Disallow: /admin/reports/
```

**Step 4: Access the hidden directory**
```bash
curl http://127.0.0.1:8000/admin/reports/
```

Or browser: `http://127.0.0.1:8000/admin/reports/`

**Step 5: Find the flag in the incident report**

The flag will be displayed in the incident report in `FROST{...}` format!

</details>

---

## Bonus: Advanced Gobuster Options

<details>
<summary>Click to reveal Advanced Tips</summary>

### Speed Up Scans:

**Use more threads (faster):**
```bash
gobuster dir -u http://127.0.0.1:8000 -w web_directories.txt -t 50
# -t 50 = 50 parallel threads
```

**Show only specific status codes:**
```bash
gobuster dir -u http://127.0.0.1:8000 -w web_directories.txt -s "200,301"
# Only show 200 OK and 301 Redirects
```

**Add file extensions:**
```bash
gobuster dir -u http://127.0.0.1:8000 -w web_directories.txt -x php,html,txt
# Test for admin.php, admin.html, admin.txt, etc.
```

**Recursive scanning:**
```bash
gobuster dir -u http://127.0.0.1:8000 -w web_directories.txt -r
# Automatically follow redirects
```

**Use a bigger wordlist:**
```bash
# If installed on Kali/Ubuntu:
gobuster dir -u http://127.0.0.1:8000 -w /usr/share/wordlists/dirb/common.txt
```

</details>

---

## Common Issues

### Issue: Gobuster not installed

**Fix:**
```bash
sudo apt update
sudo apt install gobuster
```

Or use an alternative:
```bash
sudo apt install dirb
dirb http://127.0.0.1:8000 web_directories.txt
```

### Issue: Portal won't start (Flask error)

**Fix:**
```bash
pip3 install flask
# Or
python3 -m pip install flask
```

### Issue: Connection refused

**Problem:** Portal isn't running

**Fix:** Make sure you started `python3 reindeer_portal.py` and it's still running

### Issue: Found /admin but get 403 Forbidden

**This is expected!** `/admin` is protected. But `/admin/reports/` is accidentally exposed.

**Scan subdirectories:**
```bash
gobuster dir -u http://127.0.0.1:8000/admin -w web_directories.txt
```

---

## Learning Checklist

After completing this challenge, you should understand:
- ✅ How to use Gobuster for directory enumeration
- ✅ What robots.txt reveals to attackers
- ✅ HTTP status codes (200, 301, 403, 404)
- ✅ How to discover hidden endpoints
- ✅ Why "security through obscurity" fails
- ✅ How attackers map web application structure

---

**Remember:** Only perform web reconnaissance on systems you own or have permission to test!
