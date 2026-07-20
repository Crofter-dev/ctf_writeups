# TryHackMe: Lost in RAMslation

**Category:** Memory Forensics / Incident Response
**Tooling:** Volatility 3
**Techniques:** Process tree analysis, command-line reconstruction, shellcode injection detection, lateral movement tracing via network artifacts

---

## Overview

DeceptiTech suffers a full ransomware-style network lockdown with corrupted backups and wiped SIEM data, leaving a single memory dump (`SRV-DMZ-GW-evidence.mem`) from the affected server `SRV-DMZ-GW` as the only evidence source. The investigation uses **Volatility** to trace the initial malicious execution, follow the resulting process chain, identify injected Meterpreter shellcode, and pinpoint lateral movement over RDP.

---

## 1. Setup

Memory dump located at `/home/ubuntu/SRV-DMZ-GW-evidence.mem`. Some Volatility output was prefetched into `/home/ubuntu/out` to speed up analysis; ran remaining Volatility plugins directly from the analyst account with `sudo`.

---

## 2. Investigation

### Q1 — Absolute path to the initial malicious file executed

Searched the prefetched `/out` directory for `ps*` files and filtered for `.dll` references, since threat actors commonly disguise malicious loads behind legitimate-looking Windows processes.

**Command:**
```bash
cat ps* | grep dll
```

**Relevant output:**
```
2928  2100  rundll32.exe  0xa68c45c03080  2  -  0  False  2025-07-02 01:04:39.000000 UTC  N/A  Disabled
\Device\HarddiskVolume1\Windows\System32\rundll32.exe  rundll32.exe  C:\Windows\Tasks\MicrosoftUpdate.dll, RunMe
```

**Answer:**
```
C:\Windows\Tasks\MicrosoftUpdate.dll
```

---

### Q2 — PID assigned to the process executing the initial payload

From the same output above.

**Answer:**
```
2928
```

---

### Q3 — Full command line used to launch initial execution

**Answer:**
```
rundll32.exe C:\windows\tasks\MicrosoftUpdate.dll, RunMe
```

---

### Q4 — Final process in the resulting process chain

Loaded the memory image and traced the full process tree and command-line history:

```bash
sudo vol -f SRV-DMZ-GW-evidence.mem windows.pstree
sudo vol -f SRV-DMZ-GW-evidence.mem windows.cmdline
```

**Answer:**
```
notepad.exe
```

---

### Q5 — First five bytes (hex) of the Meterpreter shellcode injected

Used Volatility's `malfind` plugin to identify suspicious memory regions consistent with code injection.

**Command:**
```bash
sudo vol -f SRV-DMZ-GW-evidence.mem windows.malfind
```

**Answer:**
```
fc4889ce48
```

---

### Q6 — IP address used for lateral movement over port 3389 (RDP)

Used the `netscan` plugin to enumerate all network connections captured in memory, filtering for RDP traffic.

**Command:**
```bash
sudo vol -f SRV-DMZ-GW-evidence.mem windows.netscan | grep 3389
```

**Answer:**
```
172.16.2.9
```

---

## Key Lesson

This room demonstrates why memory forensics is invaluable when disk logs and SIEM data are unreliable or wiped: the entire attack chain — masquerading DLL execution via `rundll32.exe`, process injection into a trusted binary (`notepad.exe`), Meterpreter shellcode identification via `malfind`, and RDP-based lateral movement via `netscan` — was reconstructed purely from a single RAM capture. Threat actors frequently hide payloads behind legitimate system process names (`MicrosoftUpdate.dll`) and inject into benign-looking processes specifically to blend into normal activity, which is exactly why memory analysis catches what static log review misses.
