# CTF Writeup: Memory Forensics - Hidden in Plain Sight

## Challenge Information

| Field | Details |
|-------|---------|
| **Challenge Name** | Hidden in Plain Sight |
| **Category** | Memory Forensics |
| **Difficulty** | Beginner |
| **Flag Format** | `SecLeaf{}` |
| **Tools Used** | `strings`, `xxd`, `binwalk`, `exiftool`, Python |

---

## Challenge Description

> A memory dump file (`memory.bin`) has been provided.
> Somewhere inside, the real flag is hidden.
> *"NOTE: The flag you need is hidden in plain sight."*

---

## Analysis

### Step 1 — Identify the File

```bash
file memory.bin
```

**Output:**
memory.bin: data

The file has no recognized format. It starts with the magic header `MEMDUMP_V1.0`,
suggesting a custom/simulated memory dump.

---

### Step 2 — Extract Readable Strings

```bash
strings memory.bin
```

This reveals several interesting artifacts inside the dump:

- Multiple fake flags (decoys)
- SQL injection patterns (`SELECT * FROM users`, `DROP TABLE IF EXISTS`)
- System file references (`/etc/passwd`, `/bin/bash`, `root:x:0:0`)
- SSH public key fragments (`ssh-rsa AAAA`)
- RSA private key headers (`BEGIN RSA PRIVATE KEY`)
- HTTP responses (`HTTP/1.1 200 OK`)
- Authorization headers (`Authorization: Bearer`)
- A password hint: `password: *Glitched*_something_interesting`
- A note: `NOTE: The flag you need is hidden in plain sight.`

---

### Step 3 — Spot the Decoy Flags

The dump contains **multiple fake flags** designed to mislead:
SecLeaf{y0u_f0und_m3_haha_n0pe}       ← FAKE
SecLeaf{alm0st_th3r3_just_k1dd1ng}    ← FAKE
SecLeaf{n1c3_try_th1s_1snt_1t}        ← FAKE
SecLeaf{wr0ng_flag_ag41n}             ← FAKE
SecLeaf{r3ally_th1s_t1me_nope}        ← FAKE

---

### Step 4 — Find the Hidden Flag

The real flag appears **concatenated directly after the last decoy**, with no
separator or whitespace:
SecLeaf{r3ally_th1s_t1me_nope}SecLeaf{1m_R3@ll_Fla9}

The challenge author hid the real flag by appending it to a convincing-looking
fake. A casual reader stops at the first `}` and misses the continuation.

This pattern repeats several times throughout the dump, always in the same form.

---

### Step 5 — Verify with xxd

```bash
xxd memory.bin | grep -A2 "1m_R3"
```

Confirms at hex offset `0x006f60`:
00006f60: 2564 2e53 6563 4c65 6166 7b31 6d5f 5233  %d.SecLeaf{1m_R3
00006f70: 406c 6c5f 466c 6139 7d0a ...             @ll_Fla9}

The bytes `53 65 63 4c 65 61 66 7b 31 6d 5f 52 33 40 6c 6c 5f 46 6c 61 39 7d`
decode directly to `SecLeaf{1m_R3@ll_Fla9}`.

---

### Step 6 — Check with binwalk

```bash
binwalk memory.bin
```

**Output:**
DECIMAL    HEXADECIMAL    DESCRIPTION
3917       0xF4D          OpenSSH RSA public key
5568       0x15C0         OpenSSH RSA public key
...

The SSH key hits are random byte sequences matching the `ssh-rsa AAAA` pattern —
not real embedded files. No additional hidden content was found.

---

## Exploitation Steps

### Manual Method

```bash
# Step 1: Extract all strings
strings memory.bin

# Step 2: Filter for all SecLeaf flags
strings memory.bin | grep -oE 'SecLeaf\{[^}]+\}'

# Step 3: Look for concatenated flags
strings memory.bin | grep 'SecLeaf.*SecLeaf'
```

**Output of Step 3:**
SecLeaf{r3ally_th1s_t1me_nope}SecLeaf{1m_R3@ll_Fla9}

Split on `}S` and take the second flag.

---

## Python Solve Script

```python
import re

def solve(filepath):
    with open(filepath, 'rb') as f:
        data = f.read()

    # Decode bytes, ignoring non-UTF8 characters
    text = data.decode('latin-1')

    # Find all SecLeaf flags
    all_flags = re.findall(r'SecLeaf\{[^}]+\}', text)

    print("[*] All flags found:")
    for flag in all_flags:
        print(f"    {flag}")

    # Find concatenated flags (real flag pattern)
    concatenated = re.findall(r'SecLeaf\{[^}]+\}(SecLeaf\{[^}]+\})', text)

    print("\n[*] Flags found concatenated after a decoy:")
    unique = set(concatenated)
    for flag in unique:
        print(f"    {flag}")

    # The real flag is the one that appears AFTER another flag
    if unique:
        print(f"\n[+] Real flag: {list(unique)[0]}")

if __name__ == "__main__":
    solve("memory.bin")
```

**Script Output:**
[*] All flags found:
SecLeaf{y0u_f0und_m3_haha_n0pe}
SecLeaf{alm0st_th3r3_just_k1dd1ng}
SecLeaf{n1c3_try_th1s_1snt_1t}
SecLeaf{r3ally_th1s_t1me_nope}
SecLeaf{1m_R3@ll_Fla9}
SecLeaf{wr0ng_flag_ag41n}
...
[*] Flags found concatenated after a decoy:
SecLeaf{1m_R3@ll_Fla9}
[+] Real flag: SecLeaf{1m_R3@ll_Fla9}

---

## Key Concepts Explained

### Why So Many Fake Flags?
This is a classic **red herring** technique in CTF memory forensics. The
challenge author floods the binary with convincing-looking flags to make you
give up before finding the real one.

### Why Was It Hidden This Way?
By concatenating the real flag directly after a fake one with no separator,
the `strings` output shows it on the same line. A solver who only looks at
unique flag patterns and stops at the first `}` will miss it completely.

### What Was the Encoding?
There was **no encoding** — the flag was stored as **plaintext ASCII** inside
the binary. The obfuscation was purely positional (hidden by proximity to a
decoy), not cryptographic.

### Lessons Learned
- Always `grep` for **all** instances of a pattern, not just the first
- Check for flags that appear **concatenated** to other flags
- Read the final hint in the file — `"hidden in plain sight"` was literal
- `xxd` and `strings` are sufficient for basic memory forensics challenges

---

## Final Flag
SecLeaf{1m_R3@ll_Fla9}
