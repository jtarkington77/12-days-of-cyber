# DNS Tunneling & Network Forensics

## What You'll Learn

This lesson covers:
- What network forensics is and why it's critical
- How DNS works and why attackers abuse it
- What DNS tunneling is and how it bypasses firewalls
- How to analyze PCAP files with Wireshark and command-line tools
- Detecting and decoding DNS tunneling exfiltration
- Real-world case studies and defensive strategies

---

## Part 1: Introduction to Network Forensics

### What is Network Forensics?

**Network forensics** is the capture, recording, and analysis of network traffic to detect security incidents, investigate breaches, and gather evidence. Think of it as being a detective for network communications.

**Why it matters:**
- Attackers must communicate over networks to steal data
- Network evidence can reveal attack timelines, methods, and scope
- Many attacks leave traces in network traffic even when other logs are wiped

### Key Concepts

**Packet:** The smallest unit of data sent over a network. Every email, web request, and file transfer is broken into packets.

**PCAP (Packet Capture):** A file format that stores captured network traffic. Tools like Wireshark can record all network activity into a PCAP file for later analysis.

**Protocol:** A set of rules for how data is formatted and transmitted. Common protocols include:
- **HTTP/HTTPS** - Web browsing
- **DNS** - Domain name resolution
- **SMTP** - Email
- **FTP** - File transfers
- **TCP/UDP** - Transport layer protocols

---

## Part 2: Understanding DNS

### What is DNS?

**DNS (Domain Name System)** is the "phone book" of the internet. It translates human-readable domain names (like `google.com`) into IP addresses (like `142.250.80.46`) that computers use to communicate.

### How DNS Works

```
1. You type "google.com" in your browser
2. Your computer asks a DNS server: "What's the IP for google.com?"
3. DNS server responds: "It's 142.250.80.46"
4. Your browser connects to 142.250.80.46
```

### DNS Query Structure

A DNS query has several parts:
```
subdomain.domain.tld
   |        |     |
   |        |     └── Top-level domain (.com, .org, .net)
   |        └── Main domain (google, microsoft)
   └── Subdomain (www, mail, api)
```

**Example:** `mail.google.com`
- Subdomain: `mail`
- Domain: `google`
- TLD: `com`

### Why DNS is Special

DNS traffic is almost ALWAYS allowed through firewalls because:
1. **Essential for internet functionality** - Block DNS, block everything
2. **Uses port 53** - Standard, well-known port
3. **Small, frequent queries** - Hard to monitor all of them
4. **Rarely inspected** - Most security tools ignore DNS content

**This makes DNS a perfect covert channel for attackers!**

---

## Part 3: DNS Tunneling Explained

### What is DNS Tunneling?

**DNS tunneling** is a technique where attackers encode data (files, commands, stolen information) inside DNS queries and responses. Instead of sending data directly, they "tunnel" it through the DNS protocol.

### How It Works

**Normal DNS query:**
```
Query: www.google.com → Response: 142.250.80.46
```

**DNS tunneling query:**
```
Query: JRQWC3THEBSXQ5BO.attacker.com → Response: (acknowledgment)
         ^^^^^^^^^^^^^^^^^
         This is encoded stolen data!
```

The attacker controls `attacker.com` and receives all queries to it. The "subdomain" part contains Base32 or Base64 encoded data.

### Step-by-Step Attack Flow

```
1. Malware on victim computer wants to steal a file
2. File is split into small chunks (DNS has size limits)
3. Each chunk is Base32 encoded
4. Encoded chunks become subdomains: chunk1.attacker.com, chunk2.attacker.com
5. Victim's computer sends DNS queries for each "domain"
6. Attacker's DNS server receives queries, extracts encoded data
7. Attacker decodes and reassembles the stolen file
```

### Why Base32?

DNS domain names have restrictions:
- Only letters, numbers, and hyphens allowed
- Case-insensitive
- Maximum 63 characters per label (subdomain)
- Maximum 253 characters total

**Base32** uses only: `A-Z` and `2-7` (plus `=` for padding)
- No special characters
- Works within DNS restrictions
- Any binary data can be encoded

### Visual Example

**Original data:** "SECRET FILE CONTENTS"

**Base32 encoded:** `KNSXS5DFNZ2C4YLNMU======`

**DNS query:** `KNSXS5DFNZ2C4YLNMU.frostnet-cdn.com`

---

## Part 4: Detecting DNS Tunneling

### Red Flags to Look For

**1. Long Subdomains**
```
Normal:     www.google.com (3 chars)
Suspicious: JRQWC3THEBSXQ5BONVTCA5DIMUQHO33S.evil.com (32+ chars)
```

**2. High Entropy (Randomness)**
```
Normal:     mail.company.com (readable words)
Suspicious: X7KQ2NLMB4VXYZ.evil.com (random characters)
```

**3. High Query Volume**
```
Normal:     50-100 DNS queries per minute
Suspicious: 200+ queries per minute to single domain
```

