# TryHackMe: Warzone 1

**Category:** SOC / Network Traffic Analysis
**Tooling:** Brim, Wireshark, VirusTotal
**Techniques:** PCAP triage, alert validation, C2 traffic analysis, threat attribution

---

## Overview

This room simulates a Tier 1 SOC analyst shift at an MSSP. Two alerts fire — **Potentially Bad Traffic** and **Malware Command and Control Activity Detected** — and the task is to inspect the provided PCAP, validate whether the alert is a true positive, and trace the full attack chain including all IOCs and dropped files.

---

## 1. Initial Triage

Opened the Zone 1 PCAP in **Brim** and filtered directly for the alert category to confirm the signature that fired.

**Search query:**
```
alert.category == "Malware Command and Control Activity Detected"
```

---

## 2. Investigation

### Q1 — Alert signature for the C2 activity

**Answer:**
```
ET Malware MirrorBlast CnC Activity M3
```

---

### Q2 — Source IP address

**Answer (defanged):**
```
172[.]16[.]1[.]102
```

---

### Q3 — Destination IP address in the alert

**Answer (defanged):**
```
169[.]239[.]128[.]11
```

---

### Q4 — Threat group attributed to the destination IP (via VirusTotal Community tab)

**Answer:**
```
TA505
```

---

### Q5 — Malware family

**Answer:**
```
MirrorBlast
```

---

### Q6 — Majority file type under "Communicating Files" for the malicious domain (VirusTotal)

**Answer:**
```
Windows Installer
```

---

### Q7 — User-agent in the web traffic to the flagged IP

Opened the PCAP in **Wireshark**, filtered by the destination IP, and inspected the HTTP request headers.

**Answer:**
```
REBOL View 2.7.8.3.1
```

---

### Q8 — Two additional IP addresses tied to the attack (numerical order)

Filtered Wireshark for the source IP (`172.16.1.102`) combined with the HTTP protocol filter to trace additional connections made during the infection chain.

**Answer (defanged, numerical order):**
```
185[.]10[.]68[.]235,192[.]36[.]27[.]92
```

---

### Q9 — File names downloaded from the two additional IPs

Searched for both IP addresses in Brim to isolate the file transfer events tied to each.

**Answer (order matches Q8):**
```
filter.msi,10opd3r_load.msi
```

---

### Q10 — Files dropped by the first downloaded file (directory + filenames)

Traced the traffic for `filter.msi` in Wireshark to identify the two files written to disk after execution.

**Answer:**
```
C:\ProgramData\001\arab.bin,C:\ProgramData\001\arab.exe
```

---

### Q11 — Files dropped by the second downloaded file (directory + filenames)

Repeated the same traffic analysis for `10opd3r_load.msi`.

**Answer:**
```
C:\ProgramData\Local\Google\rebol-view-278–3–1.exe,C:\ProgramData\Local\Google\exemple.rb
```

---

## Key Lesson

This room is a good end-to-end example of validating a SOC alert rather than taking it at face value: confirm the signature, pivot to VirusTotal for threat attribution (TA505 / MirrorBlast), then drop into packet-level analysis in Wireshark/Brim to reconstruct the full delivery chain — from initial C2 beacon, to staged `.msi` downloads from secondary infrastructure, to the actual binaries dropped on disk. The use of REBOL (an obscure scripting language) as the payload delivery mechanism is a notable MirrorBlast/TA505 signature worth remembering for future triage.
