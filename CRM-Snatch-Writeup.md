# TryHackMe: CRM Snatch

**Category:** Windows Forensics / Data Exfiltration Investigation
**Tooling:** FTK Imager, EZ Tools (Eric Zimmerman's tools)
**Techniques:** Remote session identification, PowerShell session duration analysis, C2/exfiltration tracing, anti-forensics detection (log wiping, shadow copy deletion)

---

## Overview

An attacker uses Matthew's stolen domain credentials to access `SRV-CRM-01` under cover of a routine "nightly export" distraction, grabs the latest customer CSV export, archives it with a password, and exfiltrates it externally using **Rclone** to a Mega cloud bucket. The attacker attempts to cover their tracks by wiping event logs and deleting shadow copies. The investigation works from a forensic disk snapshot to reconstruct the timeline, identify the masqueraded exfiltration tooling, and confirm the data exposure.

---

## 1. Setup

Resources provided on `C:\Users\Administrator\Desktop`, with the disk snapshot under `.\Image\*` and EZ Tools under `.\EZTools\*`.

---

## 2. Investigation

### Q1 — Domain account used to initiate the remote session

Opened the disk image in **FTK Imager** and reviewed the user account section.

**Answer:**
```
matthew.collins
```

---

### Q2 — Duration (seconds) the attacker's PowerShell session remained active

**Answer:**
```
3455
```

---

### Q3 — Attacker's C2 IP address used for staging and exfiltration

**Answer:**
```
167.172.41.141
```

---

### Q4 — Well-known tool used to exfiltrate the collected data

**Answer:**
```
Rclone
```

---

### Q5 — Obscured password to the attacker-controlled Mega account

**Answer:**
```
yWKgVA7Rv1iIoG-VWAr7NAFbwKHNiMZGNybJ4QybJHtiFg
```

---

### Q6 — Lucas's email address found in the exfiltrated data

Not captured in this pass — worth revisiting with the exfiltrated archive contents directly if the room requires it for completion.

---

## Key Lesson

This room demonstrates a stolen-credential exfiltration scenario built around legitimate remote access (a valid domain account, an active PowerShell session) rather than exploit-based intrusion — meaning detection depends entirely on behavioral and timeline analysis rather than signature-based alerts. The use of **Rclone**, a legitimate cloud-sync tool frequently abused for exfiltration, paired with anti-forensic cleanup (event log wiping, shadow copy deletion), is a common pattern worth recognizing: the absence of expected logs is itself a strong indicator of compromise, not just missing data.
