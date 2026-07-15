# TryHackMe: Investigating with Splunk

**Category:** SOC / Digital Forensics
**Tooling:** Splunk
**Techniques:** Windows Event Log correlation, backdoor account detection, PowerShell logging analysis, encoded command decoding

---

## Overview

This room simulates a compromised Windows environment where an adversary creates a backdoor local user, attempts lateral movement via WMIC, and executes obfuscated PowerShell to reach out to a remote host. The investigation is done entirely through Splunk searches against Windows Event Logs ingested into the `main` index.

---

## 1. Initial Access

Added the target IP to the local hosts file, connected to the Splunk web interface, and began searching the `main` index.

---

## 2. Investigation

### Q1 — Total events ingested in the `main` index

**Search query:**
```spl
index=main
```
Set the time range to **All Time** to capture the full ingested dataset.

**Answer:**
```
12256
```

---

### Q2 — Backdoor username created by the adversary

Windows Security Event ID **4720** logs new user account creation. This required checking Microsoft's Event ID reference externally since it isn't always memorized.

**Search query:**
```spl
index="main" EventID="4720"
```

**Answer:**
```
A1berto
```

---

### Q3 — Registry key updated for the new backdoor user

**Search query:**
```spl
index="main" EventID=13 A1berto
```

`EventID=13` (Sysmon Registry Value Set) revealed the SAM registry hive entry created for the new account.

**Answer:**
```
HKLM\SAM\SAM\Domains\Account\Users\Names\A1berto
```

---

### Q4 — User the adversary was impersonating

Identified by comparing the backdoor username against existing legitimate accounts in the logs — `A1berto` (with a numeral `1`) was crafted to visually mimic a real user.

**Answer:**
```
Alberto
```

---

### Q5 — Command used to add the backdoor user remotely

**Search query:**
```spl
index="main" EventID=4688 A1berto
```

`EventID=4688` logs process creation. Filtering for the backdoor username surfaced the exact WMIC remote-execution command used to create the account on a remote node.

**Answer:**
```
C:\windows\System32\Wbem\WMIC.exe" /node:WORKSTATION6 process call create "net user /add A1berto paw0rd1
```

---

### Q6 — Number of observed login attempts from the backdoor user

Logon events are tracked under **4624** (success) and **4625** (failure).

**Search query:**
```spl
index="main" EventID="4625" OR EventID="4624" A1berto
```

**Answer:**
```
0
```

The backdoor account was created but never actually used to log in during the observed window — a useful distinction between account creation and account usage.

---

### Q7 — Host where malicious PowerShell was executed

Identified early in the investigation while scoping which host showed anomalous process activity.

**Answer:**
```
James.browne
```

---

### Q8 — Number of malicious PowerShell events logged

PowerShell Script Block Logging and Module Logging correspond to Event IDs **4104** and **4103**.

**Search query:**
```spl
index="main" EventID="4104" OR EventID="4103"
```

**Answer:**
```
79
```

---

### Q9 — Full URL contacted by the encoded PowerShell script

Traced through the decoded/logged PowerShell script block content to find the outbound web request target.

**Answer (defanged):**
```
http[://]10[.]10[.]10[.]5/news[.]php
```

---

## Key Lesson

This room highlights how Windows Security and PowerShell Event IDs map directly to different attacker actions: **4720** (account creation), **13** (registry changes tied to that account), **4688** (process creation, catching the remote WMIC command), **4624/4625** (logon activity), and **4104/4103** (PowerShell script block/module logging). Chaining searches by a single indicator — here, the backdoor username `A1berto` — across multiple Event IDs is what ties the full attack chain together, from creation to (attempted) impersonation to C2 callback.
