[← Previous Day](../day06/README.md) | [Main README](../README.md) | **Day 7** | [Next Day →](../day08/README.md)

---

![Day 7](../Images/Day07.png)

# 🎄 Day 7 (December 18) - Stealing the Nice List

## 🎅 The Story

**December 18, 2024 - 02:47 AM (North Pole Time)**

Tom Icicle is dreaming about tropical beaches and warm sand—anywhere but the frozen North Pole—when his phone erupts with alerts. Not just one or two. **Forty-three consecutive alerts.**

He jolts awake, snow-themed pajamas tangled around him, and grabs his secure laptop. The North Pole Security Operations Center dashboard is lit up like a Christmas tree, but not in a good way.

**ALERT: Unusual DNS Query Volume - External Resolver**
**ALERT: Suspicious Domain Pattern Detected**
**ALERT: Data Exfiltration Suspected - DNS Protocol**

"No no no no no," Tom mutters, his fingers flying across the keyboard. He pulls up the network traffic logs from the past 24 hours. The graphs tell a disturbing story: massive spikes in DNS queries to external DNS servers, all from the **Nice List Database Server**.

His phone rings. It's Merry Tinselcode.

"Tom, please tell me you're seeing what I'm seeing," Merry's voice is tight with stress.

"Abnormal DNS traffic from the Nice List server? Yeah, I see it." Tom scrolls through thousands of DNS queries. "These domain names... they don't make sense. Look at this: `n5xw4zdpnzxw4.frostnet-cdn.com`. That's not a legitimate subdomain."

"How long has this been going on?" Merry asks.

Tom checks the timestamps. "First unusual query at **11:34 PM yesterday**. It's been running for over three hours. Merry, I think Jack Frost is using DNS tunneling to steal the Nice List."

There's a sharp intake of breath on the other end. "The *entire* Nice List? That's 2.7 million children's names, addresses, gift preferences, behavioral assessments..."

"And it's all encoded in those DNS queries," Tom confirms grimly. "He's using our own DNS infrastructure to smuggle data out. It bypasses our firewalls because DNS traffic is allowed through port 53. It's brilliant and terrifying."

Dekker Frostbeard's gruff voice cuts into the call—he's been listening. "Tom, capture that network traffic. Everything. I want a full PCAP file of the last 24 hours from the Nice List server. If Jack Frost is tunneling data through DNS, we need to extract it and see what he got."

"Already on it," Tom replies, initiating a full packet capture. "The PCAP file is going to be huge, but I can filter it down to just the DNS traffic."

Merry's typing sounds echo through the call. "Tom, I'm looking at our DNS query logs. The queries are going to `frostnet-cdn.com` and `north-pole-cache.net`. Both domains resolve to **203.0.113.99**."

Tom's blood runs cold. "That's Jack Frost's IP subnet. Same infrastructure from the previous attacks."

"He's been inside our network for a week," Dekker growls. "Day 3 he brute-forced admin access. Day 5 he dropped malware. Day 6 he mapped our web services. And now Day 7—he's stealing our most valuable asset."

Tom watches the packet capture complete. "I've got the PCAP. Twenty-four hours of network traffic, filtered to DNS queries. If the Nice List is in there, we can extract it. But we need to decode the DNS tunneling first."

"How does DNS tunneling work?" Dekker asks.

"Attackers encode data—like files, passwords, or in this case, the Nice List—into DNS queries," Tom explains. "Instead of sending data directly, they split it into small chunks and embed it in subdomain names. Each DNS query looks up a different subdomain, and the attacker's DNS server collects all the pieces and reassembles the data."

Merry jumps in. "So when we see a query for `n5xw4zdpnzxw4.frostnet-cdn.com`, that `n5xw4zdpnzxw4` part is actually *encoded data*?"

"Exactly. Usually Base32 or Base64 encoding. The attacker's DNS server receives the query, extracts the encoded data, decodes it, and reconstructs the stolen file."

"Can we reverse it?" Dekker demands.

"Yes," Tom says with determination. "If we analyze the PCAP file, extract all the DNS queries, pull out the encoded subdomains, decode them, and reassemble the data, we can see exactly what Jack Frost stole."

