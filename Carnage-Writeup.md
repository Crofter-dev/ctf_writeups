# TryHackMe: Carnage

**Category:** SOC / Network Traffic Analysis
**Tooling:** Brim, Wireshark, VirusTotal
**Techniques:** PCAP triage, malicious document delivery analysis, C2 infrastructure identification (Cobalt Strike), DNS/spam traffic correlation

---

## Overview

A phishing email with a weaponized Word document leads Eric Fischer to click "Enable Content," triggering suspicious outbound connections flagged by the endpoint agent. The investigation works from a PCAP to trace the full infection chain: a malicious zip download, multiple staged malware domains, two Cobalt Strike C2 servers, post-infection traffic to a separate domain, an IP-check API call, and unrelated malspam (SMTP) activity observed in the same capture.

---

## 2. Investigation

### Q1 — Date/time of the first HTTP connection to the malicious IP

Opened the PCAP in **Brim** and scrolled through the connection log to locate the first HTTP request.

**Answer:**
```
2021-09-24 16:44:38
```

---

### Q2 — Name of the downloaded zip file

**Answer:**
```
documents.zip
```

---

### Q3 — Domain hosting the malicious zip file

Found via the HTTP request headers in **Wireshark**:

```http
GET /incidunt-consequatur/documents.zip HTTP/1.1
Host: attirenepal.com
```

**Answer:**
```
attirenepal.com
```

---

### Q4 — Name of the file inside the zip (without downloading it)

Inspected the HTTP response body's raw bytes, which included the zip's internal file listing metadata without needing to extract it:

```
PK.........d8S.a../...........chart-1530076591.xlsUT
```

**Answer:**
```
chart-1530076591.xls
```

---

### Q5 — Webserver name hosting the malicious IP

**Answer:**
```
LiteSpeed
```

---

### Q6 — Version of the webserver

**Answer:**
```
PHP/7.2.34
```

---

### Q7 — Three domains involved in delivering malicious files to the victim

Correlated the file hashes (MD5) against **VirusTotal** to confirm all associated malicious domains.

**Answer:**
```
finejewels.com.au, thietbiagt.com, new.americold.com
```

---

### Q8 — Certificate authority for the SSL cert of the first domain (Q7)

**Answer:**
```
GoDaddy
```

---

### Q9 — Two Cobalt Strike server IP addresses (sequential order)

Identified via **Wireshark > Statistics > Conversations**, then confirmed against VirusTotal's Community tab as known Cobalt Strike C2 infrastructure.

**Answer:**
```
185.106.96.158, 185.125.204.174
```

---

### Q10 — Host header for the first Cobalt Strike IP

```http
GET /spfooh/cacerts.crl HTTP/1.1
Host: ocsp.verisign.com
```

**Answer:**
```
ocsp.verisign.com
```

---

### Q11 — Domain name for the first Cobalt Strike IP

**Answer:**
```
survmeter.live
```

---

### Q12 — Domain name for the second Cobalt Strike IP

**Answer:**
```
securitybusinpuff.com
```

---

### Q13 — Domain of the post-infection traffic

**Answer:**
```
maldivehost.net
```

---

### Q14 — First eleven characters sent to the post-infection malicious domain

**Answer:**
```
zLIisQRWZI9
```

---

### Q15 — Length of the first packet sent to the C2 server

**Answer:**
```
281
```

---

### Q16 — Server header for the post-infection malicious domain

**Answer:**
```
Apache/2.4.49 (cPanel) OpenSSL/1.1.1l mod_bwlimited/1.4
```

---

### Q17 — Timestamp of the DNS query for the IP-check domain

The malware queried an external service to check the victim's public IP address.

**Answer (UTC):**
```
2021-09-24 17:00:04
```

---

### Q18 — Domain used for the IP check

**Answer:**
```
api.ipify.org
```

---

### Q19 — First MAIL FROM address in the malspam traffic

Separate, unrelated SMTP activity was also present in the same capture.

**Answer:**
```
farshin@mailfa.com
```

---

### Q20 — Number of SMTP packets observed

**Answer:**
```
1439
```

---

## Key Lesson

This room is a comprehensive end-to-end malware delivery and C2 traffic analysis exercise: a weaponized Office document leads to a staged zip download from one domain, additional payloads from three more domains, and ultimately **two Cobalt Strike beacons** confirmed via VirusTotal community intel rather than signature alone. The malware's use of `api.ipify.org` for victim IP fingerprinting and a distinct post-infection domain (separate from the initial Cobalt Strike infrastructure) illustrates how modern intrusions often span multiple, loosely-connected pieces of infrastructure rather than a single C2 server — and the incidental SMTP/malspam traffic in the same capture is a good reminder that a single PCAP can contain multiple unrelated indicators worth separating carefully rather than assuming everything traces to one campaign.
