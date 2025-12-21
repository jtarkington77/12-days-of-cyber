# Lesson: Windows Persistence Mechanisms

## Introduction

**Persistence** is the ability for malware to survive system reboots, user logouts, and maintain access to a compromised system over time. It's one of the most critical techniques attackers use after initial compromise.

In the **MITRE ATT&CK Framework**, persistence is **Tactic TA0003** - one of the 14 core tactics used by adversaries.

---

## Why Persistence Matters

### The Attacker's Perspective:
1. **Initial compromise is expensive** - Exploits, phishing campaigns, and zero-days cost time and resources
2. **Systems reboot** - Without persistence, access is lost on the next restart
3. **Long-term access = more damage** - Data exfiltration, lateral movement, and reconnaissance require time
4. **Detection evasion** - Good persistence mechanisms survive even some security tool scans

### The Defender's Perspective:
1. **Persistence = confirmed compromise** - Finding persistence proves an attacker has been on the system
2. **Multiple mechanisms = advanced attacker** - APT groups deploy 3-5 different persistence methods
3. **Persistence hunting is forensics 101** - First thing you check during incident response
4. **Removal is critical** - Malware can survive even after "cleaning" if persistence remains

---

## The 4 Persistence Techniques in This Challenge

### 1. Registry Run Keys (MITRE T1547.001)

**How It Works:**
Windows automatically executes programs listed in certain registry keys when a user logs in.

**Common Locations:**
```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

**Attacker Technique:**
```
REG ADD "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v "WindowsUpdate" /t REG_SZ /d "C:\malware.exe"
```

**Why Attackers Love It:**
- Simple to implement (one registry write)
- Survives reboots
- User-level keys don't require admin privileges
- Blends in with legitimate startup programs

**Detection Methods:**
- Use `Autoruns` from Sysinternals
- PowerShell: `Get-ItemProperty HKCU:\Software\Microsoft\Windows\CurrentVersion\Run`
- Registry analysis tools (like our `reg_analyzer.py`)
- Monitor for new registry Run key entries (Event ID 13 in Sysmon)

**Real-World Examples:**
- **Emotet** - Uses Run keys for basic persistence
- **TrickBot** - Creates multiple Run key entries
- **IcedID** - Modifies user Run keys during installation

---

### 2. Scheduled Tasks (MITRE T1053.005)

**How It Works:**
Windows Task Scheduler executes programs at specific times or events (logon, startup, idle, etc.)

**Attacker Technique:**
```powershell
$action = New-ScheduledTaskAction -Execute "C:\malware.exe"
$trigger = New-ScheduledTaskTrigger -AtLogon
Register-ScheduledTask -TaskName "SystemUpdate" -Action $action -Trigger $trigger
```

**Task Configuration File:**
Tasks are stored as XML in `C:\Windows\System32\Tasks\` and configured via:
- **Triggers** - When to execute (logon, boot, time interval)
- **Actions** - What to execute (command, script, binary)
- **Principals** - What user/privileges to run with
- **Settings** - Hidden, restart on failure, etc.

**Why Attackers Love It:**
- More flexible than Run keys (time-based, event-based)
- Can be configured as "Hidden" (invisible in Task Scheduler GUI)
- Can run with elevated privileges
- Can trigger on system events (network connection, idle, etc.)

**Detection Methods:**
- PowerShell: `Get-ScheduledTask | Where-Object {$_.State -eq "Ready"}`
- Analyze XML files in `C:\Windows\System32\Tasks\`
- Look for tasks with suspicious names mimicking Windows tasks
- Check for CommandLineEventConsumer executions
- Monitor task creation (Event ID 4698 in Security log)

**Red Flags:**
- Task is marked as "Hidden"
- Runs with HighestAvailable privileges
- Triggers on user logon
- Command uses `cmd.exe`, `powershell.exe`, or `wscript.exe`
- Task name mimics legitimate Windows task names

**Real-World Examples:**
- **APT41** - Uses scheduled tasks for both persistence and execution
- **BRONZE BUTLER** - Creates tasks that run every 10 minutes
- **Kimsuky** - Schedules tasks to run at specific times daily

---

### 3. WMI Event Subscriptions (MITRE T1546.003)

**How It Works:**
Windows Management Instrumentation (WMI) can execute actions based on system events through **Event Subscriptions**.

**Three Components:**
1. **Event Filter** - Defines WHAT to monitor (e.g., "process starts", "time interval")
2. **Event Consumer** - Defines WHAT to execute (command, script)
3. **Filter-to-Consumer Binding** - Links them together

**Attacker Technique:**
```powershell
# Create Event Filter (trigger every 60 seconds)
$FilterArgs = @{
    Name = '__EventFilter_Persistence'
    EventNamespace = 'root\CIMv2'
    QueryLanguage = 'WQL'
    Query = "SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"
}
$Filter = Set-WmiInstance -Namespace root\subscription -Class __EventFilter -Arguments $FilterArgs