"Then do it," Dekker orders. "And Tom? We're seven days away from Christmas. If Jack Frost has the Nice List, he could sabotage Christmas entirely. Find out what he took."

Tom cracks his knuckles and opens Wireshark. The PCAP file loads, revealing thousands of DNS queries scrolling past. Hidden in those queries is the Nice List—and the key to stopping Jack Frost.

"Time to go fishing for some DNS tunnels," Tom mutters, diving into the packet capture.

---

## 🛠️ Prerequisites & Setup

**This is your first time using Wireshark!** Don't worry - we'll guide you through everything.

### Tools You'll Need

For this challenge, you'll need **ONE** of the following:

**Option 1: Pre-extracted DNS Queries (Recommended - No installation required!)**
- **File:** `dns_queries_list.txt` - DNS queries already extracted from PCAP
- **File:** `dns_subdomains_only.txt` - Just the base32-encoded subdomains
- **Best for:** Quick start, students without Wireshark, or troubleshooting
- **Skills taught:** DNS tunneling concepts, base32 decoding, data reconstruction
- **Jump to:** Section "Quick Start with Pre-extracted Queries" below

**Option 2: Wireshark (Full PCAP analysis experience)**
- **Windows/Mac:** Download from https://www.wireshark.org/
- **Linux:** `sudo apt install wireshark`
- **Includes:** GUI interface + tshark (command-line version)
- **First time using Wireshark?** → See `artifacts/wireshark_quickstart.md` for a complete beginner guide!

**Option 2: tshark (Command-line only)**
- Installed with Wireshark
- **Linux:** `sudo apt install tshark`
- Faster for scripting, but less visual

**Option 3: Python Script (parse_pcap.py)**
- Requires Python 3 and scapy library
- **Install scapy:**
  - **Windows:** `pip install scapy`
  - **Linux/Mac:** `pip3 install scapy` or `sudo apt install python3-scapy`
- **Usage:** `python parse_pcap.py` (in artifacts directory)
- Automatically extracts all DNS queries for you

**⚠️ IMPORTANT:** You must install at least ONE of these tools before starting!

### Additional Tools

You'll also use tools from previous days:
- **Base32 decoder** - CyberChef (Day 1) OR `base32` command (Linux/Mac)
- **Text editor** - For viewing extracted data

### Getting Started Checklist

Before you begin:
- [ ] Extract `day07_artifacts.zip` using Day 6's flag as password
- [ ] Install Wireshark (see links above)
- [ ] Open `artifacts/wireshark_quickstart.md` for a beginner tutorial
- [ ] Verify you can open `artifacts/nice_list_traffic.pcap` in Wireshark

**Estimated setup time:** 5-10 minutes (if installing Wireshark for first time)

---

## 🚀 Quick Start with Pre-extracted Queries (Option 1)

**NEW! No Wireshark required!** We've already extracted the DNS queries from the PCAP file for you.

### Step 1: View the DNS Queries

```bash
cd artifacts/
cat dns_queries_list.txt
```

You'll see queries like:
```
HU6T2PJ5HU6T2PJ5HU6T2PJ5HU6T2PJ5HU6T2PJ5HU6T2.frostnet-cdn.com
PJ5HU6T2PJ5HU6T2PIKJZHVEVCIEBIE6TCFEBJUKQ2VKJ.frostnet-cdn.com
...
```

### Step 2: Extract the Subdomains

The encoded data is in the subdomain (before the first `.`):

```bash
cat dns_subdomains_only.txt
```

### Step 3: Decode Base32

Concatenate all subdomains and decode:

**Linux/Mac/WSL:**
```bash
cat dns_subdomains_only.txt | tr -d '\n' | base32 -d
```

