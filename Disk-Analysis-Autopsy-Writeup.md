# TryHackMe: Disk Analysis & Autopsy

**Category:** Digital Forensics
**Tooling:** Autopsy
**Techniques:** E01 disk image triage, OS artifact extraction, user activity reconstruction, file recovery

---

## Overview

This room provides an E01 disk image to investigate using **Autopsy**. The goal is to profile the compromised machine (OS metadata, user accounts, network config), then trace user activity, discovered hacking tools, and evidence of privilege escalation attempts left behind on the system.

---

## 1. Setup

Loaded the provided E01 image into Autopsy as a new case/data source and let it run its standard ingest modules before starting the investigation.

---

## 2. Investigation

### Q1 — MD5 hash of the E01 image

Found in the image's metadata section on the disk's main details page.

**Answer:**
```
3f08c518adb3b5c1359849657a9b2079
```

---

### Q2 — Computer account name

Located under **Extracted Content > Operating System Information**.

**Answer:**
```
DESKTOP-0R59DJ3
```

---

### Q3 — All user accounts (alphabetical order)

Found directly below OS info, under **Operating System User Accounts**.

**Answer:**
```
H4S4N,joshwa,keshav,sandhya,shreya,sivapriya,srini,suba
```

---

### Q4 — Last user to log into the computer

Determined from the most recent logon timestamp among the user account artifacts.

**Answer:**
```
sivapriya
```

---

### Q5 — IP address of the computer

Found in the **Network Monitoring** section, under Volume 3's program files.

**Answer:**
```
192.168.130.216
```

---

### Q6 — MAC address of the computer

Retrieved alongside the network configuration artifacts.

**Answer (format: XX-XX-XX-XX-XX-XX):**
```
0800272cc4b9
```

---

### Q7 — Name of the network card

Found under **OS Info > Software > Microsoft > Network Card** section.

**Answer:**
```
Intel(R) PRO/1000 MT Desktop Adapter
```

---

### Q8 — Name of the network monitoring tool

Identified from installed program artifacts (matches the tool referenced in Q5's network monitoring data).

**Answer:**
```
Look@LAN
```

---

### Q9 — Coordinates of a bookmarked Google Maps location

Recovered from browser bookmark/history artifacts.

**Answer:**
```
12°52'23.0"N 80°13'25.0"E
```

---

### Q10 — Full name printed on a user's desktop wallpaper

Found by browsing **Images and Videos**, specifically inside a user folder (`joshua`) with an image in Downloads showing the full name.

**Answer:**
```
Anto Joshwa
```

---

### Q11 — First flag on a user's desktop file (later changed via PowerShell)

Recovered the original file content/version before the PowerShell modification, found via file history/deleted file recovery on the user's desktop.

**Answer:**
```
flag{HarleyQuinnForQueen}
```

---

### Q12 — Message to the device owner from the privilege escalation exploit

Found alongside the exploit artifact used by the same user identified in Q11.

**Answer:**
```
Flag{I-hacked-you}
```

---

### Q13 — Two password-focused hacking tools found (alphabetical order)

Identified via installed/executed program artifacts on the system.

**Answer:**
```
lazagne,mimikatz
```

---

### Q14 — Author of the YARA file found on the computer

Found under **Recent Documents**, inspecting the YARA rule file's metadata/header.

**Answer:**
```
Benjamin DELPY (gentilkiwi)
```

---

### Q15 — Archive filename for the MS-NRPC (Zerologon) exploit

Found among file artifacts related to domain controller exploitation attempts.

**Answer (include spaces):**
```
2.2.0 20200918 Zerologon encrypted.zip
```

---

## Key Lesson

This room is a solid walkthrough of Autopsy's standard triage flow: start with **OS Information** for machine identity (hostname, users, network config), then branch into **Recent Documents**, **Images/Videos**, and **Web History** for user activity and intent. The presence of tools like Mimikatz, LaZagne, and a Zerologon exploit archive — alongside a YARA rule authored by Benjamin Delpy (Mimikatz's own creator) — paints a picture of a user actively researching and staging credential-theft and domain-escalation tooling rather than just being a victim of it.
