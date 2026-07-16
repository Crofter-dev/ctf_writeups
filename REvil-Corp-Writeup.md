# TryHackMe: REvil Corp

**Category:** Incident Response / Malware Analysis
**Tooling:** Redline (Mandiant), VirusTotal
**Techniques:** Endpoint triage, ransomware artifact analysis, IOC extraction

---

## Overview

An incident response scenario at "Lockman Group" — an employee's files were suddenly renamed to an unfamiliar extension after opening a downloaded executable. The investigation uses **Redline** (via a pre-collected Mandiant Analysis file) to trace the full ransomware infection chain: initial download, encryption behavior, ransom note artifacts, and malware family attribution.

---

## 1. Setup

Opened the **Mandiant Analysis** file located in the Analysis File folder on the Desktop to begin reviewing the collected host artifacts in Redline.

---

## 2. Investigation

### Q1 — Compromised employee's full name

**Answer:**
```
John Coleman
```

---

### Q2 — Operating system of the compromised host

**Answer:**
```
Windows 7 Home Premium 7601 Service Pack 1
```

---

### Q3 — Malicious executable the user opened

Found in the file download history within Redline.

**Answer:**
```
WinRAR2021.exe
```

---

### Q4 — Full URL used to download the malicious binary

**Answer:**
```
http://192.168.75.129:4748/Documents/WinRAR2021.exe
```

---

### Q5 — MD5 hash of the binary

Retrieved from the file system details section, which lists hash values for all files.

**Answer:**
```
890a58f200dfff23165df9e1b088e58f
```

---

### Q6 — Size of the binary (KB)

**Answer:**
```
164
```

---

### Q7 — Extension the user's files got renamed to

**Answer:**
```
.t48s39la
```

---

### Q8 — Number of files renamed to that extension

Determined via the **Timeline > Timestamp** view, counting file rename events matching the extension.

**Answer:**
```
48
```

---

### Q9 — Full path to the wallpaper changed by the attacker

**Answer:**
```
C:\Users\John Coleman\AppData\Local\Temp\hk8.bmp
```

---

### Q10 — Name of the ransom note left on the Desktop

**Answer:**
```
t48s39la-readme.txt
```

---

### Q11 — File left in the attacker-created "Links for United States" folder

Found under `C:\Users\John Coleman\Favorites\Links for United States\`.

**Answer:**
```
GobiernoUSA.gov.url.t48s39la
```

---

### Q12 — Hidden 0-byte file created on the Desktop

**Answer:**
```
d60dff40.lock
```

---

### Q13 — MD5 hash of the decryptor the user downloaded (unsuccessfully)

**Answer:**
```
f617af8c0d276682fdf528bb3e72560b
```

---

### Q14 — Full URL from the ransom note (free single-file decryption test)

**Answer:**
```
http://decryptor.top/644E7C8EFA02FBB7
```

---

### Q15 — Three names associated with the malware family (alphabetical order)

Required external research via VirusTotal to identify all known aliases for the ransomware family.

**Answer:**
```
REvil,Sodin,Sodinokibi
```

---

## Key Lesson

This room walks through a complete ransomware incident from an IR perspective using Redline's endpoint artifact collection — download history, file system hashes/timestamps, and Timeline correlation are enough to reconstruct the full attack: delivery URL → executed binary → mass file renaming → ransom note and wallpaper change → user's failed decryption attempt. Cross-referencing the ransomware extension and hashes against VirusTotal was necessary to pin the family to REvil/Sodinokibi, since the artifacts alone (a random extension string) don't self-identify the malware family without threat intel enrichment.
