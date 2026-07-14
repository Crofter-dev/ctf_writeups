# TryHackMe: New Hire Old Artifacts

**Category:** SOC / Digital Forensics
**Tooling:** Splunk
**Techniques:** Log correlation, process tree analysis, Sysmon event parsing, registry/network artifact hunting

---

## Overview

This is a Splunk-based SOC investigation challenge simulating a compromised finance workstation. The goal is to trace an attacker's activity across process execution, network connections, registry modifications, and DLL loads using Sysmon-ingested logs in Splunk.

---

## 1. Initial Access

Added the target IP to the local hosts file and scanned for open ports — found **port 8000** open (Splunk's default web interface port).

---

## 2. Investigation

### Q1 — Web Browser Password Viewer binary

Searched using the keyword `password viewer` and located a screenshot/image artifact in the search results showing the binary path.

**Search approach:** keyword search across index for `password viewer`

**Answer:**
```
C:\Users\FINANC~1\AppData\Local\Temp\11111.exe
```

---

### Q2 — Company name listed for the binary

Found in the binary's metadata/company field, visible in the same result set as Q1.

**Answer:**
```
NirSoft
```

---

### Q3 — Second suspicious binary from the same folder

**Search query:**
```spl
index=* CurrentDirectory="C:\\Users\FINANC~1\\AppData\\Local\\Temp"
```

Reviewed the results to identify the second binary and its original filename metadata.

**Answer (format: file,original file):**
```
IonicLarge.exe,PalitExplorer.exe
```

---

### Q4 — Outbound connections to malicious IP

**Search query:**
```spl
index=* EventCode=3 Image="C:\\Users\\Finance01\\AppData\\Local\\Temp\\IonicLarge.exe" | stats count by DestinationIp
```

`EventCode=3` = Sysmon Network Connection event. Filtering by the malicious binary's image path and aggregating by destination IP surfaced the C2 address.

**Answer (defanged):**
```
2[.]56[.]59[.]42
```

---

### Q5 — Registry key modified by the binary

**Search query:**
```spl
index=* EventCode=13 Image="C:\\Users\\Finance01\\AppData\\Local\\Temp\\IonicLarge.exe" | table TargetObject
```

`EventCode=13` = Sysmon Registry Value Set event. The attacker's binary modified a Windows Defender policy key — a common defense-evasion move.

**Answer:**
```
HKLM\SOFTWARE\Policies\Microsoft\Windows Defender
```

---

### Q6 — Killed processes with deleted binaries

**Search query:**
```spl
index=* taskkill.exe Image="C:\\Windows\\SysWOW64\\taskkill.exe" | table ParentImage Image CommandLine
```

Traced `taskkill.exe` invocations to identify which binaries were terminated and subsequently removed from disk (likely anti-forensic cleanup).

**Answer (format: file,file):**
```
WvmIOrcfsuILdX6SNwIRmGOJ.exe,phcIAmLJMAIMSa9j9MpgJo1m.exe
```

---

### Q7 — Last PowerShell command in the Defender-tampering series

**Search query:**
```spl
index=* EventCode=1 ParentImage="*Powershell.exe*" | table _time ParentImage Image CommandLine
```

`EventCode=1` = Sysmon Process Creation. Sorted by timestamp to find the final command in the attacker's sequence of Windows Defender preference modifications.

**Answer:**
```
powershell WMIC /NAMESPACE:\root\Microsoft\Windows\Defender PATH MSFT_MpPreference call Add ThreatIDDefaultAction_Ids=2147737394 ThreatIDDefaultAction_Actions=6 Force=True
```

---

### Q8 — Four Threat IDs set by the attacker (in execution order)

Derived from reviewing the full sequence of `MSFT_MpPreference` calls in chronological order (from Q7's search results).

**Answer (1st,2nd,3rd,4th):**
```
2147735503,2147737010,2147737007,2147737394
```

---

### Q9 — Additional malicious binary from another AppData location

**Search query:**
```spl
index=* Image="C:\\Users\\Finance01\\AppData\\*" | dedup Image | table Image
```

Deduplicated across all `AppData` execution events to spot a binary running from a less obvious subfolder.

**Answer:**
```
C:\Users\Finance01\AppData\Roaming\EasyCalc\EasyCalc.exe
```

---

### Q10 — DLLs loaded by the binary (alphabetical order)

**Search query:**
```spl
index=* EventCode=7 Image="C:\\Users\\Finance01\\AppData\\Roaming\\EasyCalc\\EasyCalc.exe" | dedup ImageLoaded | table ImageLoaded
```

`EventCode=7` = Sysmon Image/DLL Loaded event.

**Answer:**
```
ffmpeg.dll,nw.dll,nw_elf.dll
```

---

## Key Lesson

This room reinforces a core SOC workflow: pivoting through Sysmon Event Codes (`1` = process creation, `3` = network connection, `7` = image load, `13` = registry set) rather than searching blind. Each event code answers a different investigative question — execution, C2 communication, persistence/loaded modules, and defense evasion via registry tampering — and chaining searches by `Image` path lets you follow one binary's full footprint across the log set.
