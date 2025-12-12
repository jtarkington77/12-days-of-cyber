# Lesson: Password Hashing & Cracking - What, Why, and How

## What is a Password Hash?

Imagine you run a website. Users create accounts with passwords. **Question:** Where do you store those passwords?

**Bad answer:** Store them in plain text in a database.
- If someone hacks your database, they get everyone's passwords
- Employees with database access can see all passwords
- This is a disaster waiting to happen

**Good answer:** Store **password hashes** instead of the actual passwords.

### What is Hashing?

**Hashing** is a one-way mathematical function that converts any input into a fixed-length string of characters.

**Example:**
```
Password: "MyPassword123"
MD5 Hash: "6c569aabbf7775ef8fc570e228c16b98"
```

**Key properties of hashing:**
1. **One-way**: You can't reverse a hash to get the original password
2. **Deterministic**: Same input always produces the same hash
3. **Fixed-length**: Output is always the same length regardless of input
4. **Avalanche effect**: Tiny change in input completely changes the hash

```
"MyPassword123" → 6c569aabbf7775ef8fc570e228c16b98
"MyPassword124" → 8a474a23de48c07cd032d8a9c50253cc (completely different!)
```

---

## Why Use Password Hashes?

### How Authentication Works:

**Registration:**
1. User creates account with password "MySecretPass"
2. System hashes it: `5f4dcc3b5aa765d61d8327deb882cf99`
3. System stores the HASH in database (not the password)

**Login:**
1. User enters password "MySecretPass"
2. System hashes what they entered: `5f4dcc3b5aa765d61d8327deb882cf99`
3. System compares: does stored hash match new hash?
4. If yes → login successful
5. If no → wrong password

**Why this is good:**
- Database breach? Attacker gets hashes, not passwords
- System administrators can't see your actual password
- Even the company doesn't know your real password

---

## Common Hash Types

### MD5 (Message Digest 5)

**What it looks like:**
```
5f4dcc3b5aa765d61d8327deb882cf99
```

**Characteristics:**
- 32 hexadecimal characters
- Produces 128-bit hash
- Created in 1991

**Status:** ⚠️ **BROKEN** - Do not use for passwords!
- Very fast to compute (good for attackers)
- Collision attacks possible (two inputs produce same hash)
- Can crack billions of hashes per second with modern GPUs

**Where you still see it:**
- Legacy systems (pre-2010)
- File integrity checks (checksums)
- Old backup files (like in today's challenge!)

### SHA-1 (Secure Hash Algorithm 1)

**What it looks like:**
```
2fd4e1c67a2d28fced849ee1bb76e7391b93eb12
```

**Characteristics:**
- 40 hexadecimal characters
- Produces 160-bit hash
- Created by NSA in 1995

**Status:** ⚠️ **DEPRECATED** - Being phased out
- Collision attacks demonstrated in 2017
- Still faster to crack than modern algorithms
- Should not be used for new systems

**Where you still see it:**
- Git commit IDs
- Legacy authentication systems
- Old backups

---

## How Attackers Crack Password Hashes

Remember, hashing is one-way. You can't "decrypt" a hash. So attackers **guess passwords, hash each guess, and compare**.

### Attack Methods:

#### 1. Dictionary Attack
Try common words from a list (like `rockyou.txt`).

#### 2. Rule-Based Attack
Take dictionary words and apply patterns:
- "password" → "Password2023!"

#### 3. Custom Wordlists (Today's Focus)
If passwords follow a company pattern, create targeted lists.

---

## Tools for Password Cracking

### Hashcat (Recommended)
- GPU-accelerated, extremely fast
- Download: https://hashcat.net/hashcat/

**Basic syntax:**
```cmd
hashcat -m 0 -a 0 hashes.txt wordlist.txt
```
- `-m 0` = MD5
- `-m 100` = SHA1
- `-a 0` = dictionary attack

### John the Ripper
- Easier for beginners, auto-detects hash types
- Download: https://www.openwall.com/john/

---

## Why This Matters for Security

**Real scenario:** A company has old backup files with weak, unsalted password hashes. If these leak:
1. Attackers crack the hashes easily
2. Users often reuse passwords
3. Accounts in current systems get compromised

**Your job:** Identify these risks during incident response!

---

## Ready for the Challenge?

You now understand password hashing and cracking. Head back to README.md to investigate FrostNet's backup files!
