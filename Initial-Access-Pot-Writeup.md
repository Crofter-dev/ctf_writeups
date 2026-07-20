# TryHackMe: Initial Access Pot

**Category:** Honeypot Analysis / Incident Response
**Tooling:** Auditd, Bash history analysis, DeceptiPot
**Techniques:** Web brute-force detection, backdoor identification, privilege escalation tracing, malware hash identification

---

## Overview

DeceptiTech deploys a honeypot (**DeceptiPot**) configured to mimic a corporate WordPress blog, exposed to the internet in the DMZ to observe real attacker behavior. The honeypot was misconfigured by the junior analyst who set it up, allowing an attacker to fully compromise it. The investigation traces the attack from initial brute-force login attempts through a backdoored PHP file, privilege escalation via an exposed SSH key, and persistence via malware — all captured through non-standard auditd rules.

---

## 2. Investigation

### Q1 — Web page the attacker attempted to brute force

Found by reviewing the WordPress installation directory.

**Command:**
```bash
ls -la /var/www/html/wordpress
```

**Answer:**
```
/wp-login.php
```

---

### Q2 — Absolute path to the backdoored PHP file

Identified a modified theme file containing injected backdoor code within the active WordPress theme (`blocksy`).

**Answer:**
```
/var/www/html/wordpress/wp-content/themes/blocksy/404.php
```

---

### Q3 — File path that allowed privilege escalation to root

An exposed SSH private key backup file left accessible on the filesystem allowed the attacker to escalate.

**Answer:**
```
/etc/ssh/id_ed25519.bak
```

---

### Q4 — IP scanned after privilege escalation

Reviewed root's bash history for post-escalation activity.

**Command:**
```bash
cat /root/.bash_history
```

**Answer:**
```
172.16.8.216
```

---

### Q5 — MD5 hash of the malware persisting on the host

Identified via auditd's non-standard rule set, which logged the file integrity/execution details of the persistence mechanism.

**Answer:**
```
d6f2d80e78f264aff8c7aea21acb6ca6
```

---

### Q6 — Accessing DeceptiPot in recovery mode

Found the DeceptiPot service and its security key stored in the configuration file within the home directory.

**Command:**
```bash
deceptipot -r Em1lyR0ss_DeCePti!
```

**Answer:**
```
THM{acc3ss_gr4nt3d!}
```

---

## Key Lesson

This room flips the usual investigative angle: instead of analyzing a victim's compromised host, it analyzes a **honeypot** that was deployed with real production-grade weaknesses (an exposed SSH key backup, a live theme file that could be backdoored). The result is a genuine end-to-end attack chain — WordPress login brute force → theme file backdoor → privilege escalation via a leftover key backup → network reconnaissance → persistence — captured entirely by non-standard auditd rules. A strong reminder that even intentionally-exposed decoy systems need the same hardening discipline as production ones, or the honeypot itself becomes a real liability rather than a controlled observation tool.
