[← Previous Day](../day11/README.md) | [Main README](../README.md) | **Day 12** | [Next Day →](../day13/README.md)

---

![Day 12](../Images/Day12.png)

# 🎄 Day 12 (December 23) - The Final Payload

## 🎅 The Story

**December 23, 2024 - 06:45 AM (North Pole Time)**

*One day before Christmas Eve.*

Dekker Frostbeard hasn't slept. Coffee cups litter his desk at the North Pole Security Operations Center. After 11 days of tracking Jack Frost's attacks—from encoded messages to database breaches, from steganography to persistence mechanisms—they've thwarted every attack. The systems are secured. The Nice List is safe. Christmas preparations continue.

But something feels wrong.

"Dekker, you need to see this," Merry Tinselcode rushes in, her normally cheerful face pale. "We just got a hit from our email security gateway. Yesterday afternoon, someone sent a PDF attachment to 47 elves across the workshop—right before the end of shift."

She pulls up the email on the main screen:

```
From: North Pole Safety Committee <safety@n0rthp0le-safety.com>
To: Workshop Staff <workshop-all@northpole.com>
Date: December 22, 2024, 4:47 PM
Subject: URGENT: Toy Recall - Action Required

Dear Workshop Team,

We've identified a safety issue with Batch #2024-12 of wooden trains.
Please review the attached recall notice IMMEDIATELY and follow the
documented procedures.

Failure to comply may result in delayed Christmas deliveries.

Attachment: Toy_Recall_Notice.pdf (247 KB)

- North Pole Safety Committee
```

Tom Icicle examines the email headers. "That domain—`n0rthp0le-safety.com`—it's a homograph attack. Zero instead of O. This isn't from our safety committee."

"47 elves," Sarah Hollycode says grimly. "That's nearly everyone in the workshop. If even a few of them opened that PDF..."

Dekker pulls up the email gateway logs. "Our system flagged it as suspicious and quarantined the attachment. But the email got through. Did anyone open it?"

Merry checks the mail server. "No opens recorded. The PDF never left the quarantine folder. We caught it just in time."

"We need to analyze that PDF," Dekker says. "If this is Jack Frost's final attack, we need to know what it does. What was he trying to deploy?"

Tom retrieves the quarantined file. "I've got it. `Toy_Recall_Notice.pdf` - 247 kilobytes. Looks innocent, but if Jack Frost sent it, there's something nasty inside."

Sarah opens her forensics toolkit. "PDF malware is sophisticated. Attackers embed JavaScript, hide streams with encoded payloads, use multiple layers of obfuscation. We'll need to dissect this carefully."

"Can we just open it and see what happens?" Merry asks.

"Absolutely not," Tom replies. "Opening a malicious PDF can trigger exploits, download malware, or call back to command and control servers. We analyze it statically—without executing anything."

Dekker stands. "Then let's tear it apart. Find out what Jack Frost was trying to deploy with one day left until Christmas Eve. This is his endgame—and we need to understand it before tomorrow."

Outside, snow falls steadily. Inside the SOC, the team prepares to analyze the most sophisticated attack yet: a weaponized PDF designed to compromise the entire North Pole workshop on the eve of Christmas.

---

## 🎯 **YOUR MISSION: Analyze Jack Frost's Weaponized PDF!**

You are **Tom Icicle**, Incident Response Lead. Your mission is to perform static analysis on a malicious PDF without executing it, extract hidden payloads, and understand Jack Frost's final attack.

**You will:**
✅ Use **PDF analysis tools** to examine malicious document structure
✅ Identify suspicious PDF elements (JavaScript, streams, embedded objects)
✅ Extract and decode obfuscated payloads
✅ Uncover Jack Frost's command & control infrastructure
✅ Understand real-world PDF-based attacks

**You're learning real malware analysis skills used by security researchers!**

---

## 🛠️ Prerequisites & Setup

**You'll need:**
- **Python 3.7 or higher** (already installed from previous days)
- **PDF analysis tools** - We'll install these
- **Text editor** - For examining extracted content
- **CyberChef** (web-based, for decoding) - Used in Day 1
- **Completed Day 11?** You'll need the Day 11 flag to unlock these artifacts!

### Check Your Python Version First!