# Create Event Consumer (execute malware)
$ConsumerArgs = @{
    Name = '__EventConsumer_Persistence'
    CommandLineTemplate = 'C:\malware.exe'
}
$Consumer = Set-WmiInstance -Namespace root\subscription -Class CommandLineEventConsumer -Arguments $ConsumerArgs

# Bind them together
Set-WmiInstance -Namespace root\subscription -Class __FilterToConsumerBinding -Arguments @{
    Filter = $Filter
    Consumer = $Consumer
}
```

**Why Attackers Love It:**
- **Stealth** - Not visible in standard startup locations
- **Flexible** - Can trigger on almost any system event
- **Powerful** - WMI has deep system access
- **Persistent** - Survives reboots
- **Overlooked** - Many defenders don't check WMI subscriptions

**Detection Methods:**
- PowerShell:
  ```powershell
  Get-WmiObject -Namespace root\subscription -Class __EventFilter
  Get-WmiObject -Namespace root\subscription -Class CommandLineEventConsumer
  Get-WmiObject -Namespace root\subscription -Class __FilterToConsumerBinding
  ```
- Look for suspicious consumer names (especially with `__` prefix)
- Check for CommandLineEventConsumer or ActiveScriptEventConsumer
- Monitor WMI subscription creation (Event ID 19-21 in Sysmon)

**Red Flags:**
- Event filter or consumer name starts with `__` (double underscore)
- Consumer executes `cmd.exe`, `powershell.exe`, or `wscript.exe`
- Query monitors system events frequently (WITHIN 60 seconds)
- Consumer name doesn't match a known legitimate subscription

**Real-World Examples:**
- **APT29 (Cozy Bear)** - Famous for using WMI event subscriptions
- **BRONZE BUTLER** - Uses WMI for persistence in targeted attacks
- **Turla** - Combines WMI subscriptions with other techniques

---

### 4. Windows Services (MITRE T1543.003)

**How It Works:**
Windows Services are long-running background processes managed by the Service Control Manager (SCM). They can start automatically on boot, before any user logs in.

**Attacker Technique:**
```powershell
New-Service -Name "WindowsDefender" -BinaryPathName "C:\malware.exe" -StartupType Automatic
```

Or via `sc.exe`:
```cmd
sc create "WindowsDefender" binPath= "C:\malware.exe" start= auto
sc start "WindowsDefender"
```

**Service Configuration:**
Services are stored in the registry at:
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\<ServiceName>
```

**Key Properties:**
- **ServiceName** - Internal name
- **DisplayName** - What users see
- **Description** - Service description
- **Start Type** - AUTOMATIC, MANUAL, DISABLED, BOOT_START, SYSTEM_START
- **BinaryPathName** - Executable to run
- **ServiceStartName** - Account to run as (LocalSystem, NetworkService, etc.)

**Why Attackers Love It:**
- **High privileges** - Can run as SYSTEM
- **Early execution** - Starts before user logon
- **Persistent** - Survives reboots
- **Stealthy** - Can mimic legitimate Windows services
- **Robust** - Service Control Manager can auto-restart failed services