**Windows PowerShell:**
```powershell
$data = Get-Content dns_subdomains_only.txt -Raw
$data = $data -replace "`r`n",""
$data = $data -replace "`n",""
[System.Text.Encoding]::UTF8.GetString([Convert]::FromBase32String($data))
```

**CyberChef (any platform):**
1. Go to https://gchq.github.io/CyberChef/
2. Copy all content from `dns_subdomains_only.txt`
3. Paste into Input
4. Add "From Base32" recipe
5. See the decoded message with the flag!

### Step 4: Find the Flag

Look for `FROST{...}` in the decoded output. That's your flag for Day 8!

---

## 📚 Lesson: Learn the Technique

**Before starting this challenge, read the lesson:**
📖 [DNS Tunneling & Network Forensics - What, Why, and How](./lesson_dns_tunneling.md)

This lesson covers:
- What network forensics is and why it's critical
- How DNS works and why attackers abuse it
- What DNS tunneling is and how it bypasses firewalls
- How to analyze PCAP files with Wireshark and command-line tools
- Detecting and decoding DNS tunneling exfiltration
- Real-world case studies and defensive strategies

**New to network forensics?** Start with the lesson - it assumes zero prior knowledge and walks you through PCAP analysis step-by-step!

**First time using Wireshark?** → Check `artifacts/wireshark_quickstart.md` for installation and basic usage!

---

## 🎯 The Challenge

You are **Tom Icicle**, Incident Response Lead. Your mission:

1. Analyze the network packet capture (PCAP) file from the Nice List server
2. **Use your Day 1 skills** - You'll need to decode Base32/Base64 encoded data
3. **Use your Day 2 skills** - Recognize patterns in the DNS queries
4. **Use your Day 3 skills** - Analyze logs to find the attack timeline
5. **Use your Day 6 skills** - Understand how Jack Frost used reconnaissance data
6. Identify DNS tunneling indicators
7. Extract the encoded data from DNS queries
8. Decode the exfiltrated data
9. Find the flag hidden in the stolen Nice List data

**This is your most complex investigation yet, combining skills from all previous days!**

---

## 🔍 Artifacts

**Password required:** Use Day 6's flag to extract `day07_artifacts.zip`

After extracting, the `artifacts/` folder contains:

- **`dns_queries_list.txt`** - ✨ **NEW! Pre-extracted DNS queries** ← **Start here if you don't have Wireshark!**
- **`dns_subdomains_only.txt`** - ✨ **NEW! Just the base32-encoded subdomains** ← **Ready to decode!**
- **`nice_list_traffic.pcap`** - 24-hour network capture from the Nice List server (for Wireshark users)
- **`parse_pcap.py`** - Python script to extract DNS queries (requires scapy: `pip install scapy`)
- **`wireshark_quickstart.md`** - Complete Wireshark beginner guide (installation, filters, usage)
- **`dns_baseline.txt`** - Normal DNS query patterns for comparison (helps identify anomalies)
- **`dns_tunneling_indicators.txt`** - Signs of DNS tunneling to look for (reference guide)

### Quick Start Steps

1. **Extract the ZIP** with Day 6's flag as password
2. **Install analysis tools** (choose ONE):
   - Wireshark: `sudo apt install wireshark` (Linux) or download from wireshark.org
   - Python + scapy: `pip install scapy` then use `parse_pcap.py`
   - tshark: `sudo apt install tshark` (command-line only)
3. **Analyze the PCAP** and extract DNS queries
4. **Decode Base32** encoded subdomains
5. **Find the flag** in the decoded data!

---

## 🎁 Your Mission

**Analyze the DNS tunneling attack and extract the stolen data to find the flag.**

The flag is hidden in the exfiltrated Nice List data that Jack Frost tunneled through DNS queries. You'll need to:
- Filter the PCAP for DNS queries
- Identify suspicious domain patterns
- Extract the encoded subdomains
- Decode the Base32 data
- Reassemble the exfiltrated file
- Find the flag in the decoded output

---

## 🛠️ Tool Options: Choose Your Approach

**You have TWO ways to analyze the PCAP file - pick the one that works best for you!**

### Option A: Wireshark/tshark (Recommended - Learn More!)

**Best for:** Learning professional network forensics tools used in real SOC teams

**Pros:**
- Industry-standard tool used by security professionals
- Visual packet inspection and filtering
- More educational - you'll learn Wireshark skills
- Can analyze any PCAP file (useful beyond this challenge)

**Cons:**
- ~200MB download + installation required
- Requires GUI (Wireshark) or command-line comfort (tshark)

**Setup:**
1. Download Wireshark: https://www.wireshark.org/download.html
2. Install (5-10 minutes)
3. See `wireshark_quickstart.md` for beginner guide

---

### Option B: Python Script (Easier Installation!)

**Best for:** Students who prefer Python or can't/don't want to install Wireshark

**Pros:**
- Faster installation (just `pip install scapy`)
- Familiar if you know Python
- Automated extraction of DNS queries
- No GUI needed

**Cons:**
- Less educational (automates the analysis)
- One-off script (Wireshark is more versatile)

**Setup:**
```bash
# Install scapy
pip install scapy          # Windows
pip3 install scapy         # Mac/Linux