```bash
python3 --version
# Should show: Python 3.7.x or higher (3.8, 3.9, 3.10, 3.11, 3.12 all work)
```

**If you get errors or have an older version:**
- **Windows:** Download from https://python.org/downloads/ (get the latest 3.x version)
- **Ubuntu/Debian:** `sudo apt update && sudo apt install python3`
- **macOS:** `brew install python3`

### Installing PDF Analysis Tools

**⚠️ IMPORTANT: The pip package names have changed. Use this method:**

**Recommended Method - Download Didier Stevens' Tools Directly:**

These are the standard tools used by professional malware analysts:

```bash
# Create a tools directory
mkdir -p ~/pdf-tools
cd ~/pdf-tools

# Download PDFiD (identifies suspicious PDF elements)
curl -O https://raw.githubusercontent.com/DidierStevens/DidierStevensSuite/master/pdfid.py

# Download PDF-Parser (extracts and analyzes PDF objects)
curl -O https://raw.githubusercontent.com/DidierStevens/DidierStevensSuite/master/pdf-parser.py

# For Windows users without curl, use PowerShell:
# Invoke-WebRequest -Uri "https://raw.githubusercontent.com/DidierStevens/DidierStevensSuite/master/pdfid.py" -OutFile "pdfid.py"
# Invoke-WebRequest -Uri "https://raw.githubusercontent.com/DidierStevens/DidierStevensSuite/master/pdf-parser.py" -OutFile "pdf-parser.py"
```

**Usage (after downloading):**
```bash
# Navigate to your tools directory, then:
python3 pdfid.py /path/to/Toy_Recall_Notice.pdf
python3 pdf-parser.py /path/to/Toy_Recall_Notice.pdf
```

**Alternative Method - pip (may have compatibility issues):**

```bash
# Try pip install - some packages may not be available for all Python versions
pip3 install pdfid
pip3 install pdf-parser

# If those fail, the direct download method above always works!
```

**Verify your setup:**
```bash
python3 --version  # Should be 3.7+
cd ~/pdf-tools && python3 pdfid.py --help  # Should show help text
```

---

## 📚 Lesson: Learn the Technique

**Before starting this challenge, read the lesson:**
📖 [PDF Malware Analysis - Static Analysis Techniques](./LESSON.md)

This comprehensive lesson covers:
- How PDFs work and their internal structure
- Common PDF-based attack vectors
- Tools for PDF analysis (pdfid, pdf-parser, peepdf)
- Extracting and decoding embedded payloads
- Real-world PDF exploits and campaigns
- Safe analysis practices

**New to PDF analysis?** The lesson assumes zero prior knowledge!

**Already familiar with PDF forensics?** Jump straight to the challenge! 🚀

---

## 🔍 Artifacts

**Password required:** Use Day 11's flag to extract `artifacts.7z`

In the `artifacts/` folder you'll find:

- **`Toy_Recall_Notice.pdf`** - The weaponized PDF Jack Frost sent
- **`email_headers.txt`** - Complete email headers from the phishing message
- **`pdf_analysis_guide.txt`** - Quick reference for PDF analysis commands
- **`incident_notes.txt`** - SOC team's initial findings and clues

⚠️ **IMPORTANT:** Do NOT open the PDF in a normal PDF reader! Analyze it with the provided tools only.

---

## 🚀 Quick Start

```bash
# 1. Extract the artifacts using Day 11's flag as the password
# Hint: The flag you got from Day 11's steganography challenge

# 2. Navigate to artifacts directory
cd day12/artifacts/

# 3. Identify suspicious PDF elements
pdfid Toy_Recall_Notice.pdf

# 4. Extract JavaScript and streams
pdf-parser --search javascript Toy_Recall_Notice.pdf

# 5. Decode extracted payloads
# Use CyberChef or Python for decoding

# 6. Find the flag in the decoded content
```

---

## 🎯 The Challenge

You are **Tom Icicle**, Incident Response Lead. Your mission: analyze the weaponized PDF to understand Jack Frost's final attack without executing any malicious code.

**Step-by-Step Process:**

### Phase 1: Setup and Initial Recon (10 minutes)

**Navigate to the artifacts folder:**

```bash
cd /mnt/c/Users/YOUR_USERNAME/Desktop/12-days-of-cyber/day12/artifacts
```

