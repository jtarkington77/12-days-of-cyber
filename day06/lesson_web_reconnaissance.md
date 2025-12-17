# Web Application Reconnaissance - Learning to Think Like an Attacker

## What is Web Reconnaissance?

**Web reconnaissance** (or web recon) is the process of discovering and mapping a web application's structure, endpoints, and potential vulnerabilities **before** attempting exploitation.

Think of it like casing a building before a heist - you need to know:
- All the entrances (endpoints)
- Which doors are locked (403 Forbidden)
- Which doors are unlocked (200 OK)
- Hidden rooms nobody talks about (undocumented endpoints)

### Why Attackers Do Web Recon:

1. **Find attack surface** - Discover all possible entry points
2. **Identify technologies** - What frameworks, servers, languages are used
3. **Locate sensitive data** - Admin panels, backups, config files
4. **Discover hidden functionality** - Debug modes, test endpoints, old versions
5. **Map the application** - Understand how it's structured

---

## Common Web Recon Tools

### 1. Gobuster - Directory/File Brute Forcing

**What it does:** Tests thousands of common directory and file names to find hidden content

**How it works:**
```
Wordlist: admin, backup, config, test, dev, api...
Test: https://example.com/admin → 404 Not Found
Test: https://example.com/backup → 403 Forbidden (exists but protected!)
Test: https://example.com/api → 200 OK (found it!)
```

**Why attackers love it:**
- Fast (tests hundreds of paths per second)
- Customizable wordlists
- Finds things not linked on the website

---

### 2. Nikto - Web Server Vulnerability Scanner

**What it does:** Scans web servers for known vulnerabilities, misconfigurations, and security issues

**How it works:**
- Tests for outdated software versions
- Checks for dangerous files (config backups, database exports)
- Identifies security headers missing
- Finds default credentials
- Detects common misconfigurations

**Why defenders use it too:**
- Automated security assessment
- Finds issues before attackers do
- Part of routine security audits

---

### 3. Burp Suite - Web Proxy and Scanner

**What it does:** Intercepts HTTP traffic between browser and server, allowing inspection and modification

**How it works:**
- Acts as a proxy between you and the website
- Captures all requests/responses
- Lets you modify requests before sending
- Scans for vulnerabilities (SQLi, XSS, etc.)

---

## Directory Enumeration: How It Works

### The Attack Process:

**Step 1: Choose a Wordlist**
```
Common wordlists:
- /usr/share/wordlists/dirb/common.txt (4,614 entries)
- /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt (220,560 entries)
- Custom wordlists for specific technologies
```

**Step 2: Launch Gobuster**
```bash
gobuster dir -u http://target.com -w /path/to/wordlist.txt
```

**Step 3: Analyze Results**
```
Found: /admin         (Status: 403) [Size: 1234]
Found: /api           (Status: 200) [Size: 5678]
Found: /backup        (Status: 200) [Size: 9012]
Found: /images        (Status: 301) → /images/
Found: /robots.txt    (Status: 200) [Size: 42]
```

**Step 4: Investigate Findings**
- 200 OK = Accessible, investigate further
- 403 Forbidden = Exists but blocked, might be important
- 301 Redirect = Follow the redirect
- 404 Not Found = Doesn't exist

---

## HTTP Status Codes: What They Mean

| Code | Meaning | Significance |
|------|---------|--------------|
| **200** | OK | Page exists and is accessible |
| **301** | Moved Permanently | Resource redirected (follow it) |
| **302** | Found (Temporary Redirect) | Resource temporarily moved |
| **400** | Bad Request | Malformed request |
| **401** | Unauthorized | Authentication required |
| **403** | Forbidden | Resource exists but you can't access it |
| **404** | Not Found | Resource doesn't exist |
| **500** | Internal Server Error | Server-side error (might reveal info) |
| **503** | Service Unavailable | Server overloaded or down |

### Security Implications:

**403 vs 404:**
- **403** = "This exists, but you can't see it" → Attacker knows it's real
- **404** = "This doesn't exist" → Attacker moves on

**Best practice for security:**
- Return 404 for nonexistent AND unauthorized resources
- Don't leak information about what exists

---

## What Attackers Look For

### High-Value Targets:

**1. Admin Panels**
```
/admin, /administrator, /admin.php, /wp-admin
/cpanel, /control-panel, /dashboard
```

**2. Backup Files**
```
/backup, /backups, /backup.zip, /db_backup.sql
/site-backup.tar.gz, /.git, /.svn
```

