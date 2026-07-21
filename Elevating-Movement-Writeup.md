# TryHackMe: Elevating Movement

**Category:** Windows Forensics / Incident Response
**Tooling:** Windows Event Viewer, EZ Tools (Eric Zimmerman's tools)
**Techniques:** RDP login tracing, binary replacement persistence detection, credential dumping identification, lateral movement analysis, NTLM hash extraction

---

## Overview

A continuation of the DeceptiTech compromise — with an entry point already secured and Emily's domain credentials stolen, the attacker moves to `SRV-IT-QA` to escalate privileges. The investigation traces the attacker's RDP login, a disguised persistence binary masquerading as a legitimate Sysinternals tool, a credential dumping operation against LSASS, and the resulting lateral movement using stolen domain credentials.

---

## 2. Investigation

### Q1 — When the attacker performed RDP login on the server

Opened **Event Viewer** and searched for **Event ID 1149** (Remote Desktop Services — Network Connection Successful).

**Answer:**
```
2025-06-30 16:33:18
```

---

### Q2 — Full path to the binary replaced for persistence and privesc

Identified a legitimate-looking Sysinternals binary (`Coreinfo64.exe`) that had been swapped for a malicious executable.

**Answer:**
```
C:\Users\emily.ross\Documents\Coreinfo64.exe
```

---

### Q3 — Type/malware family of the replaced binary

**Answer:**
```
Meterpreter
```

---

### Q4 — Full command line used to dump OS credentials

Identified a Procdump-style invocation targeting the LSASS process to extract credentials to a text file.

**Answer:**
```
pcd.exe /accepteula -ma lsass.exe text.txt
```

---

### Q5 — When the attacker performed lateral movement using stolen credentials

**Answer:**
```
2025-06-30 19:47:14
```

---

### Q6 — NTLM hash of matthew.collins' domain password

Extracted from the LSASS dump output referenced in Q4.

**Answer:**
```
eb3d2de2f21b31933fb4a4fd7a7d314d
```

---

## Key Lesson

This room highlights a classic privilege escalation pattern: disguising a Meterpreter payload as a trusted, commonly-whitelisted admin tool (Sysinternals' Coreinfo) to avoid raising suspicion when executed by IT staff. Once running with elevated context, the attacker dumped LSASS to harvest domain credentials (NTLM hashes) and pivoted laterally using them — reinforcing why binary integrity monitoring for known-good tool names matters just as much as monitoring for outright unknown executables, and why LSASS access should always be tightly audited regardless of the process name invoking it.