**Or for Linux/macOS:**
```bash
cd ~/Desktop/12-days-of-cyber/day12/artifacts
```

**Examine what you have:**
```bash
ls -lh
```

**Expected files:**
```
Toy_Recall_Notice.pdf
email_headers.txt
pdf_analysis_guide.txt
incident_notes.txt
```

**Read the incident notes:**
```bash
cat incident_notes.txt
```

💡 **Hint:** The incident notes contain important clues about what to look for in the PDF.

---

### Phase 2: PDF Identification (15 minutes)

**Run PDFiD to identify suspicious elements:**

```bash
pdfid Toy_Recall_Notice.pdf
```

**What to look for:**
- `/JS` or `/JavaScript` - Embedded JavaScript code
- `/AA` or `/OpenAction` - Automatic actions
- `/AcroForm` - Form data
- `/EmbeddedFile` - Embedded files
- `/ObjStm` - Object streams (can hide malicious content)

**Your Task:**
1. Identify which suspicious elements are present
2. Note their object/stream numbers
3. Document which ones to investigate further

💡 **Hint:** JavaScript in PDFs is almost always suspicious!

---

### Phase 3: Extract JavaScript (20 minutes)

**Use pdf-parser to search for JavaScript:**

```bash
pdf-parser --search javascript Toy_Recall_Notice.pdf
```

**Or search for specific objects:**
```bash
pdf-parser --object [OBJECT_NUMBER] Toy_Recall_Notice.pdf
```

**Your Task:**
1. Extract the JavaScript code
2. Save it to a file for analysis
3. Look for obfuscation patterns

**To save extracted content:**
```bash
pdf-parser --object [NUMBER] --raw Toy_Recall_Notice.pdf > extracted_js.txt
```

💡 **Hint:** JavaScript might be hex-encoded or obfuscated. Look for patterns like `\x41\x42\x43`.

---

### Phase 4: Decode Obfuscated Content (25 minutes)

**Common encoding in PDFs:**
- **Hex encoding:** `\x48\x65\x6c\x6c\x6f` = "Hello"
- **Octal encoding:** `\110\145\154\154\157` = "Hello"
- **Base64 encoding:** `SGVsbG8=` = "Hello"
- **URL encoding:** `%48%65%6C%6C%6F` = "Hello"

**Decoding methods:**

**Method 1: Python (for hex)**
```python
hex_string = "\\x48\\x65\\x6c\\x6c\\x6f"
decoded = bytes.fromhex(hex_string.replace('\\x', '')).decode('utf-8')
print(decoded)
```

**Method 2: CyberChef**
1. Go to https://gchq.github.io/CyberChef/
2. Paste the encoded content
3. Try recipes: "From Hex", "From Base64", "URL Decode"

**Your Task:**
1. Decode all obfuscated strings in the JavaScript
2. Look for URLs, IP addresses, commands
3. Find the hidden payload

💡 **Hint:** PDFs often use multiple layers of encoding. Decode in stages.

---

### Phase 5: Extract Streams and Objects (20 minutes)

**List all objects in the PDF:**

```bash
pdf-parser --stats Toy_Recall_Notice.pdf
```

**Extract specific streams:**

```bash
# Extract stream with FlateDecode filter
pdf-parser --object [NUMBER] --filter Toy_Recall_Notice.pdf
```

**Your Task:**
1. Identify suspicious streams
2. Extract and decode them
3. Look for hidden data

💡 **Hint:** `/Filter` types like `/FlateDecode` compress data. Use `--filter` to decompress.

---

### Phase 6: Analyze the Payload (15 minutes)

**Once you've extracted and decoded the content, analyze it:**

**Look for:**
- Command & Control (C2) server URLs/IPs
- Downloaded malware URLs
- Exfiltration endpoints
- Encoded commands
- The investigation flag

**Questions to answer:**
1. What does the malicious payload do?
2. What C2 infrastructure does it contact?
3. What data would it exfiltrate?
4. Where is the flag hidden?

💡 **Hint:** The flag will be in the final decoded payload, in `FROST{...}` format.

---

### Phase 7: Document Your Findings (10 minutes)

**Create a summary of the attack:**

```bash
# What you found:
# - JavaScript that executes on PDF open
# - Obfuscated payload that [does what?]
# - C2 server: [IP/URL]
# - Exfiltration method: [how?]
# - Flag: FROST{...}
```

