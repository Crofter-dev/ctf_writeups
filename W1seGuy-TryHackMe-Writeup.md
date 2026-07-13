# TryHackMe: W1seGuy

**Category:** Cryptography
**Difficulty:** Easy
**Techniques:** XOR encryption, known-plaintext attack, brute forcing

---

## Overview

W1seGuy is a cryptography challenge where a remote service XOR-encrypts a flag with a randomly generated 5-character key and challenges you to recover the key. The room ships the server's source code, which is enough to fully break the scheme without any guessing.

---

## 1. Recon — Source Code

The room provides `source-1705339805281.py`, the script running on the remote server:

```python
import random
import socketserver
import socket, os
import string

flag = open('flag.txt','r').read().strip()

def send_message(server, message):
    enc = message.encode()
    server.send(enc)

def setup(server, key):
    flag = 'THM{thisisafakeflag}'
    xored = ""

    for i in range(0,len(flag)):
        xored += chr(ord(flag[i]) ^ ord(key[i%len(key)]))

    hex_encoded = xored.encode().hex()
    return hex_encoded

def start(server):
    res = ''.join(random.choices(string.ascii_letters + string.digits, k=5))
    key = str(res)
    hex_encoded = setup(server, key)
    send_message(server, "This XOR encoded text has flag 1: " + hex_encoded + "\n")
    send_message(server,"What is the encryption key? ")
    key_answer = server.recv(4096).decode().strip()

    try:
        if key_answer == key:
            send_message(server, "Congrats! That is the correct key! Here is flag 2: " + flag + "\n")
            server.close()
        else:
            send_message(server, 'Close but no cigar' + "\n")
            server.close()
    except:
        send_message(server, "Something went wrong. Please try again. :)\n")
        server.close()

class RequestHandler(socketserver.BaseRequestHandler):
    def handle(self):
        start(self.request)

if __name__ == '__main__':
    socketserver.ThreadingTCPServer.allow_reuse_address = True
    server = socketserver.ThreadingTCPServer(('0.0.0.0', 1337), RequestHandler)
    server.serve_forever()
```

**Key takeaways from the source:**
- The key is 5 random characters from `ascii_letters + digits`.
- The flag is XOR'd byte-by-byte against the key, repeating the key as needed (`key[i % len(key)]`).
- The output is hex-encoded before being sent.
- Since every THM flag starts with `THM{`, this is a textbook **known-plaintext attack**: knowing 4 bytes of plaintext against 4 bytes of ciphertext instantly recovers 4 bytes of the key.

---

## 2. Enumeration — Connecting to the Service

```bash
nc 10.129.165.118 1337
```

Server response:

```
This XOR encoded text has flag 1: 360c031649532522034d273c3a2c4d16702d065a232a3c5e580e0837056c1030375d4c103c011f44
What is the encryption key?
```

---

## 3. Exploitation

### Step 1 — Recover the known part of the key

Take the first 4 bytes of the ciphertext and XOR them against `THM{` (`0x54 0x48 0x4D 0x7B`):

| Ciphertext byte | XOR | Plaintext byte | = Key byte |
|---|---|---|---|
| `0x3a` | ^ | `0x54` (`T`) | `0x6e` (`n`) |
| `0x09` | ^ | `0x48` (`H`) | `0x41` (`A`) |
| `0x04` | ^ | `0x4D` (`M`) | `0x49` (`I`) |
| `0x1f` | ^ | `0x7B` (`{`) | `0x64` (`d`) |

Recovered key prefix: **`nAId`**

### Step 2 — Brute-force the final key byte

The key is 5 bytes long, and only the last character is unknown. Loop over `string.ascii_letters + string.digits` (62 possibilities) and keep the candidate that decrypts to a clean, printable string ending in `}`.

```python
import string

cipher_hex = "3a09041f345f20250a302b393d25301a752a0f272f2f3b5725020d300c111c353054311c39061639"
cipher = bytes.fromhex(cipher_hex)
known = b"THM{"

# Step 1: recover known key bytes via known-plaintext attack
key_partial = bytes([c ^ p for c, p in zip(cipher, known)])

# Step 2: brute-force remaining key byte
key_len = 5
charset = string.ascii_letters + string.digits

for ch in charset:
    key = key_partial + ch.encode()
    dec = bytes([b ^ key[i % key_len] for i, b in enumerate(cipher)])
    if dec.endswith(b"}") and all(32 <= b < 127 for b in dec):
        print("Key:", key.decode())
        print("Flag:", dec.decode())
```

**Output:**

```
Key: nAIdD
Flag: THM{p1alntExtAtt4ckcAnr3alLyhUrty0urxOr}
```

### Step 3 — Submit the key

The actual encryption key sent by the server for this session was `nAId` (5th char varies per connection). Submitting it back to the service returns the second flag directly:

```
THM{BrUt3_ForC1nG_XOR_cAn_B3_FuN_nO?}
```

---

## Flags

| Flag | Value |
|---|---|
| Flag 1 (decoded ciphertext) | `THM{p1alntExtAtt4ckcAnr3alLyhUrty0urxOr}` |
| Flag 2 (correct key submitted) | `THM{BrUt3_ForC1nG_XOR_cAn_B3_FuN_nO?}` |

---

## Key Lesson

A repeating-key XOR cipher is only as strong as its resistance to **known-plaintext attacks**. Because the plaintext format (`THM{...}`) was predictable, most of the key was recoverable instantly — the only "brute force" needed was over a single unknown byte, a 62-way search instead of a 62^5 one.