**4. Unusual Query Types**
```
Normal:     Mix of A, AAAA, MX, TXT records
Suspicious: Only A records, repeatedly
```

**5. Newly Registered Domains**
```
Normal:     google.com (registered 1997)
Suspicious: frostnet-cdn.com (registered last week)
```

**6. Base32/Base64 Patterns**
```
Look for: Only A-Z, 2-7, and = characters
Pattern:  Ends with === (Base32 padding)
```

### Detection Checklist

When analyzing DNS traffic, ask:
- [ ] Are subdomains unusually long (>30 characters)?
- [ ] Do subdomains look random (not real words)?
- [ ] Is query volume abnormally high?
- [ ] Are queries going to unknown/new domains?
- [ ] Is traffic occurring at unusual times?
- [ ] Does the pattern match Base32/Base64 encoding?

---

## Part 5: Analyzing PCAP Files

### What is a PCAP File?

A **PCAP (Packet Capture)** file is a recording of network traffic. It contains every packet that passed through a network interface during the capture period.

### Tools for PCAP Analysis

**Wireshark (GUI)**
- Visual packet inspection
- Powerful filtering
- Color-coded protocols
- Best for beginners

**tshark (Command-line)**
- Wireshark's CLI version
- Faster for large files
- Scriptable
- Good for automation

**Python + Scapy**
- Programmatic analysis
- Custom extraction
- Flexible scripting

### Basic Wireshark Workflow

1. **Open the PCAP file**
   - File → Open → Select your .pcap file

2. **Get overview**
   - Statistics → Protocol Hierarchy
   - See breakdown of traffic types

3. **Filter for relevant traffic**
   - Type filter in the filter bar
   - Press Enter to apply

4. **Analyze packets**
   - Click a packet to see details
   - Expand protocol layers
   - Look for anomalies

5. **Export findings**
   - File → Export → Save filtered data

### Essential Wireshark Filters

```
# Show only DNS traffic
dns

# Show only DNS queries (not responses)
dns.flags.response == 0

# Show queries containing specific text
dns.qry.name contains "frostnet"

# Show traffic to/from specific IP
ip.addr == 203.0.113.99

# Combine filters with AND
dns and ip.src == 10.10.10.50
```

### tshark Commands

```bash
# View all packets
tshark -r capture.pcap

# Filter for DNS queries
tshark -r capture.pcap -Y "dns.flags.response == 0"

# Extract specific field (query names)
tshark -r capture.pcap -Y "dns" -T fields -e dns.qry.name

# Save filtered output
tshark -r capture.pcap -Y "dns.qry.name contains \"frostnet\"" \
  -T fields -e dns.qry.name > queries.txt
```

---

## Part 6: Decoding Exfiltrated Data

### The Decoding Process

Once you identify DNS tunneling queries, follow these steps:

**Step 1: Extract the query names**
```bash
# Using tshark
tshark -r capture.pcap \
  -Y "dns.qry.name contains \"evil.com\"" \
  -T fields -e dns.qry.name > queries.txt
```

**Step 2: Extract just the subdomains**
```bash
# Remove the domain part, keep only encoded data
cat queries.txt | cut -d'.' -f1 > subdomains.txt
```

**Step 3: Concatenate all chunks**
```bash
# DNS tunneling splits data across many queries
cat subdomains.txt | tr -d '\n' > encoded_data.txt
```

**Step 4: Decode**
```bash
# Base32 decode
cat encoded_data.txt | base32 -d

# Or use CyberChef:
# 1. Paste encoded data
# 2. Add "From Base32" operation
# 3. See decoded output
```

### Common Encoding Types

**Base32**
- Characters: A-Z, 2-7, =
- Used when: DNS restrictions apply
- Padding: `=` at end

**Base64**
- Characters: A-Z, a-z, 0-9, +, /, =
- More efficient but may not work in all DNS contexts
- Padding: `=` at end

**Hex**
- Characters: 0-9, A-F
- Simple but inefficient (doubles data size)
- No padding

---

## Part 7: Real-World Examples

### Case Study 1: OilRig APT (2017)

**Attacker:** Iranian threat group
**Target:** Middle Eastern organizations
**Method:** Custom DNS tunneling tool called "BONDUPDATER"

**How it worked:**
- Malware communicated with C2 servers using DNS TXT records
- Commands were sent via DNS responses
- Stolen data was exfiltrated via DNS queries
- Bypassed traditional security controls

**Detection:** Abnormal DNS TXT record queries to newly registered domains

### Case Study 2: DNSpionage (2018)

**Attacker:** Unknown (suspected nation-state)
**Target:** Lebanese and UAE organizations
**Method:** DNS hijacking combined with tunneling

**How it worked:**
- Compromised DNS registrars
- Redirected legitimate domains to attacker servers
- Used DNS tunneling for C2 communication
- Stole credentials and sensitive data

**Detection:** DNS resolution changes for known domains

### Case Study 3: Maze Ransomware (2020)