# Run the parser
cd artifacts/
python3 parse_pcap.py      # Extracts DNS queries automatically
```

**The script will show you all DNS queries - you'll still need to decode them yourself!**

---

**💡 Can't decide?** If you have time, use **Option A (Wireshark)** - it's a valuable skill. If you're in a hurry or having installation issues, use **Option B (Python)**.

**Both options lead to the same flag!** Choose what works best for you.

---

### 💡 Hints

<details>
<summary>Hint 1: Extracting DNS queries</summary>

**Option A - Using Wireshark/tshark:**

**Wireshark (GUI):**
1. Download Wireshark: https://www.wireshark.org/
2. Open `nice_list_traffic.pcap`
3. Apply filter: `dns` to see only DNS traffic
4. Look at the "Name" column for query domains

**tshark (Command-line):**
```bash
# View DNS queries only
tshark -r nice_list_traffic.pcap -Y "dns.flags.response == 0" -T fields -e dns.qry.name

# Save to file
tshark -r nice_list_traffic.pcap -Y "dns.flags.response == 0" -T fields -e dns.qry.name > dns_queries.txt
```

**Option B - Using Python Script:**

```bash
cd artifacts/
python3 parse_pcap.py
```

The script will show you all DNS queries and optionally save them to `dns_queries.txt`.

**Either way, you're looking for DNS queries to suspicious domains!**

</details>

<details>
<summary>Hint 2: Identifying suspicious DNS queries</summary>

Normal DNS queries look like:
- `google.com`
- `mail.northpole.local`
- `workshop.internal`

**Suspicious DNS tunneling queries look like:**
- `n5xw4zdpnzxw4zdpnzxw4.frostnet-cdn.com` (long random subdomain)
- `mfzxg5djnztsa43fon2a.north-pole-cache.net` (encoded data)
- Pattern: **[BASE32_DATA].[ATTACKER_DOMAIN]**

Look for:
- Very long subdomains (30+ characters)
- Random-looking character strings
- High volume of queries to the same base domain
- Only A record queries (no PTR, MX, etc.)

</details>

<details>
<summary>Hint 3: Extracting the encoded data</summary>

In Wireshark:
1. Filter: `dns and dns.qry.name contains "frostnet-cdn"`
2. Right-click a packet → Follow → UDP Stream
3. Look for the query names in the DNS packets

**Command-line extraction:**
```bash
# Extract all DNS query names
tshark -r nice_list_traffic.pcap -Y "dns.flags.response == 0" -T fields -e dns.qry.name > dns_queries.txt

# Filter for suspicious domains
grep "frostnet-cdn.com" dns_queries.txt