**Detection Methods:**
- PowerShell: `Get-Service | Where-Object {$_.Status -eq "Running"}`
- Check service registry keys
- Analyze service binary paths (should be in `C:\Windows\System32\`)
- Look for services with names similar to legitimate Windows services
- Monitor service creation (Event ID 7045 in System log)

**Red Flags:**
- Service name mimics Windows services ("WindowsDefender" vs "Windows Defender")
- Binary path is NOT in `System32` (e.g., `C:\Users\`, `C:\ProgramData\`)
- Service runs as LocalSystem but was recently created
- Description is generic ("Windows service", "System service")
- DisplayName doesn't match known services

**Real-World Examples:**
- **APT41** - Creates services disguised as Windows Update
- **BRONZE ATLAS** - Installs services for long-term persistence
- **Wizard Spider (Ryuk)** - Uses services for ransomware deployment

---

## Persistence Hunting Methodology

### Step 1: Enumerate Startup Locations
Check all common persistence locations:
```powershell
# Registry Run keys
Get-ItemProperty HKCU:\Software\Microsoft\Windows\CurrentVersion\Run
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Run

# Scheduled Tasks
Get-ScheduledTask | Where-Object {$_.State -ne "Disabled"}

# WMI Subscriptions
Get-WmiObject -Namespace root\subscription -Class __EventFilter
Get-WmiObject -Namespace root\subscription -Class CommandLineEventConsumer

# Services
Get-Service | Where-Object {$_.StartType -eq "Automatic"}
```

### Step 2: Identify Suspicious Entries
Look for:
- Names that mimic legitimate Windows components
- Recently created/modified entries
- Executables in unusual locations (`C:\Users\`, `C:\Temp\`, `C:\ProgramData\`)
- Hidden or obfuscated configurations
- Encoded commands (Base64, hex, ROT13)

### Step 3: Analyze the Payload
- Check file hashes against VirusTotal
- Examine the binary with static analysis tools
- Review strings in the executable
- Check digital signatures (malware often isn't signed)

### Step 4: Document and Remove
- Screenshot/export the persistence configuration
- Document the technique (map to MITRE ATT&CK)
- Safely remove the persistence mechanism
- Verify removal with re-scan

---

## Defense Recommendations

### Prevention:
1. **Application Whitelisting** - Only allow approved executables (e.g., AppLocker)
2. **Principle of Least Privilege** - Users shouldn't have admin rights
3. **Monitoring** - Enable Sysmon to log registry/WMI/service changes
4. **Group Policy** - Restrict registry write access to Run keys

### Detection:
1. **Autoruns** - Use Sysinternals Autoruns to enumerate all persistence
2. **PowerShell Monitoring** - Log all PowerShell commands (Event ID 4104)
3. **Sysmon Rules** - Create alerts for:
   - New registry Run keys (Event ID 13)
   - New scheduled tasks (Event ID 4698)
   - New WMI subscriptions (Event IDs 19-21)
   - New services (Event ID 7045)
4. **EDR Solutions** - Deploy endpoint detection and response tools

### Response:
1. **Isolate** the affected system from the network
2. **Enumerate** all persistence mechanisms (don't just remove what you found first!)
3. **Collect** forensic artifacts (memory dump, disk image)
4. **Remove** persistence carefully (document before deleting)
5. **Investigate** how the attacker got in (find the initial compromise)

---

## Encoding Techniques Used by Malware

### Base64
**Purpose:** Obfuscate strings to avoid simple signature detection

**Encoding:**
```
Text: SecretMessage
Base64: U2VjcmV0TWVzc2FnZQ==
```

**Decoding:**
```powershell
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("U2VjcmV0TWVzc2FnZQ=="))
```

### ROT13
**Purpose:** Simple Caesar cipher (rotate alphabet by 13)

**Encoding:**
```
Text: }
ROT13: }  (symbols unchanged)