**Understanding the attack chain:**
1. User opens PDF
2. JavaScript executes automatically
3. Payload decodes and runs
4. Connects to C2 server
5. Exfiltrates data or downloads malware

---

## 🛡️ Real-World Application

**These techniques are used in REAL attacks:**

### PDF-Based Attack Campaigns:
- **APT28 (Fancy Bear)** - PDF exploits with embedded Flash
- **Carberp Banking Trojan** - Malicious PDFs with JavaScript
- **DarkHotel APT** - Weaponized PDFs in spear-phishing
- **Emotet** - PDF attachments with malicious macros

### Real Incidents:
- **Adobe 0-days** - CVE-2018-4990, CVE-2021-28550
- **Banking Trojans** - Zeus, Dridex use PDF droppers
- **Ransomware** - PDFs as initial infection vector
- **APT Campaigns** - Targeted attacks via PDF

### Why Defenders Need These Skills:
1. **Malware Analysis** - Understanding attack payloads
2. **Incident Response** - Analyzing phishing attachments
3. **Threat Hunting** - Finding malicious documents
4. **Email Security** - Building detection rules

---

## ❓ Troubleshooting

**Problem:** "pdfid: command not found"
**Solution:**
```bash
# Install via pip
pip3 install pdfid-python

# OR download directly
python3 pdfid.py Toy_Recall_Notice.pdf
```

**Problem:** "Can't decode the content"
**Solution:**
- Try multiple encoding methods (hex, base64, octal)
- Use CyberChef with different recipes
- Look for encoding indicators in the code

**Problem:** "Don't know which objects to extract"
**Solution:**
```bash
# Start with JavaScript objects
pdf-parser --search javascript Toy_Recall_Notice.pdf

# Then check OpenAction
pdf-parser --search openaction Toy_Recall_Notice.pdf
```

**Problem:** "Extracted content is gibberish"
**Solution:**
- Check if it's compressed (FlateDecode)
- Use `--filter` flag to decompress
- Try different character encodings

---

## 📖 Additional Resources

**PDF Analysis Tools:**
- PDFiD: https://blog.didierstevens.com/programs/pdf-tools/
- PDF-Parser: https://blog.didierstevens.com/programs/pdf-tools/
- Peepdf: https://eternal-todo.com/tools/peepdf-pdf-analysis-tool

**Learning Resources:**
- PDF Format Specification: https://www.adobe.com/devnet/pdf/pdf_reference.html
- Malicious PDF Analysis: https://zeltser.com/analyzing-malicious-documents/
- SANS PDF Analysis: https://www.sans.org/blog/

**MITRE ATT&CK Framework:**
- **Tactic:** TA0001 - Initial Access
- **Technique:** T1566.001 - Phishing: Spearphishing Attachment
- **Sub-technique:** T1204.002 - User Execution: Malicious File

---

## ➡️ Next Steps

Once you've analyzed the PDF and captured the flag, use it to unlock the **Christmas Eve Final Challenge**:

1. Navigate to `../day13/`
2. Find `artifacts.7z`
3. Use your Day 12 flag (including `FROST{}`) as the password
4. Complete the final investigation...

Jack Frost's endgame is revealed. Can you stop him before Christmas? 🎄

---

**Difficulty:** Hard (PDF malware analysis)
**Estimated Time:** 90-120 minutes
**Skills:** PDF forensics, static malware analysis, decoding, JavaScript analysis
**Tools Needed:** Python 3, PDFiD, PDF-Parser, CyberChef

**NEW Skills Learned:**
- ✅ PDF internal structure and format
- ✅ Using PDFiD to identify suspicious elements
- ✅ Using PDF-Parser to extract objects/streams
- ✅ Static malware analysis (no execution)
- ✅ Multi-layer decode techniques
- ✅ JavaScript deobfuscation in PDFs
- ✅ Understanding PDF-based attacks
- ✅ Safe document analysis practices

---

🎄 **You've completed Day 12! You're now a PDF Malware Analyst!** ❄️

*Good luck analyzing, and may your forensic skills save Christmas! 🎅*

---

[← Previous Day](../day11/README.md) | [Main README](../README.md) | **Day 12** | [Next Day →](../day13/README.md)