**3. Configuration Files**
```
/.env, /config.php, /wp-config.php, /web.config
/settings.ini, /database.yml
```

**4. API Endpoints**
```
/api, /api/v1, /api/v2, /graphql, /rest
/api/admin, /api/internal, /api/debug
```

**5. Development/Test Endpoints**
```
/test, /dev, /staging, /debug, /phpmyadmin
/phpinfo.php, /test.php, /debug.log
```

**6. Sensitive Data**
```
/users, /customers, /passwords.txt, /credentials
/private, /confidential, /internal
```

---

## Real-World Example: The Attack Chain

### Scenario: Attacker targets a company website

**Step 1: Initial Scan**
```bash
gobuster dir -u https://example.com -w common.txt
```

**Results:**
```
/admin         → 403 Forbidden (exists!)
/api           → 200 OK (accessible!)
/backup        → 404 Not Found
/images        → 301 → /images/
/robots.txt    → 200 OK
```

**Step 2: Investigate robots.txt**
```
# robots.txt
User-agent: *
Disallow: /admin
Disallow: /api/internal
Disallow: /old-site
```

**What the attacker learns:**
- `/admin` exists (confirmed by 403 + robots.txt)
- `/api/internal` exists and is supposed to be hidden
- `/old-site` might have vulnerabilities

**Step 3: Test Found Endpoints**
```bash
curl https://example.com/api/internal
# Response: {"error": "Unauthorized", "hint": "Try /api/internal/debug"}

curl https://example.com/api/internal/debug
# Response: {"database": "mysql://user:pass@localhost/prod_db", ...}
```

**Result:** Attacker found debug endpoint leaking database credentials!

---

## Defending Against Web Reconnaissance

### 1. Remove Unnecessary Endpoints
```
Don't deploy to production:
- /test, /dev, /debug, /phpinfo.php
- Development tools and test files
- Old/deprecated versions
```

### 2. Proper Access Controls
```
Implement authentication on:
- Admin panels (/admin, /dashboard)
- API endpoints (/api/*)
- Internal tools
```

### 3. Consistent Error Responses
```python
# BAD - Leaks information
if user.exists() and not user.password_matches():
    return "Invalid password"  # Confirms user exists!

# GOOD - Consistent response
if not (user.exists() and user.password_matches()):
    return "Invalid credentials"  # Doesn't leak info
```

### 4. Security Headers
```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
```

### 5. Rate Limiting
```python
# Limit directory brute forcing
if requests_per_minute > 100:
    block_ip(ip_address, duration=15_minutes)
```

### 6. Web Application Firewall (WAF)
- Detects and blocks scanning tools
- Identifies suspicious patterns (100+ 404s in 1 minute)
- Protects against common attacks

---

## Ethical and Legal Considerations

### ⚠️ CRITICAL: Only scan systems you own or have permission to test

**Legal to scan:**
- ✅ Your own websites and applications
- ✅ CTF challenges and practice labs (like this one!)
- ✅ Bug bounty programs with clear scope
- ✅ Client systems with signed penetration testing agreement

**ILLEGAL to scan:**
- ❌ Any website you don't own
- ❌ Company systems without authorization
- ❌ "Testing" a site to "help them" without permission
- ❌ Scanning during a job interview to "show skills"

**Legal Consequences:**
- **Computer Fraud and Abuse Act (CFAA):** Up to 10 years prison
- **State computer crime laws:** Additional penalties
- **Civil lawsuits:** Damages, legal fees

### The Golden Rule:
**"Can I scan this?"**
- If you have to ask → NO
- If you own it → YES
- If you have written permission → YES
- Otherwise → NO

---

## MITRE ATT&CK Mapping

**Technique:** T1595 - Active Scanning
- T1595.001 - Scanning IP Blocks
- T1595.002 - Vulnerability Scanning

**Tactic:** TA0043 - Reconnaissance

**Related Techniques:**
- T1590 - Gather Victim Network Information
- T1592 - Gather Victim Host Information
- T1593 - Search Open Websites/Domains

---

## In This Challenge

You'll learn to:
1. ✅ Use Gobuster for directory enumeration
2. ✅ Interpret HTTP status codes
3. ✅ Identify hidden endpoints and sensitive paths
4. ✅ Understand what attackers look for during recon
5. ✅ Think like a penetration tester (but only on authorized targets!)

**You'll perform the same reconnaissance Jack Frost did** - but on a safe, local test environment you control.

Ready to discover hidden endpoints? Let's start scanning!
