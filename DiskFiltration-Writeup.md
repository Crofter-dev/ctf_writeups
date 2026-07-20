# TryHackMe: DiskFiltration

**Category:** Digital Forensics / Insider Threat Investigation
**Tooling:** Autopsy
**Techniques:** USB device forensics, network profile artifacts, archive/password recovery, PDF metadata analysis, magic byte identification, PowerShell history analysis

---

## Overview

An insider threat scenario: a terminated employee, Liam, is suspected of exfiltrating critical company data with the help of an external accomplice. The investigation uses a disk image in Autopsy to trace USB exfiltration, evidence of network-detection evasion via a personal hotspot, a password-protected archive containing exfiltration instructions, and a trail of file activity, deletions, and browser/PowerShell history confirming intent.

---

## 2. Investigation

### Q1 — Serial number of the USB device used for exfiltration

Found under the **USB Devices** section in Autopsy.

**Answer:**
```
2651931097993496666
```

---

### Q2 — Profile name of the personal hotspot used to evade network detection

Found by searching **Profiles** under the network list in Disk Volume 3.

**Answer:**
```
Liam's Iphone
```

---

### Q3 — Name of the zip file copied from the USB for exfiltration instructions

Found under **Archives**.

**Answer:**
```
Shadow_Plan.zip
```

---

### Q4 — Password for the zip file

Located via a hint pointing to a text file containing the password, stored in a folder named "plain text."

**Answer:**
```
Qwerty@123
```

---

### Q5 — Author of the PDF revealing the external accomplice

The PDF (`breach_plan.pdf`) inside the archive had its author listed in the file **Properties**.

**Answer:**
```
Henry
```

---

### Q6 — Correct extension of an extensionless file in the zip

Opened the file in a **hex editor** and identified its true format via magic bytes (the fixed byte sequence every file type begins with).

**Answer:**
```
PNG
```

---

### Q7 — Files Liam searched for in File Explorer (alphabetical order)

**Answer:**
```
Financial, Revenue
```

---

### Q8 — Folder names present on the USB device (alphabetical order)

Found via a hint, located inside the shell bag artifacts folder.

**Answer:**
```
Critical Data TECH THM, Exfiltration Plan
```

---

### Q9 — Last execution time and count for `file_uploader.exe`

Found in the **Data Artifacts > Run Programs** section, tracking execution history for the file the external entity instructed Liam to run.

**Answer (timestamp, execution count):**
```
2025-01-29 11:26:09, 2
```

---

### Q10 — Hidden flag received from the external accomplice

Found hidden inside one of the files within the zip archive.

**Answer:**
```
FLAGT{THM_TECH_DATA}
```

---

### Q11 — Deletion time of "Tax Records.docx"

Traced as Liam's final act before departure.

**Answer:**
```
2025-01-29 11:29:02
```

---

### Q12 — Social media site searched via web browser

Found in **web history** — likely visited to appear unsuspicious to observers.

**Answer:**
```
https://www.facebook.com/
```

---

### Q13 — PowerShell command executed per the accomplice's plan

Found via `ConsoleHost_history.txt`, which logs every command typed in a PowerShell session using PSReadLine.

**Answer:**
```
Get-WmiObject -Class Win32_Share | Select-Object Name, Path
```

---

## Key Lesson

This room reconstructs a full insider-threat exfiltration timeline purely from disk artifacts: USB device history establishes the exfiltration vector, network profile data reveals evasion of monitored infrastructure via a personal hotspot, archive/PDF metadata exposes the external accomplice, and Run Program + PowerShell history artifacts confirm the technical steps taken under the accomplice's direction. Notably, the final PowerShell command (`Get-WmiObject -Class Win32_Share`) enumerates network shares — a reconnaissance step suggesting further lateral movement was likely the intended next stage, reinforcing why PSReadLine history is such a high-value artifact in insider threat cases.
