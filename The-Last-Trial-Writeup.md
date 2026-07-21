# TryHackMe: The Last Trial

**Category:** macOS Forensics / Incident Response
**Tooling:** apfs-fuse, sqlite3, mac_apt, plistutil
**Techniques:** Browser history analysis, download artifact recovery, TCC permission analysis, C2 traffic identification, persistence mechanism detection

---

## Overview

While the primary DeceptiTech domain compromise unfolds, a separate, untargeted attack hits Lucas, the lead developer, on his macOS system. Lucas downloaded a fake "AI development tool" trial that turned out to be malware. The investigation uses a mounted APFS disk image to trace the malicious download source, its installer, TCC permission abuse, C2 communication, and its persistence mechanism.

---

## 1. Setup

Mounted the disk image for analysis:

```bash
sudo apfs-fuse -v 4 /home/ubuntu/Lucas_Disk.img /home/ubuntu/mac_mount
sudo su
```

(Optional automated analysis available via `mac_apt`:)
```bash
source /root/mac_apt/venv/bin/activate
cd /root/mac_apt
```

---

## 2. Investigation

### Q1 — Website from which the malicious installer was downloaded

Located Safari's history database within the mounted image and queried for AI-related entries.

**Command:**
```sql
SELECT * FROM history_items WHERE url LIKE '%AI%';
```

**Answer:**
```
developai.thm
```

---

### Q2 — Name of the malicious application's installer

Extracted from Safari's Downloads plist.

**Command:**
```bash
plistutil -i Safari/Downloads.plist
```

**Answer:**
```
DevelopAIInstaller.pkg
```

---

### Q3 — Installation timestamp of the malicious application

**Answer:**
```
2025-07-04 10:09:03
```

---

### Q4 — First TCC permission requested by the application

Queried the TCC (Transparency, Consent, and Control) database for entries related to the malicious app.

**Command:**
```bash
sqlite3 /home/ubuntu/mac_mount/root/Users/lucasrivera/Library/Application\ Support/com.apple.TCC/TCC.db
select * from access where client like '%AI%';
```

**Answer:**
```
kTCCServiceSystemPolicyDesktopFolder
```

---

### Q5 — Full C2 URL used for data exfiltration

Found by grepping the application bundle directly for embedded URLs.

**Command:**
```bash
grep -Eir 'http|https' /home/ubuntu/mac_mount/root/Applications/DevelopAI.app 2>/dev/null
```

**Answer:**
```
http://c7.macos-updatesupport.info:8080
```

---

### Q6 — Persistence mechanism used by the application

Searched for common macOS persistence directories within the user's Library folder.

**Command:**
```bash
find /home/ubuntu/mac_mount/root -type d -name "LaunchAgents"
```

**Answer:**
```
LaunchAgent
```

---

## Key Lesson

This room is a good macOS-specific forensics exercise, showing that Apple's security model doesn't prevent social-engineering-driven compromise: the malware still had to *request* TCC permissions (Desktop folder access first), which is visible and auditable in `TCC.db` — a uniquely macOS artifact with no direct Windows/Linux equivalent. Combined with a classic **LaunchAgent** for persistence and a disguised "AI tool" lure, this reinforces that untargeted, opportunistic compromises (someone just browsing for dev tools) can be just as damaging as a targeted domain-wide attack, and that TCC database review should be a standard step in any macOS IR investigation.
