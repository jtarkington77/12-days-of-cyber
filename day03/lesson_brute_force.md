# Brute Force Attacks - Understanding the Technique

## What is a Brute Force Attack?

A **brute force attack** is a method of trying every possible combination or password from a list until the correct one is found. It's like trying every key on a keyring until one unlocks the door.

### Real-World Examples:
- **2012 LinkedIn Breach**: Attackers cracked millions of weak passwords using brute force
- **2016 DNC Hack**: Initial access gained through password spraying (a type of brute force)
- **Everyday**: Hackers attempt billions of brute force login attempts daily on internet-facing systems

---

## Types of Brute Force Attacks

### 1. Dictionary Attack
- Uses a **wordlist** of common passwords
- Much faster than trying every combination
- Effective because people use predictable passwords

**Common passwords that appear in every wordlist:**
- `password`, `123456`, `admin`, `welcome`
- `password123`, `qwerty`, `letmein`
- Seasonal: `Summer2024`, `Winter2023`

### 2. Credential Stuffing
- Uses username/password pairs leaked from other breaches
- "People reuse passwords, so if it worked on Site A, try it on Site B"

### 3. Password Spraying
- Tries a few common passwords against MANY usernames
- Avoids account lockouts (only 1-2 attempts per account)
- Very stealthy

### 4. Pure Brute Force
- Tries every possible character combination
- `a`, `aa`, `aaa`, `aab`, `aac`...
- Extremely slow, only works on short/simple passwords

---

## How Attackers Use Brute Force Tools

### The Most Common Tool: **Hydra**

Hydra is the industry-standard brute force tool. It can attack:
- HTTP login forms (web applications)
- SSH servers
- FTP servers
- RDP (Remote Desktop)
- And 50+ other protocols

**Why attackers love Hydra:**
- Fast: Tests hundreds of passwords per second
- Flexible: Works on many protocols
- Free and open-source
- Built into Kali Linux

---

## How Brute Force Attacks Work (Step-by-Step)

### Step 1: Reconnaissance
Attacker identifies:
- Target login page: `http://example.com/login`
- Username field name: `username`
- Password field name: `password`
- How the form submits (POST method, parameters)

### Step 2: Prepare the Attack
- Choose a **wordlist** (rockyou.txt is most popular - 14 million passwords)
- Decide on username (or use a username list)
- Configure the tool

### Step 3: Launch the Attack
```bash
hydra -l admin -P rockyou.txt example.com http-post-form "/login:username=^USER^&password=^PASS^:Invalid"
```

This command:
- `-l admin` = username is "admin"
- `-P rockyou.txt` = use this password wordlist
- `http-post-form` = attack HTTP login form
- `/login:username=^USER^&password=^PASS^:Invalid` = form details and failure message

### Step 4: Wait for Success
Hydra tries each password:
```
[ATTEMPT] target example.com - login "admin" - pass "password"
[ATTEMPT] target example.com - login "admin" - pass "123456"
[ATTEMPT] target example.com - login "admin" - pass "admin123"
[SUCCESS] login: admin   password: snowflake2024
```

---

## Detecting Brute Force Attacks

### How Defenders Spot Brute Force:

**1. Volume of Failed Logins**
- Normal: 1-2 failed attempts, then success
- Attack: 100+ failed attempts from same IP

**2. Speed of Attempts**
- Normal: Humans type, wait 5-10 seconds between attempts
- Attack: Automated tools try 10-50 passwords per second

**3. Sequential Usernames**
- Attack pattern: `admin`, `admin1`, `admin2`, `administrator`
- Tools try predictable username variations

**4. Common Passwords**
- Seeing attempts with `password`, `admin`, `123456` in logs
- Clear sign of dictionary attack

### Log Evidence (What You'll See):

```
Dec 14 02:42:15 Failed login: admin from 203.0.113.99
Dec 14 02:42:15 Failed login: admin from 203.0.113.99
Dec 14 02:42:16 Failed login: admin from 203.0.113.99
Dec 14 02:42:16 Failed login: admin from 203.0.113.99
[... 1,847 more attempts ...]
Dec 14 02:47:32 Successful login: admin from 203.0.113.99
```

**Red flags:**
- Same IP address
- Same username
- Rapid-fire attempts (seconds apart)
- Successful login after many failures

---

## Defending Against Brute Force

### 1. Account Lockout Policies
- Lock account after 5 failed attempts
- Require admin unlock or 15-minute wait

### 2. Rate Limiting
- Slow down login attempts (max 3 per minute)
- Temporary IP blocks after repeated failures

### 3. CAPTCHA
- "Prove you're human" after failed attempts
- Stops automated tools

### 4. Multi-Factor Authentication (MFA)
- Even if password is cracked, attacker needs 2nd factor
- **Most effective defense**

### 5. Strong Password Policies
- Minimum 12+ characters
- No common words from dictionaries
- Complexity requirements (uppercase, numbers, symbols)

### 6. Monitoring & Alerting
- Alert on 10+ failed logins from same IP
- SOC team investigates immediately

---

## MITRE ATT&CK Framework

**Technique:** T1110 - Brute Force

**Sub-techniques:**
- T1110.001 - Password Guessing
- T1110.002 - Password Cracking (offline)
- T1110.003 - Password Spraying
- T1110.004 - Credential Stuffing

**Tactic:** Credential Access (TA0006)

---

## Legal and Ethical Considerations

### ⚠️ IMPORTANT: Only attack systems you own or have written permission to test

**Legal to brute force:**
- Your own systems
- CTF challenges and practice labs
- Systems where you have a signed penetration testing agreement

**ILLEGAL to brute force:**
- Any system you don't own
- Company systems without authorization
- "Testing" a website without permission

**Penalties for unauthorized access:**
- Computer Fraud and Abuse Act (CFAA): Up to 10 years in prison
- State laws: Fines and criminal charges

---

## In This Challenge

You'll learn to:
1. ✅ Use Hydra to perform a dictionary attack
2. ✅ Understand how brute force tools work
3. ✅ Identify successful credential compromise
4. ✅ Follow the attack chain (brute force → access → find data)

This is the same technique Jack Frost used to compromise the Workshop Portal after stealing the username from Day 2's backup files.

**Next:** Let's use Hydra to brute force the Workshop Portal and see what Jack Frost accessed!
