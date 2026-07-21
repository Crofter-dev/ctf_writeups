# TryHackMe: Shock and Silence

**Category:** Digital Forensics / Ransomware Investigation
**Tooling:** DFIR-Tools suite (NTFS log analysis), partial disk image analysis (.ad1)
**Techniques:** Download artifact tracing, ransomware binary identification, file extension analysis, threat attribution

---

## Overview

The final chapter in the DeceptiTech compromise chain: domain controller **DC-01** gets fully encrypted by ransomware right as an admin attempts routine Group Policy changes. The only available evidence is a partial post-encryption disk image (`DC-01-NTFS-Logs.ad1`) containing NTFS logs. The investigation traces the ransomware's download source, its original filename, the specific executable that triggered encryption, the appended file extension, and ultimately attributes the attack to a known ransomware group.

---

## 1. Setup

Working evidence and tools were provided on the DFIR Analyst's Desktop:
- Disk image: `.\Artifacts\DC-01-NTFS-Logs.ad1`
- Analysis tools: `.\DFIR-Tools\*`

---

## 2. Investigation

### Q1 — Full URL from which the ransomware was downloaded

Traced via download/browser artifacts within the NTFS log analysis.

**Answer:**
```
https://store5.gofile.io/download/web/e23cb33f-0e4d-4a5f-8c55-ea2d78057d40/HiddenFile.zip
```

---

### Q2 — Original file name of the downloaded ransomware executable

**Answer:**
```
pb.exe
```

---

### Q3 — Executable that initiated the encryption process

Identified the process responsible for triggering mass file encryption on the system — notably disguised under a name resembling a legitimate agent/monitoring tool.

**Answer:**
```
HpAgent.exe
```

---

### Q4 — File extension appended to encrypted files

**Answer:**
```
EeUfy
```

---

### Q5 — Ransomware group behind the attack

Required deeper analysis beyond the obvious surface artifacts (extension, ransom note style) to correctly attribute the attack.

**Answer:**
```
BlackLock
```

---

## Key Lesson

This room closes out the DeceptiTech incident chain by showing the final, most damaging stage: full domain controller encryption. The ransomware binary (`pb.exe`, renamed to `HpAgent.exe` — mimicking a legitimate agent process) was traced entirely through NTFS artifact analysis from a partial disk image, without live system access. Attribution to **BlackLock** required correlating multiple indicators rather than relying on any single obvious marker, reinforcing that ransomware naming conventions and extensions alone are rarely sufficient for confident group attribution — cross-referencing TTPs and known IOCs against threat intel is still necessary.
