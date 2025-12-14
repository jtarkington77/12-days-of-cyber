# Lesson: Log Analysis & Brute Force Detection - What, Why, and How

## What Are Logs?

**Logs** are records of events that happen in computer systems. Think of them like a diary that computers keep - every action, error, or event gets written down with a timestamp.

**Example log entry:**
```
2024-12-14 02:15:43 authd[2451]: FAILED - User 'admin' from 203.0.113.47 - Invalid password
```

This single line tells us:
- **When**: December 14, 2024 at 02:15:43 AM
- **What**: Failed login attempt
- **Who**: Someone trying to log in as 'admin'
- **Where from**: IP address 203.0.113.47
- **Why it failed**: Invalid password

---

## Why Do Systems Keep Logs?

### 1. Security Monitoring
- Detect unauthorized access attempts
- Track suspicious behavior
- Investigate security incidents
- Evidence for incident response

### 2. Troubleshooting
- Debug application errors
- Track down system failures
- Monitor performance issues

### 3. Compliance
- Legal requirements (GDPR, HIPAA, PCI-DSS)
- Audit trails for financial systems
- Prove who accessed what and when

### 4. Forensics
- Reconstruct attack timelines
- Understand what the attacker did
- Identify compromised systems

---

## Common Types of Logs

### Authentication Logs
Records login attempts (successful and failed).

**Example:**
```
2024-12-14 09:15:22 SUCCESS - User 'mchen' from 10.0.1.45
2024-12-14 09:16:05 FAILED - User 'admin' from 203.0.113.47
```

**What to look for:**
- Failed login patterns
- Logins from unusual locations
- After-hours access
- Multiple failed attempts followed by success

### Web Server Logs (Access Logs)
Records every HTTP request to a web server.

**Example:**
```
203.0.113.47 - - [14/Dec/2024:02:15:43] "GET /admin/login HTTP/1.1" 200 1234
```

**What to look for:**
- Scanning activity (accessing many URLs rapidly)
- Suspicious user agents
- Access to sensitive endpoints
- SQL injection attempts in URLs

### Application Logs
Records events from specific applications.

**What to look for:**
- Error patterns
- Unusual behavior
- Performance degradation
- Security exceptions

### System Logs
Records operating system events.

**What to look for:**
- New user accounts created
- Scheduled tasks added
- Services started/stopped
- File system changes

---

## What is a Brute Force Attack?

**Brute force** is when an attacker tries many password guesses to break into an account.

### How It Works:

1. Attacker has a target username (like "admin")
2. They try common passwords:
   ```
   admin / password
   admin / 123456
   admin / admin
   admin / letmein
   admin / welcome
   ...thousands more...
   ```
3. Eventually, they might guess correctly

### Why It Works:

- Many people use weak, common passwords
- Automated tools can try thousands of passwords per minute
- If there's no rate limiting, nothing stops the attacker

### Types of Brute Force:

**1. Simple Brute Force**
- One username, many password guesses
- Easy to detect (many failures from same IP)

**2. Credential Stuffing**
- Using username:password pairs from previous breaches
- Higher success rate
- Harder to detect

**3. Password Spraying**
- Try one common password against many usernames
- Avoids account lockouts
- Example: Try "Welcome2024!" against 1000 usernames

---

## How to Detect Brute Force in Logs

### Red Flags:

#### 1. High Failure Rate
```
FAILED - User 'admin' from 203.0.113.47
FAILED - User 'admin' from 203.0.113.47
FAILED - User 'admin' from 203.0.113.47
FAILED - User 'admin' from 203.0.113.47
... 500 more times...
SUCCESS - User 'admin' from 203.0.113.47
```

**Pattern**: Many failures from same IP, same target user, then success.

#### 2. Timing Pattern
```
02:15:01 FAILED
02:15:03 FAILED
02:15:05 FAILED
02:15:07 FAILED
```

**Pattern**: Regular intervals (automated tool), usually very fast (seconds apart).

#### 3. Source IP
- Comes from unusual location (foreign country)
- Not in your company's IP ranges
- Known malicious IP

#### 4. Time of Day
- 3 AM when no legitimate users should be logging in
- Outside business hours
- Holidays/weekends

---

## Log Analysis Tools & Techniques

### Windows PowerShell

PowerShell is built into Windows and perfect for log analysis.

#### Basic Commands:

**Read a log file:**
```powershell
Get-Content auth.log
```

**Search for pattern:**
```powershell
Get-Content auth.log | Select-String "FAILED"
```

**Count occurrences:**
```powershell
(Get-Content auth.log | Select-String "FAILED").Count
```

**Filter by IP address:**
```powershell
Get-Content auth.log | Select-String "203.0.113.47"
```

**Extract and count IPs:**
```powershell
Get-Content auth.log | Select-String "FAILED" | ForEach-Object {
    if ($_ -match 'from (\d+\.\d+\.\d+\.\d+)') {
        $matches[1]
    }
} | Group-Object | Sort-Object Count -Descending
```