# Extract just the subdomains (before the first dot)
grep "frostnet-cdn.com" dns_queries.txt | cut -d'.' -f1
```

</details>

<details>
<summary>Hint 4: Decoding Base32 data</summary>

The subdomains are encoded in **Base32** (A-Z, 2-7, no special characters).

To decode:
- **CyberChef:** Use "From Base32" operation
- **Python:**
  ```python
  import base32
  decoded = base32.b32decode("N5XW4ZDPNZXW4===")
  print(decoded)
  ```
- **Linux:**
  ```bash
  echo "N5XW4ZDPNZXW4===" | base32 -d
  ```

Concatenate all the subdomain data first, then decode!

</details>

<details>
<summary>Hint 5: Finding the timeline</summary>

In Wireshark, look at the **Time** column to see when DNS tunneling started.

Filter: `dns and dns.qry.name contains "frostnet-cdn"`

The first packet timestamp shows when Jack Frost began exfiltration.

Compare to `dns_baseline.txt` to see the difference between normal and attack traffic.

</details>

<details>
<summary>Hint 6: Assembling the exfiltrated data</summary>

DNS tunneling splits data across many queries. You need to:
1. Extract ALL suspicious query names (sorted by time)
2. Extract just the subdomain part (before the `.`)
3. Concatenate them in order
4. Decode the full Base32 string
5. The decoded data will contain the flag!

**Example workflow:**
```bash
# Extract queries in time order
tshark -r nice_list_traffic.pcap -Y "dns.qry.name contains \"frostnet-cdn\"" -T fields -e dns.qry.name | cut -d'.' -f1 > subdomains.txt

# Concatenate all lines
cat subdomains.txt | tr -d '\n' > encoded_data.txt

# Decode (add padding if needed: =)
cat encoded_data.txt | base32 -d
```

</details>

<details>
<summary>Hint 7: Complete solution path</summary>

**Step-by-step:**

1. **Open PCAP in Wireshark or tshark**
2. **Filter for DNS queries:**
   - Wireshark: `dns.flags.response == 0 and dns.qry.name contains "frostnet-cdn"`
   - tshark: `-Y "dns.qry.name contains \"frostnet-cdn\""`
3. **Extract query names to file:**
   ```bash
   tshark -r nice_list_traffic.pcap -Y "dns.qry.name contains \"frostnet-cdn\"" -T fields -e dns.qry.name > queries.txt
   ```
4. **Extract subdomains only:**
   ```bash
   cat queries.txt | cut -d'.' -f1 > subdomains.txt
   ```
5. **Concatenate and decode:**
   ```bash
   cat subdomains.txt | tr -d '\n' | base32 -d
   ```
6. **The decoded output contains the flag!**

The flag will be in the format: `FROST{...}`

</details>

---

## 🤔 Questions to Consider

As you solve this challenge, think about:

1. Why is DNS traffic often allowed through firewalls without inspection?
2. How can defenders detect DNS tunneling in their networks?
3. What are the indicators of DNS tunneling vs. legitimate DNS traffic?
4. Why would an attacker choose DNS tunneling over direct data exfiltration?
5. How could the North Pole prevent this type of attack?
6. What other protocols could be abused for data exfiltration?
7. How does the volume of DNS queries help identify tunneling?

---

## ➡️ Next Steps

Once you've decoded the exfiltrated data and found the flag, use it to unlock **Day 8's** artifacts:

1. Navigate to `../day08/`
2. Find `day08_artifacts.zip`
3. Use your Day 7 flag (including `FROST{}`) as the password
4. Continue the investigation...

Jack Frost has the Nice List. The elves need to stop him before Christmas Eve! 🎄

---

**Difficulty:** Hard
**Estimated Time:** 45-60 minutes
**Skills:** PCAP analysis, DNS protocol knowledge, Base32/Base64 decoding, pattern recognition, network forensics
**Tools Needed:** Wireshark (or tshark), CyberChef (or base32 command), text editor
**Uses Previous Skills:** Base32/Base64 decoding (Day 1) ✅ | Pattern recognition (Day 2) ✅ | Log analysis (Day 3) ✅ | Reconnaissance correlation (Day 6) ✅

---

## 🎓 Real-World Context

DNS tunneling is a real technique used by:
- **APT groups** for command-and-control (C2) communication
- **Ransomware** for data exfiltration before encryption
- **Nation-state actors** to bypass network monitoring
- **Insider threats** to steal sensitive data

Famous examples:
- **OilRig APT** (2017): Used DNS tunneling for C2
- **DNSpionage** (2018): DNS hijacking + tunneling campaign
- **Maze Ransomware** (2020): Exfiltrated data via DNS before encryption

**Detection methods:**
- DNS query volume monitoring
- Subdomain length analysis
- Entropy detection (randomness)
- Baseline comparison
- Domain reputation checks
