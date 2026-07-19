# TryHackMe: IronShade

**Category:** Linux Incident Response / Digital Forensics
**Tooling:** Native Linux CLI (cat, ps, systemctl, grep, dpkg)
**Techniques:** Backdoor account discovery, cron persistence analysis, hidden process detection, log correlation, malicious package identification

---

## Overview

A Linux-based incident response investigation. The scenario involves identifying a backdoor user account, its persistence mechanisms (cron jobs, systemd services), hidden processes running from its home directory, and tracing the attacker's SSH login activity and malicious package installation — all using native Linux forensic commands rather than a GUI tool.

---

## 2. Investigation

### Q1 — Machine ID of the investigated machine

**Command:**
```bash
cat /etc/machine-id
```

**Answer:**
```
dc7c8ac5c09a4bbfaf3d09d399f10d96
```

---

### Q2 — Backdoor user account created on the server

Reviewed `/etc/passwd` and identified the most recently created account as suspicious.

**Command:**
```bash
cat /etc/passwd
```

**Answer:**
```
mircoservice
```

---

### Q3 — Cronjob set up by the attacker for persistence

Checked `/var/spool/cron`, found a `crontabs` directory containing a file named `root`, and inspected it with elevated privileges.

**Answer:**
```
@reboot /home/mircoservice/printer_app
```

---

### Q4 — Suspicious hidden process from the backdoor account

**Command:**
```bash
ps aux -u microservice
```

**Answer:**
```
root         569  0.0  0.0   2364   580 ?        Ss   07:30   0:00 /home/mircoservice/.tmp/.strokes
```

---

### Q5 — Number of processes running from the backdoor account's directory

**Command:**
```bash
ps aux -u mircoservice | grep -i home | grep -i mircoservice
```

**Output:**
```
root         569  0.0  0.0   2364   580 ?        Ss   07:30   0:00 /home/mircoservice/.tmp/.strokes
root         928  0.0  0.0   2496    68 ?        S    07:30   0:00 /home/mircoservice/printer_app
```

**Answer:**
```
2
```

---

### Q6 — Hidden file in memory from the root directory

Found by browsing the root directory directly.

**Answer:**
```
.systmd
```

---

### Q7 — Suspicious services installed on the server (alphabetical order)

**Command:**
```bash
systemctl list-unit-files
```

**Answer:**
```
backup.service, strokes.service
```

---

### Q8 — When the backdoor account was created

Reviewed `auth.log` for the account creation event.

**Answer:**
```
Aug 5 22:05:33
```

---

### Q9 — IP address with multiple SSH connections against the backdoor account

**Command:**
```bash
grep -a ssh /var/log/auth.log* | grep -i mircoservice
```

**Answer:**
```
10.11.75.247
```

---

### Q10 — Number of failed SSH login attempts on the backdoor account

Determined by counting failed authentication entries in `auth.log` tied to the same IP/account.

**Answer:**
```
8
```

---

### Q11 — Malicious package installed on the host

**Command:**
```bash
grep "install" /var/log/dpkg.log
```

**Answer:**
```
pscanner
```

---

### Q12 — Secret code in the metadata of the suspicious package

**Command:**
```bash
dpkg -s pscanner
```

**Answer:**
```
{_tRy_Hack_ME_}
```

---

## Key Lesson

This room is a clean example of Linux host forensics using only built-in tools — no Autopsy, no Splunk, just `/etc/passwd`, `/var/spool/cron`, `ps aux`, `systemctl`, and the standard log files (`auth.log`, `dpkg.log`). The attacker's full footprint (backdoor account → cron persistence → hidden `.tmp` process → disguised systemd services → malicious package) was reconstructable purely from default system logging, reinforcing why baseline logging and package installation auditing matter even without dedicated EDR tooling in place.
