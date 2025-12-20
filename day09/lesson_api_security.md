# 📖 Lesson: API Security, Authorization, and Secret Management

## 🎯 Learning Objectives

By the end of this lesson, you'll understand:

- What APIs are and why they're critical attack targets
- The difference between authentication and authorization
- Common API security vulnerabilities (OWASP API Security Top 10)
- What secrets are and why they shouldn't be in source code
- How secrets leak through git repositories
- How to detect leaked secrets (GitHub Secret Scanning, TruffleHog, GitLeaks)
- How authorization bypass attacks work
- Real-world API security incidents and their impact
- How to test APIs safely and ethically

**This lesson assumes zero prior knowledge!** We'll start from the basics.

---

## 📚 Table of Contents

1. [What are APIs?](#1-what-are-apis)
2. [Authentication vs. Authorization](#2-authentication-vs-authorization)
3. [Common API Vulnerabilities](#3-common-api-vulnerabilities)
4. [Secret Management](#4-secret-management)
5. [Secrets in Git Repositories](#5-secrets-in-git-repositories)
6. [Secret Scanning Tools](#6-secret-scanning-tools)
7. [Authorization Bypass Attacks](#7-authorization-bypass-attacks)
8. [Real-World Incidents](#8-real-world-incidents)
9. [Defensive Strategies](#9-defensive-strategies)
10. [Ethical API Testing](#10-ethical-api-testing)

---

## 1. What are APIs?

### What is an API?

**API (Application Programming Interface)** is a way for software applications to communicate with each other.

**Simple analogy:** Think of an API like a restaurant:
- **You (client)** want food but don't know how to cook it
- **Menu (API documentation)** tells you what's available
- **Waiter (API)** takes your order to the kitchen
- **Kitchen (server)** prepares your food
- **Waiter** brings back your food (response)

You don't need to know how the kitchen works—you just order from the menu!

### Real-World API Examples

**You use APIs every day without knowing it:**

1. **Weather apps** - Call weather service APIs to get forecasts
2. **Google Maps** - Apps use Google's API to show maps
3. **Social media login** - "Sign in with Google" uses Google's authentication API
4. **Payment processing** - Stores use Stripe/PayPal APIs to process payments
5. **Messaging** - Apps use Twilio API to send SMS messages

### Types of APIs

**REST APIs (most common):**
- Use HTTP methods (GET, POST, PUT, DELETE)
- Return data in JSON format
- Example: `GET /api/users/123` → Get user with ID 123

**GraphQL APIs:**
- Query language for APIs
- Ask for exactly what you need
- Example: Used by Facebook, GitHub

**SOAP APIs (older):**
- XML-based protocol
- More rigid and formal
- Example: Banking systems

### Why Are APIs Attack Targets?

APIs are lucrative targets for attackers because:

1. **They expose data directly** - Often bypass security controls
2. **They're everywhere** - Modern apps rely on dozens of APIs
3. **They're complex** - Easy to misconfigure or make mistakes
4. **They're poorly documented** - Developers forget what's exposed
5. **They're automated** - Easy to attack at scale with scripts

**Statistics:**
- 83% of web traffic is API calls (Cloudflare, 2023)
- API attacks increased 400% from 2021-2023 (Akamai)
- 95% of organizations experienced an API security incident in 2023 (Salt Security)

---

## 2. Authentication vs. Authorization

### The Critical Difference

**This is THE most important concept in API security!**

Many developers (and attackers) confuse these two concepts:

### Authentication ("Who are you?")

**Authentication** proves your identity.

**Examples:**
- Logging in with username/password
- Providing an API key
- Showing your passport at the airport
- Scanning your employee badge

**Question answered:** "Are you who you claim to be?"

**Result:** You get an identity (user ID, session token, API key)

### Authorization ("What can you do?")

**Authorization** proves what you're allowed to do.

**Examples:**
- Admin users can delete accounts, regular users cannot
- Free tier users can call API 100 times/day, paid users get 10,000
- Employees can access building, but only managers can access server room
- You can edit your own profile, but not someone else's

**Question answered:** "Are you allowed to perform this action?"

**Result:** Access granted or denied based on permissions

### Authentication vs. Authorization Table

| Aspect | Authentication | Authorization |
|--------|---------------|---------------|
| **Question** | Who are you? | What can you do? |
| **Purpose** | Verify identity | Verify permissions |
| **Example** | Username + password | User role (admin, user, guest) |
| **When** | At login / first request | Before each action |
| **Technology** | Passwords, API keys, OAuth tokens | Roles, permissions, ACLs |
| **Failure** | "Invalid credentials" (401) | "Forbidden" (403) |

### Real-World Analogy

**Airport Security:**

1. **Authentication (TSA checkpoint):**
   - You show your boarding pass and ID
   - TSA verifies YOU are the person on the ticket
   - Question: "Is this person who they claim to be?"

2. **Authorization (Boarding gate):**
   - You're verified as a passenger, but...
   - First class boards first, economy boards later
   - You can't board someone else's flight even with valid ID!
   - Question: "Is this person allowed to board THIS flight NOW?"

### The Confusion That Causes Vulnerabilities

**Broken code example (like Day 9):**

```python
# WRONG! This only checks authentication, not authorization
def get_admin_data(api_key):
    user = validate_api_key(api_key)  # Authentication

    if user:  # ← BUG! Only checks if user exists, not if they're admin!
        return admin_secrets

    return "Unauthorized"
```

**Fixed code:**

```python
# CORRECT! Check BOTH authentication AND authorization
def get_admin_data(api_key):
    user = validate_api_key(api_key)  # Authentication

    if user and user.role == "administrator":  # Authorization too!
        return admin_secrets

    return "Forbidden"
```

**Key takeaway:** Just because someone is authenticated (valid user) doesn't mean they're authorized (allowed to do that action)!

---

## 3. Common API Vulnerabilities

### OWASP API Security Top 10 (2023)

The OWASP Foundation identifies the most critical API security risks:

#### API1:2023 - Broken Object Level Authorization (BOLA)

**What it is:** API doesn't check if the authenticated user is allowed to access the requested object.

**Example:**
```
GET /api/users/123/messages  ← You can see user 123's messages
GET /api/users/456/messages  ← Can you see user 456's messages too?
```

**Attack:** Change object IDs to access other users' data

**Impact:** Data breach, privacy violation

**Real-world:** Peloton API exposed private profiles (2021)

#### API2:2023 - Broken Authentication

**What it is:** Weak authentication mechanisms allow attackers to impersonate users.

**Examples:**
- Weak password requirements
- API keys transmitted in URLs (logged in server logs)
- No rate limiting on login attempts
- Predictable API key formats

**Attack:** Brute force, credential stuffing, token theft

**Impact:** Account takeover, unauthorized access

#### API3:2023 - Broken Object Property Level Authorization

**What it is:** API exposes more object properties than users should see.

**Example:**
```json
GET /api/users/me
{
  "name": "John",
  "email": "john@example.com",
  "password_hash": "5f4dcc3b5aa...",  ← Shouldn't be exposed!
  "is_admin": false,                 ← Shouldn't be exposed!
  "credit_card": "4111-1111-1111-1111"  ← Definitely shouldn't be exposed!
}
```

**Attack:** Harvest sensitive data from API responses

**Impact:** Data leakage, PII exposure

#### API4:2023 - Unrestricted Resource Consumption

**What it is:** No rate limiting or resource quotas.

**Example:**
- Unlimited API calls → DDoS
- No timeout on requests → Resource exhaustion
- Upload unlimited files → Disk space exhaustion

**Attack:** Denial of service, resource abuse

**Impact:** Service downtime, increased costs

#### API5:2023 - Broken Function Level Authorization

**What it is:** Regular users can access admin functions. **This is Day 9's vulnerability!**

**Example:**
```
Regular user:  GET  /api/users/profile      ✅ Allowed
Regular user:  POST /api/admin/delete_user  ❌ Should block, but doesn't!
```

**Attack:** Privilege escalation, unauthorized admin actions

**Impact:** Complete system compromise

**Real-world:** This is exactly what you're exploiting in Day 9!

#### API6:2023 - Unrestricted Access to Sensitive Business Flows

**What it is:** Automated abuse of legitimate workflows.

**Examples:**
- Bulk ticket purchasing (bots buying concert tickets)
- Cryptocurrency exchange market manipulation
- Automated posting (spam, fake reviews)

**Attack:** Business logic abuse, automation

**Impact:** Revenue loss, unfair advantage

#### API7:2023 - Server Side Request Forgery (SSRF)

**What it is:** API accepts URLs from users and fetches them server-side.

**Example:**
```
POST /api/fetch_image
{"url": "http://internal-database:5432"}  ← Attacker can scan internal network!
```

**Attack:** Internal network scanning, cloud metadata access

**Impact:** Internal system access, credential theft

#### API8:2023 - Security Misconfiguration

**What it is:** Insecure default configurations, unnecessary features enabled.

**Examples:**
- Debug mode enabled in production
- Verbose error messages revealing system details
- CORS misconfiguration allowing any origin
- Unpatched security vulnerabilities

**Attack:** Information disclosure, exploitation of known vulnerabilities

**Impact:** Varies (data leak to full compromise)

#### API9:2023 - Improper Inventory Management

**What it is:** Organizations don't know which APIs they have deployed.

**Examples:**
- Old API versions still running (v1 is insecure, v2 is fixed, but v1 still accessible)
- Forgotten test/debug endpoints in production
- Undocumented shadow APIs

**Attack:** Attacking forgotten, unpatched APIs

**Impact:** Backdoor access, unmonitored breaches

#### API10:2023 - Unsafe Consumption of APIs

**What it is:** Trusting third-party APIs without validation.

**Example:**
```python
# Trusting third-party API response without validation
response = third_party_api.get_user_data()
database.save(response.data)  ← What if response is malicious?
```

**Attack:** Supply chain attack, data poisoning

**Impact:** System compromise through trusted integrations

---

## 4. Secret Management

### What are Secrets?

**Secrets** are any sensitive credentials that grant access to systems:

- **API keys** - Tokens that authenticate API access
- **Passwords** - User or service account credentials
- **Private keys** - Cryptographic keys (SSH, TLS, PGP)
- **OAuth tokens** - Access tokens for third-party services
- **Database credentials** - Username/password for databases
- **Encryption keys** - Keys to decrypt data

### Why Secrets Need Protection

**If an attacker gets your secrets, they can:**
- Access your databases
- Call your APIs (at your expense)
- Read/modify data
- Impersonate your service
- Pivot to other systems

**Example impact:**
- **Leaked AWS key** → $50,000 bill from cryptocurrency mining
- **Leaked database password** → Entire customer database stolen
- **Leaked API key** → Service abuse, data breach, compliance violation

### Where Secrets SHOULD Be Stored

✅ **Good practices:**

1. **Environment variables**
   ```bash
   export API_KEY="secret-key-here"
   ```

2. **Secret management services**
   - AWS Secrets Manager
   - HashiCorp Vault
   - Azure Key Vault
   - Google Secret Manager

3. **.env files (for local development)**
   ```
   # .env file (add to .gitignore!)
   API_KEY=secret-key-here
   DB_PASSWORD=super-secret
   ```

4. **Configuration management**
   - Kubernetes Secrets
   - Docker Secrets
   - Ansible Vault

### Where Secrets SHOULD NOT Be Stored

❌ **Bad practices:**

1. **Hardcoded in source code**
   ```python
   API_KEY = "sk-1234567890abcdef"  # ← NEVER DO THIS!
   ```

2. **Configuration files committed to git**
   ```python
   # config.py (committed to git)
   DATABASE_PASSWORD = "admin123"  # ← BAD!
   ```

3. **In URLs or logs**
   ```
   GET /api/data?api_key=secret123  ← Logged in web server logs!
   ```

4. **In client-side code (JavaScript, mobile apps)**
   ```javascript
   const API_KEY = "secret123";  // ← Anyone can read this!
   ```

5. **In Docker images**
   ```dockerfile
   ENV API_KEY="secret123"  # ← Visible to anyone with the image!
   ```

---

## 5. Secrets in Git Repositories

### How Secrets Leak Through Git

**The problem:** Git is designed to preserve history forever.

**What developers do (accidentally):**

1. **Commit secrets in initial code**
   ```bash
   # Day 1: Create config.py with hardcoded API key
   echo 'API_KEY = "secret123"' > config.py
   git add config.py
   git commit -m "Initial commit"
   git push
   ```

2. **Realize mistake and "fix" it**
   ```bash
   # Day 2: Oh no! Remove the API key
   echo 'API_KEY = os.environ["API_KEY"]' > config.py
   git add config.py
   git commit -m "Fix: Move API key to environment variable"
   git push
   ```

3. **Think the problem is solved** ❌

**The reality:** The secret is STILL in git history!

```bash
# Anyone can view the old commit
git log  # Shows: commit abc123 "Initial commit"
git show abc123  # Shows: API_KEY = "secret123"
```

### Why This is Dangerous

**Even after removing the secret:**

1. **Git history is permanent** - Old commits are never deleted
2. **Anyone who cloned the repo has the history** - Can't "unsend" it
3. **Public repos are indexed** - Search engines, GitHub crawlers
4. **Attackers scan for secrets** - Automated tools search all of GitHub

**Statistics:**
- 2+ million secrets leaked on GitHub in 2022 (GitGuardian)
- Average time to discovery: 30 seconds after pushing
- Automated bots scan new commits instantly

### Git History Deep Dive

**Example scenario (like Day 9):**

```bash
# Commit 1 (Dec 12): Add config with secret
git show commit-1
  config.py:
    API_KEY = "NPGIFT-4c7e8a9b2f1d6e3a"  ← SECRET LEAKED!

# Commit 2 (Dec 15): Add features
git show commit-2
  service.py:
    # New features added
    # config.py unchanged

# Commit 3 (Dec 19): Remove secret from config
git show commit-3
  config.py:
    API_KEY = os.environ.get("API_KEY")  ← Fixed now!
```

**But attacker can access any commit:**

```bash
git checkout commit-1  # Go back to original commit
cat config.py          # Read the secret!
```

**Even worse with public repos:**

```bash
git clone https://github.com/northpole/gift-service.git
git log -p | grep -i "api_key"  # Search all history for "api_key"
```

### How Attackers Find Leaked Secrets

**Automated tools scan GitHub:**

1. **truffleHog** - Scans git history for secrets
2. **GitLeaks** - Detects hardcoded secrets
3. **GitRob** - Finds sensitive files in GitHub orgs
4. **GitHub's Secret Scanning** - Built-in GitHub feature
5. **Custom bots** - Attackers run 24/7 scanning new commits

**These tools can:**
- Scan thousands of repos per second
- Detect 200+ types of secrets
- Alert within seconds of a commit
- Search entire git history automatically

---

## 6. Secret Scanning Tools

### GitHub Secret Scanning (Built-in)

**What it is:** GitHub automatically scans public repositories for known secret patterns.

**How it works:**
1. You push a commit containing a secret
2. GitHub's scanner detects the pattern (e.g., AWS access key)
3. GitHub alerts you via email/notification
4. GitHub notifies the secret's provider (e.g., AWS)
5. Provider may automatically revoke the secret

**Supported secrets:**
- AWS access keys
- Azure tokens
- Google Cloud keys
- GitHub personal access tokens
- Stripe API keys
- Twilio tokens
- 200+ patterns from 180+ service providers

**Limitations:**
- Only scans public repos (unless you have GitHub Advanced Security)
- Only detects known patterns
- Won't catch custom API key formats

### TruffleHog

**What it is:** Open-source tool to scan git history for secrets.

**Features:**
- Scans git commit history
- Detects entropy (randomness) to find secrets
- Supports custom regex patterns
- Can scan local and remote repos

**Usage:**
```bash
# Scan a repository
trufflehog git https://github.com/user/repo

# Scan local directory
trufflehog filesystem /path/to/repo

# Output JSON format
trufflehog git https://github.com/user/repo --json
```

### GitLeaks

**What it is:** Tool to detect hardcoded secrets in git repositories.

**Features:**
- Fast scanning engine
- Customizable rules
- Integrates with CI/CD pipelines
- Pre-commit hooks to prevent secrets

**Usage:**
```bash
# Scan repository
gitleaks detect --source /path/to/repo

# Scan uncommitted changes
gitleaks protect --staged

# Custom config
gitleaks detect --config gitleaks.toml
```

### Pre-commit Hooks

**Prevent secrets from being committed in the first place!**

**Install git hooks:**

```bash
# Install detect-secrets
pip install detect-secrets

# Set up pre-commit hook
echo 'detect-secrets scan --baseline .secrets.baseline' > .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

**Now when you try to commit:**
```bash
git commit -m "Add config"
❌ ERROR: Potential secret detected in config.py line 15
Commit aborted!
```

---

## 7. Authorization Bypass Attacks

### What is Authorization Bypass?

**Authorization bypass** is when an attacker gains access to resources or actions they shouldn't be allowed to access.

**Examples:**
- Regular user accesses admin panel
- User A views/edits User B's data
- Free tier user accesses paid features
- Service account accesses admin endpoints ← **This is Day 9!**

### Common Authorization Bypass Patterns

#### Pattern 1: Missing Authorization Check

**Vulnerable code:**
```python
@app.route('/api/admin/users')
def get_all_users():
    # Checks authentication but NOT authorization!
    if not current_user:
        return "Unauthorized", 401

    return jsonify(all_users)  # ← Any authenticated user can access!
```

**Fixed code:**
```python
@app.route('/api/admin/users')
def get_all_users():
    if not current_user:
        return "Unauthorized", 401

    if current_user.role != "admin":  # ← Added authorization check!
        return "Forbidden", 403

    return jsonify(all_users)
```

#### Pattern 2: Client-Side Authorization

**Vulnerable code:**
```javascript
// Frontend JavaScript (user can see and modify this!)
if (user.role === "admin") {
    showAdminPanel();  // ← Hiding UI doesn't prevent API access!
}
```

**Attack:**
```bash
# Attacker bypasses UI and calls API directly
curl -H "Authorization: Bearer user_token" https://api.example.com/admin/delete_user
```

**Fix:** Always check authorization on the server, never trust the client!

#### Pattern 3: Insecure Direct Object Reference (IDOR)

**Vulnerable code:**
```python
@app.route('/api/messages/<message_id>')
def get_message(message_id):
    message = database.get_message(message_id)
    return jsonify(message)  # ← Doesn't check if user owns this message!
```

**Attack:**
```bash
# User A (ID: 123)
GET /api/messages/456  ← Can read User B's messages!
GET /api/messages/789  ← Can read User C's messages!
```

**Fixed code:**
```python
@app.route('/api/messages/<message_id>')
def get_message(message_id):
    message = database.get_message(message_id)

    if message.owner_id != current_user.id:  # ← Check ownership!
        return "Forbidden", 403

    return jsonify(message)
```

#### Pattern 4: Role Confusion (Day 9's Vulnerability!)

**Vulnerable code:**
```python
@app.route('/api/admin/secrets')
def admin_secrets():
    api_key = request.headers.get('X-API-Key')
    user_info = validate_api_key(api_key)

    # BUG: Only checks if user exists (authentication)
    # Should also check if user is admin (authorization)
    if user_info.get("active"):  # ← Wrong check!
        return admin_data
```

**Attack:**
```bash
# Attacker uses service account key (not admin)
curl -H "X-API-Key: NPGIFT-4c7e8a9b2f1d6e3a" http://localhost:5001/api/admin/secrets
# Response: Admin data returned! Authorization bypassed!
```

**Fixed code:**
```python
@app.route('/api/admin/secrets')
def admin_secrets():
    api_key = request.headers.get('X-API-Key')
    user_info = validate_api_key(api_key)

    if not user_info:
        return "Unauthorized", 401

    if user_info.get("role") != "administrator":  # ← Correct check!
        return "Forbidden", 403

    return admin_data
```

### Testing for Authorization Bypass

**Methodology:**

1. **Identify protected resources**
   - Admin panels
   - User data endpoints
   - Sensitive configuration APIs

2. **Test with different privilege levels**
   - Create multiple test accounts (admin, user, guest)
   - Try accessing admin resources with user credentials
   - Try accessing User A's data with User B's credentials

3. **Manipulate identifiers**
   - Change user IDs in URLs
   - Change object IDs in requests
   - Test sequential ID guessing

4. **Test HTTP methods**
   - If GET is protected, try POST, PUT, DELETE
   - Some endpoints protect GET but not POST!

5. **Check response codes**
   - 200 OK = Access granted (might be bypass!)
   - 401 Unauthorized = Authentication required
   - 403 Forbidden = Authenticated but not authorized (correct!)

---

## 8. Real-World Incidents

### Uber (2022) - Hardcoded Credentials

**What happened:**
- Contractor's hardcoded credentials leaked on GitHub
- Attacker found credentials in public repository
- Used credentials to access internal systems
- Escalated to full network compromise

**Impact:**
- Complete breach of internal systems
- Access to source code, databases, employee data
- Estimated cost: $100+ million

**Lesson:** Never hardcode credentials, scan repositories for secrets

### Toyota (2022) - API Keys Exposed for 10 Years

**What happened:**
- API key for Toyota Connected service exposed on GitHub
- Key was public for 10 years (2012-2022)
- Allowed access to vehicle location data

**Impact:**
- 296,019 customers affected
- 10 years of vehicle location data exposed
- Regulatory fines and reputation damage

**Lesson:** Secrets in git history persist forever, regular security audits are critical

### Codecov (2021) - Leaked Credentials in CI/CD

**What happened:**
- Codecov's Docker image contained hardcoded credentials
- Attackers modified Bash Uploader script
- Harvested secrets from 29,000 customers' CI/CD pipelines

**Impact:**
- 29,000 organizations affected
- Customer credentials, tokens, API keys stolen
- Supply chain attack impacting thousands of companies

**Lesson:** Secrets in containers are visible, use secret management services

### Peloton (2021) - Broken Object Level Authorization

**What happened:**
- Peloton API didn't check user ID authorization
- Any authenticated user could access any other user's data
- User IDs were sequential and predictable

**Attack:**
```bash
GET /api/user/123/profile  # Your profile
GET /api/user/124/profile  # Someone else's profile (worked!)
GET /api/user/125/profile  # Another user (worked!)
```

**Impact:**
- All user data accessible (names, ages, locations, workout stats)
- Privacy violation for millions of users

**Lesson:** Always check authorization for object access (BOLA/IDOR)

### T-Mobile (2021) - API Authorization Bypass

**What happened:**
- API endpoint didn't properly check customer authorization
- Attackers could access any customer's account data
- Affected 54 million customers

**Impact:**
- Names, dates of birth, SSN, driver's licenses stolen
- $350 million settlement
- Multi-year reputation damage

**Lesson:** API authorization must be tested thoroughly

---

## 9. Defensive Strategies

### Secure Secret Management

✅ **Best Practices:**

1. **Never commit secrets to git**
   - Use .gitignore for .env files
   - Set up pre-commit hooks
   - Scan repositories regularly

2. **Use environment variables**
   ```python
   import os
   API_KEY = os.environ.get("API_KEY")
   if not API_KEY:
       raise ValueError("API_KEY must be set!")
   ```

3. **Use secret management services**
   - AWS Secrets Manager
   - HashiCorp Vault
   - Azure Key Vault

4. **Rotate secrets regularly**
   - Change secrets every 90 days
   - Immediately rotate if leaked

5. **Monitor for leaks**
   - Enable GitHub Secret Scanning
   - Use TruffleHog in CI/CD
   - Set up alerts for leaked credentials

### Secure API Authorization

✅ **Best Practices:**

1. **Always check authorization**
   ```python
   # Good pattern
   if not authenticated:
       return 401  # Unauthorized
   if not authorized:
       return 403  # Forbidden
   return data
   ```

2. **Use role-based access control (RBAC)**
   ```python
   @require_role("admin")
   def admin_endpoint():
       return admin_data
   ```

3. **Validate object ownership**
   ```python
   if object.owner_id != current_user.id:
       return 403
   ```

4. **Use allowlists, not denylists**
   ```python
   # Good: Default deny
   allowed_roles = ["admin", "superuser"]
   if user.role not in allowed_roles:
       return 403
   ```

5. **Test with multiple user roles**
   - Create test accounts for each role
   - Verify separation of privileges
   - Automated testing for authorization

### API Security Testing

✅ **Testing Checklist:**

- [ ] Authentication works correctly
- [ ] Authorization checked for all endpoints
- [ ] Different roles have different permissions
- [ ] Users can't access other users' data
- [ ] Admin endpoints reject non-admin users
- [ ] Rate limiting prevents brute force
- [ ] Input validation prevents injection
- [ ] Errors don't leak sensitive info
- [ ] Logs don't contain secrets
- [ ] HTTPS enforced (no HTTP)

### Security Monitoring

✅ **Monitoring Setup:**

1. **Log all API access**
   ```python
   log.info(f"User {user_id} accessed {endpoint} from {ip_address}")
   ```

2. **Alert on suspicious patterns**
   - Multiple 403 errors (authorization testing)
   - Sequential ID enumeration
   - Unusual API call volume

3. **Regular security audits**
   - Quarterly penetration testing
   - Annual security reviews
   - Continuous secret scanning

---

## 10. Ethical API Testing

### Legal and Ethical Considerations

**⚠️ CRITICAL: Only test APIs you own or have explicit permission to test!**

**Legal:**
- Unauthorized API testing may violate computer fraud laws
- Even "harmless" testing can be prosecuted
- Always get written permission (bug bounty, penetration testing contract)

**Ethical:**
- Don't cause harm (DoS, data deletion, PII exposure)
- Don't access data you don't own
- Report vulnerabilities responsibly
- Don't publicly disclose before patch is available

### Where You CAN Test Legally

✅ **Safe testing environments:**

1. **Your own APIs and systems**
2. **Educational challenges (like Day 9!)**
3. **Bug bounty programs:**
   - HackerOne
   - Bugcrowd
   - Intigriti
   - Synack
4. **Capture The Flag (CTF) competitions**
5. **Penetration testing with written authorization**

### Responsible Disclosure

**If you find a real vulnerability:**

1. **Don't exploit it** - Only test enough to confirm
2. **Document the issue** - Steps to reproduce, impact
3. **Contact the vendor privately** - Email security@company.com
4. **Give time to fix** - 90 days is standard
5. **Don't publicly disclose** - Until patch is available
6. **Accept bug bounty** - If offered (you deserve it!)

### Testing Methodology (Ethical)

**How to test APIs safely:**

1. **Start with reconnaissance**
   - Read API documentation
   - Use developer tools to see API calls
   - Map endpoints and parameters

2. **Test authentication**
   - Try accessing without credentials
   - Test with invalid credentials
   - Check for weak credential requirements

3. **Test authorization**
   - Access other users' data
   - Try admin endpoints as regular user
   - Test role separation

4. **Test input validation**
   - SQL injection in parameters
   - XSS in API responses
   - Command injection in inputs

5. **Test business logic**
   - Negative numbers (withdraw -$100 = deposit?)
   - Race conditions (parallel requests)
   - Workflow bypass

6. **Document findings**
   - Proof of concept (PoC)
   - Impact assessment
   - Remediation recommendations

---

## 📚 Additional Resources

### Official Documentation

- **OWASP API Security Top 10:** https://owasp.org/www-project-api-security/
- **GitHub Secret Scanning:** https://docs.github.com/en/code-security/secret-scanning
- **Flask Security:** https://flask.palletsprojects.com/en/latest/security/
- **REST API Security:** https://restfulapi.net/security-essentials/

### Tools

- **TruffleHog:** https://github.com/trufflesecurity/trufflehog
- **GitLeaks:** https://github.com/gitleaks/gitleaks
- **detect-secrets:** https://github.com/Yelp/detect-secrets
- **Postman:** https://www.postman.com/ (API testing)
- **Burp Suite:** https://portswigger.net/ (Web security testing)

### Books

- *"Web Application Security" by Andrew Hoffman*
- *"The Web Application Hacker's Handbook" by Stuttard & Pinto*
- *"API Security in Action" by Neil Madden*

### Practice Platforms

- **PortSwigger Web Security Academy** (free!)
- **HackTheBox** (API challenges)
- **TryHackMe** (API security rooms)
- **OWASP Juice Shop** (vulnerable web app)

---

## ✅ Key Takeaways

1. **APIs are critical attack targets** - They expose data and functionality directly

2. **Authentication ≠ Authorization**
   - Authentication: Who are you?
   - Authorization: What can you do?
   - Both are required!

3. **Secrets don't belong in git** - Ever. Not even in private repos.

4. **Git history is permanent** - Removing secrets doesn't delete them from history

5. **Authorization bypass is common** - Always check permissions, not just identity

6. **Test your APIs thoroughly** - Different users, different roles, edge cases

7. **Monitor and log everything** - Detect attacks early

8. **Use proper secret management** - Environment variables, secret vaults, rotation

9. **Only test what you own** - Or have explicit written permission

10. **Defense in depth** - Multiple layers of security, not just one check

---

## 🎓 Quiz Yourself

Test your understanding:

1. What's the difference between authentication and authorization?
2. Why is a hardcoded API key in git history still a risk after removing it?
3. What HTTP status code indicates "authenticated but not authorized"?
4. Name three places secrets should NOT be stored.
5. What is BOLA and why is it dangerous?
6. How can you prevent secrets from being committed to git?
7. What should you check before granting access to an admin endpoint?
8. Name three real-world companies affected by API security issues.

**Answers:**
1. Authentication = identity verification, Authorization = permission verification
2. Git preserves history forever; old commits still contain the secret
3. 403 Forbidden
4. Source code, git repos, URLs, client-side code, Docker images, logs
5. Broken Object Level Authorization - accessing objects you don't own
6. Pre-commit hooks, .gitignore, secret scanners, code review
7. Both authentication (valid user) AND authorization (admin role)
8. Uber, Toyota, Codecov, Peloton, T-Mobile (any three)

---

**Ready to put this knowledge into practice? Head to the Day 9 challenge and exploit an authorization bypass vulnerability!** 🎄