**Get first and last occurrence:**
```powershell
Get-Content auth.log | Select-String "attack" | Select-Object -First 1
Get-Content auth.log | Select-String "attack" | Select-Object -Last 1
```

### Alternative: findstr (Built into Windows)

**Basic search:**
```cmd
findstr "FAILED" auth.log
```

**Case-insensitive:**
```cmd
findstr /I "failed" auth.log
```

**Count matches:**
```cmd
findstr /C:"FAILED" auth.log | find /C /V ""
```

### Linux/WSL Commands

**Search logs:**
```bash
grep "FAILED" auth.log
```

**Count:**
```bash
grep -c "FAILED" auth.log
```

**Extract IPs and count:**
```bash
grep "FAILED" auth.log | grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | sort | uniq -c | sort -rn
```

---

## Real-World Example: SSH Brute Force

SSH servers (Linux remote access) are constantly under brute force attacks.

**Typical log entries:**
```
Dec 14 02:15:43 sshd[2451]: Failed password for root from 203.0.113.47 port 54892 ssh2
Dec 14 02:15:45 sshd[2453]: Failed password for root from 203.0.113.47 port 54893 ssh2
Dec 14 02:15:47 sshd[2455]: Failed password for root from 203.0.113.47 port 54894 ssh2
... (1000 more attempts) ...
Dec 14 03:42:19 sshd[3891]: Accepted password for root from 203.0.113.47 port 57234 ssh2
```

**Analysis:**
- Source: Single IP (203.0.113.47)
- Target: root account (high-value target)
- Duration: 02:15 - 03:42 (87 minutes)
- Attempts: 1000+ failures before success
- Result: **COMPROMISED**

**Defenses:**
- Rate limiting (block IP after 5 failures)
- Account lockout (disable account after failures)
- Fail2ban (automatically ban attacking IPs)
- Multi-factor authentication (password alone isn't enough)

---

## Why Log Analysis Matters for Security

### 1. Early Detection
- Catch attacks while they're happening
- Stop breaches before they succeed
- Alert SOC team in real-time

### 2. Incident Response
- Understand what happened
- Identify compromised accounts
- Determine attacker's actions

### 3. Threat Hunting
- Proactively search for hidden threats
- Find attacks that automated tools missed
- Discover persistence mechanisms

### 4. Forensics
- Build timeline of attack
- Legal evidence
- Understand attacker TTPs (tactics, techniques, procedures)

---

## Common Challenges in Log Analysis

### 1. Volume
- Large systems generate gigabytes of logs per day
- Millions of entries to sort through
- Needle in a haystack problem

**Solution:** Use filtering, search patterns, aggregate data

### 2. Noise
- Most log entries are normal activity
- False positives drown out real threats
- Hard to distinguish attack from legitimate failures (typos)

**Solution:** Baseline normal behavior, look for anomalies

### 3. Format Variations
- Different systems log differently
- Inconsistent timestamps
- Various formats (JSON, plaintext, syslog)

**Solution:** Learn to parse different formats, use standardized tools

### 4. Correlation
- Attacks span multiple systems
- Need to connect logs from different sources
- Timeline reconstruction

**Solution:** Use SIEM (Security Information and Event Management) tools

---

## Defending Against Brute Force

### Rate Limiting
```
Allow only 5 failed attempts per IP per minute
Block IP for 1 hour after exceeding limit
```

### Account Lockout
```
Lock account after 10 failed attempts
Require admin unlock or 30-minute wait
```

### CAPTCHA
```
After 3 failed attempts, require CAPTCHA
Makes automation difficult
```

### Multi-Factor Authentication (MFA)
```
Password + phone code
Even if password is cracked, attacker can't login
```

### IP Whitelisting
```
Only allow logins from known IP ranges
VPN required for remote access
```

### Monitoring & Alerting
```
Alert SOC when:
- 10+ failures from single IP
- Failures outside business hours
- Success after many failures
```

---

## Today's Challenge Context

In today's challenge, you'll analyze authentication logs from FrostNet's systems to:

1. **Find the brute force attack** - Among thousands of normal logins
2. **Identify the attacker's IP** - Where are they coming from?
3. **Determine the timeline** - When did it start/end?
4. **Find the compromised account** - Which account got breached?
5. **Extract evidence** - The flag is in a security alert log entry

**This is realistic incident response work.** You're doing what SOC analysts do every day!

---

## Key Takeaways

- Logs are essential for security monitoring and incident response
- Brute force attacks leave clear patterns in authentication logs
- PowerShell and grep are powerful tools for log analysis
- Look for: high failure rates, timing patterns, unusual IPs, odd hours
- Real-world attacks affect real organizations every day
- Your skills can help detect and stop these attacks

---

## Ready for the Challenge?

You now understand:
- What logs are and why they matter
- How brute force attacks work
- How to detect attacks in logs
- PowerShell commands for log analysis
- Why this is critical for security

Head back to README.md and start investigating those authentication logs!