**Attacker:** Maze ransomware gang
**Target:** Global organizations
**Method:** Data exfiltration before encryption

**How it worked:**
- Ransomware exfiltrated sensitive files before encrypting
- Used DNS tunneling to steal data
- Threatened to publish data if ransom wasn't paid
- "Double extortion" model

**Detection:** High volume DNS queries to unknown domains during off-hours

---

## Part 8: Defense Strategies

### Prevention

**1. DNS Filtering**
- Use DNS security services (Cisco Umbrella, Cloudflare Gateway)
- Block known malicious domains
- Restrict external DNS resolver access

**2. Network Segmentation**
- Limit which systems can make DNS queries
- Force all DNS through internal resolvers
- Monitor DNS traffic at chokepoints

**3. Query Restrictions**
- Limit subdomain length
- Block unusual record types
- Rate limit DNS queries

### Detection

**1. Monitor Query Length**
```python
if len(subdomain) > 30:
    alert("Suspicious long subdomain")
```

**2. Calculate Entropy**
```python
# High entropy = random = possibly encoded
if entropy(subdomain) > 4.0:
    alert("High entropy subdomain")
```

**3. Baseline Comparison**
- Establish normal DNS patterns
- Alert on significant deviations
- Track new domains

**4. Threat Intelligence**
- Subscribe to domain blocklists
- Check domain age/reputation
- Monitor for newly registered domains

### Response

When DNS tunneling is detected:

1. **Isolate affected systems** - Prevent further exfiltration
2. **Capture traffic** - Get full PCAP for forensics
3. **Block domains** - Add to firewall/DNS blocklist
4. **Decode exfiltration** - Determine what was stolen
5. **Hunt for malware** - Find and remove the source
6. **Assess impact** - What data was compromised?

---

## Part 9: Hands-On Practice

### Exercise 1: Identify Suspicious Queries

Look at these DNS queries. Which are suspicious?

```
1. www.google.com
2. JRQWC3THEBSXQ5BONVTCA5DIMUQHO33S.frostnet.com
3. mail.company.com
4. api.github.com
5. MZRW6ZDPEBRW63TTMVZXGIDUN5XG65LN.evil.net
```

**Answer:** Queries 2 and 5 are suspicious (long, random, Base32-like subdomains)

### Exercise 2: Decode This Data

```
Subdomain: ORSXG5A=
Encoding: Base32
```

**Steps:**
1. Open CyberChef
2. Paste: ORSXG5A=
3. Add "From Base32"
4. Result: "test"

### Exercise 3: Filter Practice

Write Wireshark filters for:
1. All DNS traffic
2. DNS queries only
3. Queries to "frostnet-cdn.com"

**Answers:**
1. `dns`
2. `dns.flags.response == 0`
3. `dns.qry.name contains "frostnet-cdn"`

---

## Part 10: Tools Reference

### Wireshark Installation

**Windows/Mac:**
1. Download from https://www.wireshark.org/
2. Run installer
3. Accept defaults

**Linux:**
```bash
sudo apt-get install wireshark
```

### tshark Quick Reference

```bash
# Basic read
tshark -r file.pcap

# Count packets
tshark -r file.pcap | wc -l

# Filter
tshark -r file.pcap -Y "filter"

# Extract field
tshark -r file.pcap -T fields -e field.name

# Save output
tshark -r file.pcap -Y "filter" > output.txt
```

### CyberChef Operations

For DNS tunneling analysis:
- **From Base32** - Decode Base32 data
- **From Base64** - Decode Base64 data
- **From Hex** - Decode hexadecimal
- **Remove whitespace** - Clean up extracted data

---

## Summary

**Key Takeaways:**

1. **DNS tunneling** abuses the DNS protocol to hide data exfiltration
2. **Attackers encode** stolen data in subdomain names
3. **Detection signs:** Long subdomains, high entropy, unusual volume
4. **Analysis tools:** Wireshark, tshark, Python/Scapy
5. **Decoding process:** Extract → Clean → Concatenate → Decode
6. **Defense:** Filter, monitor, baseline, respond

**Remember:** DNS is a critical protocol that attackers abuse because it's almost always allowed through firewalls. Learning to analyze DNS traffic is an essential skill for any security analyst!

---

## Additional Resources

**Documentation:**
- Wireshark User Guide: https://www.wireshark.org/docs/
- tshark Manual: https://www.wireshark.org/docs/man-pages/tshark.html

**Practice:**
- Malware Traffic Analysis: https://www.malware-traffic-analysis.net/
- Wireshark Sample Captures: https://wiki.wireshark.org/SampleCaptures

**Tools:**
- DNScat2 (research): https://github.com/iagox86/dnscat2
- Iodine (research): https://code.kryo.se/iodine/

**Learning:**
- SANS FOR572: Advanced Network Forensics
- TryHackMe Network Forensics rooms

---

*This lesson is part of the 12 Days of Cyber challenge. Good luck with Day 7!*
