# 🛡️ Analyzing Windows Registry for Evidence of Malicious Activity

A hands-on memory forensics project using Volatility 3 to extract Windows Registry hives from a memory image and identify evidence of malicious activity.

---

## 🎯 Project Overview

The Windows Registry is a database built into every Windows computer that stores settings for the operating system, installed programs, and user activity. When something malicious happens on a machine, it almost always leaves traces in the registry.

In this project, we use **Volatility 3** to analyze a memory image from the **MemLabs Lab 1** challenge. We extract registry hives directly from RAM, identify user accounts, check for persistence mechanisms, and uncover suspicious activity — all without touching the original machine.

---

## 📋 Pre-requisites

- Basic familiarity with Windows and Command Prompt
- A Windows machine (physical or virtual)
- Python 3 installed and added to PATH

---

## 🖥️ Lab Environment

- **OS:** Windows 11 Pro (64-bit) — VirtualBox VM on a Mac host
- **Memory Image:** MemLabs Lab 1 (`MemoryDump_Lab1.raw`)
- **Image Source:** [github.com/stuxnet999/MemLabs](https://github.com/stuxnet999/MemLabs)
- **Working Directory:** `C:\Users\nyco8\OneDrive\Desktop\Volatility\volatility3-develop`

---

## 🧰 Tools Used

| Tool | Purpose |
|---|---|
| **Volatility 3** | Memory forensics framework for extracting and analyzing registry data from a memory image |
| **Python 3.14** | Required runtime for Volatility |
| **Command Prompt** | Used to run all Volatility commands |
| **7-Zip** | Used to extract the compressed memory image |

---

## 🔬 Exercises

### Exercise 1 — Extracting Windows Registry Hives from a Memory Image

**🎯 Objective:** Learn how to locate and extract Windows Registry hives from a memory image using Volatility.

**Steps:**

1. **Install Python** — Download from [python.org/downloads](https://python.org/downloads). During installation, tick **"Add Python to PATH"**.

<p align="center">
  <img src="https://github.com/user-attachments/assets/d222b941-5763-46d9-99fd-1dfb474af93c" alt="Python download page at python.org" width="700"><br>
  <em>Figure 1: Python download page at python.org</em>
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/0f098595-bbf1-4ad6-bd82-51aefe6c5714" alt="Python installer with Add Python to PATH ticked" width="700"><br>
  <em>Figure 2: Python installer — 'Add Python to PATH' must be ticked before installing</em>
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/1492e24a-40bb-47c6-8bea-0d156a01ce2f" alt="Python installation in progress" width="700"><br>
  <em>Figure 3: Python installation in progress</em>
</p>

2. **Download Volatility 3** — Go to [github.com/volatilityfoundation/volatility3](https://github.com/volatilityfoundation/volatility3), click **Code → Download ZIP**, and extract to your Desktop.

<p align="center">
  <img src="https://github.com/user-attachments/assets/837ea230-b646-4df6-bdbd-882762bc3e1a" alt="Volatility 3 GitHub page showing Download ZIP option" width="700"><br>
  <em>Figure 4: Downloading Volatility 3 from GitHub</em>
</p>


3. **Download the memory image** — On the MemLabs GitHub page, open **Lab 1** and click the download link to get `MemLabs-Lab1.7z`. Extract it to `C:\Users\nyco8\OneDrive\Desktop\MemLabs`.


<p align="center">
  <img src="https://github.com/user-attachments/assets/18283bcd-c4cd-42ef-9a83-aa9cf02afdce" alt="MemLabs Lab 1 GitHub page" width="700"><br>
  <em>Figure 5: MemLabs Lab 1 — Beginner's Luck challenge page on GitHub</em>
</p>


<p align="center">
  <img src="https://github.com/user-attachments/assets/f6289550-d0ae-4d8f-8b8b-38065795f5bd" alt="MEGA download page for MemLabs-Lab1.7z" width="700"><br>
  <em>Figure 6: Downloading the memory image from MEGA</em>
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/6a79a08c-1e28-4dc8-a248-c49833a63a2b" alt="Extracting the memory image with 7-Zip" width="700"><br>
  <em>Figure 7: Extracting MemLabs-Lab1.7z to the MemLabs folder</em>
</p>

4. **Verify the image loads** — Run `windows.info` to confirm Volatility can read the image:

```cmd
cd C:\Users\nyco8\OneDrive\Desktop\Volatility\volatility3-develop
python vol.py -f "C:\Users\nyco8\OneDrive\Desktop\MemLabs\MemoryDump_Lab1.raw" windows.info
```

<p align="center">
  <img src="screenshots/placeholder.png" alt="windows.info output confirming the image is readable" width="700"><br>
  <em>Figure 8: windows.info output — Windows 7, captured 2019-12-11</em>
</p>

5. **List all registry hives in memory** — Run `hivelist` to see every hive loaded at the time of the memory snapshot:

```cmd
python vol.py -f "C:\Users\nyco8\OneDrive\Desktop\MemLabs\MemoryDump_Lab1.raw" windows.registry.hivelist
```

<p align="center">
  <img src="screenshots/placeholder.png" alt="hivelist output showing all registry hives in memory" width="700"><br>
  <em>Figure 9: hivelist output — hives found include SAM, SYSTEM, SOFTWARE, and NTUSER.DAT for SmartNet and Alissa Simpson</em>
</p>

6. **Dump the hives to disk** — Save all hive files to an output folder for further analysis:

```cmd
mkdir "C:\Users\nyco8\OneDrive\Desktop\MemLabs\output"
python vol.py -f "C:\Users\nyco8\OneDrive\Desktop\MemLabs\MemoryDump_Lab1.raw" -o "C:\Users\nyco8\OneDrive\Desktop\MemLabs\output" windows.registry.hivelist --dump
```

<p align="center">
  <img src="screenshots/placeholder.png" alt="Creating the output folder in Command Prompt" width="700"><br>
  <em>Figure 10: Creating the output folder</em>
</p>

<p align="center">
  <img src="screenshots/placeholder.png" alt="hivelist --dump output showing all hives saved to disk" width="700"><br>
  <em>Figure 11: All registry hives successfully extracted to the output folder</em>
</p>

---

### Exercise 2 — Analyzing the SAM Hive for User Account Information

**🎯 Objective:** Extract user account information from the SAM hive.

**Steps:**

1. **Extract user account names** — Use `printkey` to read the list of accounts stored in the SAM hive:

```cmd
python vol.py -f "C:\Users\nyco8\OneDrive\Desktop\MemLabs\MemoryDump_Lab1.raw" windows.registry.printkey --key "SAM\Domains\Account\Users\Names"
```

<p align="center">
  <img src="screenshots/placeholder.png" alt="printkey output showing user account names from the SAM hive" width="700"><br>
  <em>Figure 12: User accounts extracted from the SAM hive</em>
</p>

Five accounts were found: **Administrator**, **Alissa Simpson**, **Guest**, **HomeGroupUser$**, and **SmartNet**. Alissa Simpson had the most recent last-write timestamp (2019-12-10), indicating she was the primary active user.

2. **Attempt to extract password hashes** — The `hashdump` plugin used in Volatility 2 was removed in Volatility 3:

```cmd
python vol.py -f "C:\Users\nyco8\OneDrive\Desktop\MemLabs\MemoryDump_Lab1.raw" windows.hashdump
```

<p align="center">
  <img src="screenshots/placeholder.png" alt="Error showing windows.hashdump is not a valid plugin in Volatility 3" width="700"><br>
  <em>Figure 13: windows.hashdump is not available in Volatility 3 — password hash extraction requires an alternative approach</em>
</p>

> **Note:** Hash extraction via `impacket` was attempted but failed due to an OneDrive sync conflict on the Windows VM. This is documented as a known limitation in the Lessons Learned section.

---

### Exercise 3 — Investigating Autorun Entries in the Software Hive

**🎯 Objective:** Identify programs configured to run automatically at startup — a common malware persistence technique.

**Steps:**

1. **Check the Run key** — Read the `Microsoft\Windows\CurrentVersion\Run` key from the SOFTWARE hive:

```cmd
python vol.py -f "C:\Users\nyco8\OneDrive\Desktop\MemLabs\MemoryDump_Lab1.raw" windows.registry.printkey --key "Microsoft\Windows\CurrentVersion\Run"
```

<p align="center">
  <img src="screenshots/placeholder.png" alt="printkey output showing autorun entries in the SOFTWARE hive" width="700"><br>
  <em>Figure 14: Autorun entries in the SOFTWARE hive</em>
</p>

<p align="center">
  <img src="screenshots/placeholder.png" alt="Close-up of the VBoxTray autorun entry" width="700"><br>
  <em>Figure 15: The only autorun entry found — VBoxTray.exe, a legitimate VirtualBox process</em>
</p>

Only one entry was found: **VBoxTray** (`C:\Windows\system32\VBoxTray.exe`) — a legitimate VirtualBox guest additions process. No malicious autorun entries were present. All per-user `Run` keys for SmartNet and Alissa Simpson were empty.

---

### Exercise 4 — Examining UserAssist Keys in the NTUSER.DAT Hive

**🎯 Objective:** Analyze UserAssist keys to determine recently accessed applications and identify suspicious activity.

**Steps:**

1. **Extract UserAssist data** — Run the `userassist` plugin across all user hives:

```cmd
python vol.py -f "C:\Users\nyco8\OneDrive\Desktop\MemLabs\MemoryDump_Lab1.raw" windows.registry.userassist
```

<p align="center">
  <img src="screenshots/placeholder.png" alt="UserAssist output showing recently accessed applications for both users" width="700"><br>
  <em>Figure 16: UserAssist output — recently accessed applications for SmartNet and Alissa Simpson</em>
</p>

**SmartNet — key findings:**

| Application | Count | Last Accessed |
|---|---|---|
| `C:\Users\SmartNet\Desktop\st4G3$$1.bat` | 1 | 2019-12-11 09:02:35 UTC |
| `NOTEPAD.EXE` | 3 | 2019-12-11 09:02:13 UTC |
| `DumpIt.exe` | 4 | 2019-12-11 14:37:54 UTC |
| `cmd.exe` | 5 | 2019-12-11 14:34:54 UTC |

**Alissa Simpson — key findings:**

| Application | Count | Last Accessed |
|---|---|---|
| `WinRAR.exe` | 2 | 2019-12-11 14:37:23 UTC |
| `cmd.exe` | 1 | 2019-12-11 13:18:38 UTC |
| `NOTEPAD.EXE` | 3 | 2019-12-11 10:04:00 UTC |

The most suspicious finding is `st4G3$$1.bat` on SmartNet's Desktop — a batch script with an obfuscated name, executed 22 seconds after Notepad was opened, suggesting it was written in Notepad and then run.

---

### Exercise 5 — Correlating Registry Artifacts for Comprehensive Analysis

**🎯 Objective:** Combine findings from all previous exercises to build a complete picture of activity on this machine.

**Steps:**

1. **Build a unified timeline:**

| Time (UTC) | User | Event |
|---|---|---|
| 2019-12-04 13:26 | SmartNet | Account created |
| 2019-12-04 14:10 | SmartNet | VirtualBox Guest Additions installed |
| 2019-12-10 10:17 | Alissa Simpson | Account last modified |
| 2019-12-11 09:02:13 | SmartNet | Notepad opened |
| 2019-12-11 09:02:35 | SmartNet | ⚠️ `st4G3$$1.bat` executed |
| 2019-12-11 13:18 | Alissa Simpson | `cmd.exe` used |
| 2019-12-11 14:34 | SmartNet | `cmd.exe` activity |
| 2019-12-11 14:37:23 | Alissa Simpson | ⚠️ WinRAR used |
| 2019-12-11 14:37:54 | SmartNet | `DumpIt.exe` — memory captured |

2. **Identify key patterns:**
- **Batch script creation and execution** — Notepad opened at 09:02:13, followed 22 seconds later by execution of `st4G3$$1.bat`. The obfuscated filename is a strong indicator of deliberate concealment.
- **No persistence detected** — The autorun `Run` key was clean, meaning no standard startup persistence was established.
- **Possible data staging** — Alissa Simpson ran WinRAR 31 seconds before the memory dump, suggesting files may have been archived before capture.
- **Deliberate memory capture** — `DumpIt.exe` was the last recorded action on the machine, confirming the memory image was intentionally created by SmartNet.

---

## 🎓 Lessons Learned

### Technical insights
- **Registry keys carry timestamps.** Every key has a Last Write Time. Lining these up across multiple hives gave me a timeline of activity without needing any other logs — I didn't expect that to work as well as it did.
- **UserAssist surprised me.** I didn't know Windows quietly tracks every app you open, how many times, and when. Finding `st4G3$$1.bat` there — executed 22 seconds after Notepad — was the most interesting moment of the whole project.
- **Volatility 3 and Volatility 2 are not interchangeable.** A lot of tutorials I found online still use Volatility 2 commands like `hashdump` or `printkey` without the `windows.registry.` prefix. None of those worked. I learned to always check `python vol.py -h` first to see what's actually available.
- **A clean Run key isn't a green light.** Finding nothing malicious in the autorun entries felt reassuring at first, but persistence can hide in scheduled tasks, services, or DLL hijacking too. The Run key is just the most obvious place to start.

### SOC analyst mindset
- **Always run `hivelist` first.** Before touching any other plugin, the hive list told me which users existed, what accounts were on the machine, and confirmed the OS — all in one command.
- **Individual events mean less than sequences.** One UserAssist entry for Notepad is nothing. But Notepad opening 22 seconds before a suspiciously named batch script runs? That's a sequence worth digging into.
- **Weird filenames are red flags.** `st4G3$$1.bat` doesn't look like anything a normal program would be called. I've learned to treat any filename that mixes random letters, numbers, and symbols as suspicious by default.

### Challenges & how I overcame them
- **`windows.hashdump` doesn't exist in Volatility 3.** I kept getting an invalid plugin error. After some research I found the plugin was dropped from Volatility 3. I tried installing `impacket` as a workaround, but that failed too — pip kept throwing an `OSError: Invalid argument` because OneDrive was syncing the temp folder during the build. I ended up using `windows.registry.printkey` against the SAM hive to pull the usernames directly, which at least got me the account names even without the hashes.
- **Volatility 3 plugin names aren't obvious.** I wasted time guessing names like `windows.hashdump` and `windows.hashdump.Hashdump` before realising I should just run `python vol.py -h` and search the full list. That fixed most of my command errors immediately.

### What I would do differently next time
- **Keep tools off OneDrive.** Installing everything under `C:\Tools\` instead of the Desktop would have avoided the OneDrive sync conflicts that blocked impacket. That cost me more time than I expected.
- **Look inside `st4G3$$1.bat`.** I confirmed it was executed but never recovered what it actually contained. I'd use `windows.filescan` and `windows.dumpfiles` next time to pull the file content directly from memory.
- **Keep Volatility 2 around.** Some plugins like `hashdump` never made it into Volatility 3. Having both versions installed and knowing which supports which would have saved me a lot of dead ends.

---

## 🔗 Related Projects

- 🛡️ [Investigating Windows Event Logs for Security Incidents](https://github.com/)

---

## 📚 References

- [Volatility 3 Documentation](https://volatility3.readthedocs.io/)
- [MemLabs — Memory Forensics CTF Challenges](https://github.com/stuxnet999/MemLabs)
- [Windows Registry Forensics Overview](https://www.forensicfocus.com/)
- [UserAssist Registry Key](https://www.nirsoft.net/utils/userassist_view.html)
