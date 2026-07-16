# TryHackMe: Warzone 2

**Category:** SOC / Network Traffic Analysis
**Tooling:** Brim, Wireshark, VirusTotal
**Techniques:** PCAP triage, alert validation, malicious download identification, domain/IP reputation pivoting

---

## Overview

A follow-up SOC alert triage exercise. Two alerts fire — **A Network Trojan was Detected** and **Potential Corporate Privacy Violation** — and the investigation traces a malicious `.cab` file download, identifies its payload, and pivots through VirusTotal to uncover a wider set of related malicious infrastructure hidden in the traffic.

---

## 1. Investigation

### Q1 — Alert signature for "A Network Trojan was Detected"

The alert category search didn't return results directly. Instead, identified the alert by clicking into the alert details for source/destination IP, then searched on the source IP directly to surface the matching signature.

**Answer:**
```
ET MALWARE Likely Evil EXE download from MSXMLHTTP non-exe extension M2
```

---

### Q2 — Alert signature for "Potential Corporate Privacy Violation"

**Answer:**
```
ET POLICY PE EXE or DLL Windows file download HTTP
```

---

### Q3 — IP address that triggered either alert

**Answer (defanged):**
```
185[.]118[.]164[.]8
```

---

### Q4 — Full URI for the malicious downloaded file

Searched traffic involving the IP from Q3 to locate the exact download request.

**Answer (defanged):**
```
awh93dhkylps5ulnq-be[.]com/czwih/fxla[.]php?l=gap1[.]cab
```

---

### Q5 — Name of the payload within the CAB file

Extracted the file hash from the traffic:
```
3769a84dbe7ba74ad7b0b355a864483d3562888a67806082ff094a56ce73bf7e
```
Looked it up in VirusTotal to identify the contained payload.

**Answer:**
```
draw.dll
```

---

### Q6 — User-agent associated with the network traffic

**Answer:**
```
Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 10.0; WOW64; Trident/8.0; .NET4.0C; .NET4.0E)
```

---

### Q7 — Other malicious domains seen in traffic (VirusTotal-flagged, alphabetical order)

**Answer (defanged):**
```
a-zcorner[.]com,knockoutlights[.]com
```

---

### Q8 — IP addresses flagged as "Not Suspicious Traffic" (numerical order)

**Answer (defanged):**
```
64[.]225[.]65[.]166,142[.]93[.]211[.]176
```

---

### Q9 — Malicious domains tied to the first "Not Suspicious" IP (alphabetical order)

Despite the traffic classification label, VirusTotal showed this IP had multiple domains flagged as malicious associated with it — a good reminder that traffic-level "not suspicious" tags don't equal a clean reputation check.

**Answer (defanged):**
```
safebanktest[.]top, tocsicambar[.]xyz, ulcertification[.]xyz
```

---

### Q10 — Malicious domain tied to the second "Not Suspicious" IP

**Answer (defanged):**
```
2partscow[.]top
```

---

## Key Lesson

This room is a strong reminder not to trust IDS traffic classification labels at face value — IPs flagged as **"Not Suspicious Traffic"** still resolved to multiple domains independently flagged malicious by VirusTotal. Alert triage should always layer in threat intel lookups (file hash, IP, and domain reputation) rather than relying solely on the network sensor's own classification, since C2 infrastructure is frequently reused across domains and can evade a single detection signature while still surfacing on reputation databases.
