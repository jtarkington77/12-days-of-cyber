# Day 5 Hints - PowerShell Malware Debugging

**Stuck?** Here are progressive hints to help you debug the PowerShell malware.

---

## Hint 1: Opening PowerShell ISE

<details>
<summary>Click to reveal Hint 1</summary>

### How to Open PowerShell ISE:

**Windows:**
- Press `Win + R`
- Type: `powershell_ise`
- Press Enter

Or:
- Search for "PowerShell ISE" in Start Menu
- Right-click → Run as Administrator (recommended)

**You should see:**
- Script Pane (top) - where you'll load the malware
- Console Pane (bottom) - interactive PowerShell
- Variables Pane (right side) - shows all variable values

If you don't see the Variables pane:
- Click "View" menu → "Show Script Pane Right"

</details>

---

## Hint 2: Loading the Malware Script

<details>
<summary>Click to reveal Hint 2</summary>

### Load `system_update.ps1` in PowerShell ISE:

1. **Open PowerShell ISE** (see Hint 1)

2. **Load the script:**
   - File → Open
   - Navigate to: `day05/artifacts/`
   - Select: `system_update.ps1`

3. **You should see** the obfuscated PowerShell code in the Script Pane

**⚠️ DO NOT press F5 (Run)!**
We'll use F10 (Step Over) to debug safely.

</details>

---

## Hint 3: Understanding the Script Structure

<details>
<summary>Click to reveal Hint 3</summary>

### The Malware Has This Structure:

```powershell
# Lines 14-17: $fp1 (Flag Piece 1) - XOR encoded
# Lines 20-26: $fp2 (Flag Piece 2) - Base64 + Reverse
# Lines 42-49: $fp3 (Flag Piece 3) - Character Replace + Reverse
# Lines 54-56: $fp4 (Flag Piece 4) - Base64

# Lines 59-66: Network C2 beaconing (DANGEROUS - don't execute!)
# Lines 69-76: Registry persistence (DANGEROUS - don't execute!)
# Lines 79-86: Chrome credential theft (DANGEROUS - don't execute!)
```

**Your mission:**
- Debug through the flag piece variables ($fp1, $fp2, $fp3, $fp4)
- Extract the flag from the Variables pane
- STOP before the dangerous network/registry/file operations

</details>

---

## Hint 4: Setting Breakpoints

<details>
<summary>Click to reveal Hint 4</summary>

### Where to Set Breakpoints:

**Option 1: Set breakpoints on each flag piece**
- Line 26: After `$fp1 =` (Flag Piece 1)
- Line 39: After `$fp2 =` (Flag Piece 2)
- Line 49: After `$fp3 =` (Flag Piece 3)
- Line 56: After `$fp4 =` (Flag Piece 4)

**How to set breakpoints:**
1. Click in the left margin next to the line number (red dot appears)
2. Or: Put cursor on the line, press F9

**Option 2: Set a breakpoint before dangerous code**
- Line 59: Before `try {` (network operations)
- This stops you before any dangerous commands execute

**Recommended:** Set breakpoints at ALL flag pieces (lines 26, 39, 49, 56)

</details>

---

## Hint 5: Starting the Debugger

<details>
<summary>Click to reveal Hint 5</summary>

### How to Start Debugging:

**DO NOT USE F5 (Run)!**

Instead, use **F10 (Step Over)**:

1. **Click on Line 1** (the first comment line)
2. **Press F10** - This executes Line 1 and moves to Line 2
3. **Press F10 again** - Executes Line 2, moves to Line 3
4. **Keep pressing F10** - Each press executes one line

### What You'll See:

- **Yellow highlight** shows the current line about to execute
- **Variables pane** updates as variables are created
- **Console pane** shows any output

### The Workflow:

```
Press F10 → Line 14 executes → $x1b9 appears in Variables
Press F10 → Line 15 executes → $q5w appears (partially decoded)
Press F10 → Line 16 executes → $t3r appears
... keep going until $fp1 is set
```

</details>

---

## Hint 6: Reading the Variables Pane

<details>
<summary>Click to reveal Hint 6</summary>

### How to Use the Variables Pane:

As you step through with F10, the Variables pane shows all variables and their current values.