Text: FROST
ROT13: SEBFG
```

**Decoding:** Just apply ROT13 again (it's symmetric)

### Hex Encoding
**Purpose:** Represent bytes as hexadecimal

**Encoding:**
```
Text: flag
Hex: 666c6167
```

**Decoding:**
```python
bytes.fromhex("666c6167").decode()
```

---

## Advanced Persistence Techniques (Beyond This Challenge)

### More Sophisticated Methods:
1. **DLL Hijacking** - Replace legitimate DLLs loaded by trusted processes
2. **COM Hijacking** - Modify COM object registrations
3. **Bootkit/Rootkit** - Infect boot sector or kernel drivers
4. **Startup Folder** - Place shortcuts in `shell:startup`
5. **Browser Extensions** - Persist through browser add-ons
6. **Office Add-ins** - Malicious Word/Excel macros that auto-load
7. **Accessibility Features** - Replace sticky keys, narrator, etc.
8. **Print Providers** - Register malicious print DLLs
9. **LSA Security Providers** - Hook authentication process

### APT Favorite Combinations:
- **APT29**: WMI + Registry + Services (3 redundant mechanisms)
- **APT41**: Scheduled Tasks + Services + DLL Hijacking
- **FIN7**: Registry + COM Hijacking + Browser Extensions

---

## Hands-On Practice: Beyond This Challenge

### Tools to Master:
1. **Sysinternals Suite** (Microsoft)
   - Autoruns - Enumerate all startup locations
   - Process Explorer - Analyze running processes
   - TCPView - Monitor network connections

2. **PowerShell Empire** (Red Team)
   - Practice deploying persistence in lab environments
   - Understand how attackers use these techniques

3. **Velociraptor** (Blue Team)
   - Open-source EDR for incident response
   - Hunt for persistence across multiple systems

4. **OSQuery** (Facebook)
   - SQL-like queries for system forensics
   - Schedule queries to monitor persistence

### Lab Exercises:
1. **Create your own persistence** - Use PowerShell to add all 4 techniques
2. **Hunt your own persistence** - Can you find what you created?
3. **Automate detection** - Write a script to check all 4 locations
4. **Compare tools** - Run Autoruns, PowerShell, and your script - do they all find everything?

---

## Real-World Case Studies

### Case 1: SolarWinds (2020)
**Attacker:** APT29 (Cozy Bear / Russian SVR)

**Persistence Used:**
- Malicious Windows service ("SolarWinds.Orion.Core.BusinessLayer.dll")
- Service survived reboots for MONTHS undetected
- Used legitimate service names to blend in

**Lesson:** Even trusted software can be compromised. Always validate service binaries.

---

### Case 2: NotPetya (2017)
**Attacker:** Sandworm Team (Russian GRU)

**Persistence Used:**
- Scheduled tasks created after initial infection
- Tasks triggered every 10 minutes for ransomware deployment
- Designed to spread rapidly across networks

**Lesson:** Scheduled tasks can execute malicious actions repeatedly, enabling rapid spread.

---

### Case 3: APT29 WMI Attacks (2016)
**Attacker:** APT29 (Cozy Bear)

**Persistence Used:**
- WMI Event Subscriptions (Event Filters + Consumers)
- Triggered every 24 hours to check for C2 commands
- Survived for 6+ months without detection

**Lesson:** WMI persistence is stealthy and often overlooked. Requires specialized tools to detect.

---

## Key Takeaways

1. **Persistence is critical** - Attackers WILL deploy multiple mechanisms
2. **Hunt systematically** - Check ALL locations, not just registry Run keys
3. **Encoding is common** - Be ready to decode Base64, hex, ROT13, etc.
4. **Map to MITRE ATT&CK** - Use T-codes (T1547.001, T1053.005, etc.) for documentation
5. **Automate detection** - Manual checks don't scale; build scripts and use tools
6. **Understand attacker TTPs** - Learn how APT groups use persistence
7. **Practice in labs** - Set up your own persistence and hunt it down
8. **Monitor, monitor, monitor** - Enable logging for registry, WMI, services, and tasks

---

## Resources for Further Learning

### Documentation:
- **MITRE ATT&CK:** https://attack.mitre.org/tactics/TA0003/
- **Microsoft Sysinternals:** https://docs.microsoft.com/en-us/sysinternals/
- **Windows Event Log Reference:** https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/

### Books:
- "Windows Internals" by Russinovich & Solomon
- "The Art of Memory Forensics" by Ligh, Case, Levy, Walters
- "Practical Malware Analysis" by Sikorski & Honig

### Tools:
- Sysinternals Autoruns: https://docs.microsoft.com/en-us/sysinternals/downloads/autoruns
- Sysmon: https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon
- Velociraptor: https://www.velocidex.com/
- OSQuery: https://osquery.io/

### Training:
- SANS FOR508 (Advanced Incident Response)
- SANS SEC504 (Hacker Tools, Techniques, Exploits)
- Cybrary: Windows Forensics courses

---

**You are now equipped to hunt persistence like a professional! 🎯**

*Continue to Day 11 to learn even more advanced techniques! 🎄*
