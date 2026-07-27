# TryHackMe: Profiles

**Category:** Memory Forensics / Incident Response
**Tooling:** Volatility (Linux memory analysis)
**Techniques:** Credential extraction, malicious binary hash identification, network connection tracing, cron persistence analysis

---

## Overview

An incident response scenario involving a compromised Linux database server. Only a memory dump (`evidence-1699332676548.zip`) is available for analysis, since the server had already been taken offline before other artifacts could be collected — forcing the investigation to rely entirely on memory forensics.

---

## 2. Investigation

### Q1 — Exposed root password

**Answer:**
```
Ftrccw45PHyq
```

---

### Q2 — MD5 hash of the malicious file found

**Answer:**
```
0511ccaad402d6d13ce801e1e9136ba2
```

---

### Q3 — IP address and port of the malicious actor

**Answer (format: IP:Port):**
```
10.0.2.72:1337
```

---

### Q4 — Full path and inode number of the cronjob file

**Answer (format: filename:inode number):**
```
/var/spool/cron/crontabs/root:131127
```

---

### Q5 — Command found inside the cronjob file

**Answer:**
```
cp /opt/.bashrc /root/.bashrc
```

---

## Key Lesson

This room reinforces that a memory dump alone — even without disk access, live shell access, or logs — can still yield a full incident picture: exposed credentials, a malicious binary's hash for threat intel lookup, the attacker's C2 IP/port, and their persistence mechanism via a root crontab entry. The cronjob command here is a subtle persistence trick — overwriting `/root/.bashrc` from a planted file in `/opt` — rather than a more obvious reverse shell one-liner, showing why cron entries should always be read in full rather than pattern-matched against "obviously malicious" command signatures.