**Example:**
```
Name          Value
----          -----
$x1b9         dGVuLmR1b2xjLWVsb3BodHJvbi5zZXRhZHB1LXRlbnRzb3Jm
$q5w          ten.duolc-elophron.setadpu-tentsorf
$k9m          frostnet-updates.northpole-cloud.net
$fp1          FROST
$fp2          {p0w3rsh3ll_
$fp3          0bfusc@t10n_m@st3
$fp4          r}
```

**What to look for:**
- Variables named `$fp1`, `$fp2`, `$fp3`, `$fp4` - these are your flag pieces!
- As you step through, watch them decode from gibberish to readable text

**Tip:** Click on a variable in the pane to see its full value if it's long.

</details>

---

## Hint 7: Extracting Flag Piece 1 ($fp1)

<details>
<summary>Click to reveal Hint 7</summary>

### Flag Piece 1 is XOR Encoded

**The code (around Line 24-26):**
```powershell
$cfg1 = @(40,60,33,61,58)  # Array of numbers (ASCII values)
$fp1 = -join ($cfg1 | ForEach-Object { [char]($_ -bxor 110) })
```

**Debugging steps:**
1. **Step through** with F10 until Line 26 executes
2. **Check Variables pane** → `$fp1 = "FROST"`
3. **Write it down:** Flag Piece 1 = `FROST`

**How it works:**
- XOR decryption: Each number XORed with 110 gives ASCII chars
- `40 XOR 110 = 70` → 'F'
- `60 XOR 110 = 82` → 'R'
- Result: "FROST"

</details>

---

## Hint 8: Extracting Flag Piece 2 ($fp2)

<details>
<summary>Click to reveal Hint 8</summary>

### Flag Piece 2 is Base64 + Reversed

**The code (around Line 37-39):**
```powershell
$timing_cfg = 'X2xsM2hzcjN3MHB7'  # Base64 encoded
$fp2_temp = [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($timing_cfg))
$fp2_arr = $fp2_temp.ToCharArray()
[array]::Reverse($fp2_arr)
$fp2 = -join $fp2_arr
```

**Debugging steps:**
1. **Step through** with F10 until Line 39 executes
2. **Check Variables pane** → `$fp2 = "{p0w3rsh3ll_"`
3. **Write it down:** Flag Piece 2 = `{p0w3rsh3ll_`

**How it works:**
- Base64 decodes to: `_ll3hsr3w0p{`
- Reverse the string: `{p0w3rsh3ll_`

</details>

---

## Hint 9: Extracting Flag Piece 3 ($fp3)

<details>
<summary>Click to reveal Hint 9</summary>

### Flag Piece 3 is Character Replace + Reversed

**The code (around Line 47-49):**
```powershell
$err_mode = 'ZtsYm_nX1tYcsufbX'  # Obfuscated with character substitution
$fp3_arr = $err_mode.ToCharArray()
[array]::Reverse($fp3_arr)
$fp3_temp = -join $fp3_arr
$fp3 = $fp3_temp.Replace('X','0').Replace('Y','@').Replace('Z','3')
```

**Debugging steps:**
1. **Step through** with F10 until Line 49 executes
2. **Check Variables pane** → `$fp3 = "0bfusc@t10n_m@st3"`
3. **Write it down:** Flag Piece 3 = `0bfusc@t10n_m@st3`

**How it works:**
- Reverse: `XbfuscYt1nX_mYstZ`
- Replace X→0, Y→@, Z→3: `0bfusc@t10n_m@st3`

</details>

---

## Hint 10: Extracting Flag Piece 4 ($fp4)

<details>
<summary>Click to reveal Hint 10</summary>

### Flag Piece 4 is Simple Base64

**The code (around Line 54-56):**
```powershell
$chk = 'cn0='  # Base64 encoded
$fp4 = [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($chk))
```

**Debugging steps:**
1. **Step through** with F10 until Line 56 executes
2. **Check Variables pane** → `$fp4 = "r}"`
3. **Write it down:** Flag Piece 4 = `r}`

**How it works:**
- Base64 decode `cn0=` → `r}`

</details>

---

## Hint 11: Assembling the Complete Flag

<details>
<summary>Click to reveal Hint 11</summary>

### Combine All Four Flag Pieces:

You should now have:
- Flag Piece 1: `FROST`
- Flag Piece 2: `{p0w3rsh3ll_`
- Flag Piece 3: `0bfusc@t10n_m@st3`
- Flag Piece 4: `r}`

**Concatenate them:**
```
FROST + {p0w3rsh3ll_ + [piece3] + [piece4]
= FROST{...}
```

**Your flag:** You'll get the complete flag when you concatenate all four pieces!

**Verify:**
- Starts with `FROST{`
- Ends with `}`
- Contains leetspeak patterns

</details>

---

## Hint 12: Stopping Before Dangerous Code

<details>
<summary>Click to reveal Hint 12</summary>

### What to Do After Getting the Flag:

Once you have all four flag pieces, **STOP DEBUGGING**!

**Press Shift+F5** or **Debug → Stop Debugger**

**Why?**
- Lines 59-66: Network C2 beaconing (`Invoke-WebRequest`)
- Lines 69-76: Registry persistence (`Set-ItemProperty`)
- Lines 79-86: Credential theft (accessing Chrome database)

You don't need to execute these dangerous commands. You already have the flag!

**Bonus (Optional):**
If you're curious what the malware does, keep stepping with F10 through those sections, but the commands will likely fail safely because:
- The C2 domain doesn't exist
- EDR already quarantined the script
- You're in a safe environment

</details>

---

## Hint 13: Alternative Method (No Debugging)

<details>
<summary>Click to reveal Hint 13</summary>

### If PowerShell ISE Doesn't Work:

You can manually decode the flag pieces using PowerShell commands or CyberChef:

**Flag Piece 1 (XOR):**
```powershell
$cfg1 = @(40,60,33,61,58)
-join ($cfg1 | ForEach-Object { [char]($_ -bxor 110) })
# Output: FROST
```

**Flag Piece 2 (Base64 + Reverse):**
- CyberChef: From Base64 (`X2xsM2hzcjN3MHB7`) → Reverse → `{p0w3rsh3ll_`

**Flag Piece 3 (Reverse + Replace):**
- Reverse: `ZtsYm_nX1tYcsufbX` → `XbfuscYt1nX_mYstZ`
- Replace: X→0, Y→@, Z→3 → `0bfusc@t10n_m@st3`

**Flag Piece 4 (Base64):**
- CyberChef: From Base64 (`cn0=`) → `r}`

**Combine:** Concatenate all four pieces to get the complete flag in `FROST{...}` format!

</details>

---

## Common Issues

### Issue: PowerShell ISE Not Available (Linux/Mac)

**Solution:** Use PowerShell Core (pwsh) with manual decoding:
```bash
pwsh
# Then run the decoding commands from Hint 13
```

Or use CyberChef for all decoding.

### Issue: Variables Pane Not Showing

**Fix:**
- View → "Show Script Pane Right" (or press Ctrl+R)
- Look for the Variables tab in the right pane

### Issue: Accidentally Pressed F5 (Run)

**What happens:**
- Script will try to run all commands
- Network calls will fail (C2 domain doesn't exist)
- File operations might create harmless files

**Fix:**
- Press **Ctrl+C** to stop execution
- Close PowerShell ISE
- Reopen and start over with F10 (Step Over)

### Issue: Script Execution Policy Error

**Error:** `cannot be loaded because running scripts is disabled`

**Fix:**
```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Bypass -Force
```

Then reopen PowerShell ISE.

---

## Learning Checklist

After this challenge, you should understand:
- ✅ How to use PowerShell ISE debugger
- ✅ Setting breakpoints and stepping through code
- ✅ Reading the Variables pane to extract decoded values
- ✅ Common PowerShell obfuscation techniques (Base64, XOR, Reverse, Replace)
- ✅ How malware hides C2 domains, persistence, and credentials
- ✅ Safe malware analysis methodology

---

**Remember:** The goal is to LEARN how PowerShell debugging works, not just get the flag!

Understanding these techniques will help you in:
- Real-world malware analysis
- PowerShell script development
- Security incident response
- Understanding attacker techniques
