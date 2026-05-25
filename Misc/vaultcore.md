# vaultcore

## Challenge Description

> We recovered a protected vault executable from an abandoned workstation.
>
> Initial analysis suggests:
>
> - anti-debugging routines
> - payload decryption
> - integrity verification
>
> Can you recover the secure access token?
>
> **Flag Format:** `SecLeaf{}`

---

## Analysis

The challenge provides a binary executable named:

```bash
vaultcore
```

The description tries to make the binary sound heavily protected using:

- Anti-debugging
- Payload decryption
- Integrity checks

However, in many beginner reverse engineering challenges, these are often distractions.

The first step in binary analysis is always:

1. Identify the binary type
2. Extract readable strings
3. Check for packing or obfuscation

---

## Solution

### Step 1 — Identify the Binary Type

Run:

```bash
file vaultcore
```

Output:

```bash
ELF 64-bit LSB shared object, x86-64
```

This confirms it is a Linux ELF executable.

---

### Step 2 — Extract Readable Strings

Use the `strings` command:

```bash
strings vaultcore
```

Important output:

```text
Debugger detected
Secure vault
$Info: This file is packed with the UPX executable packer
```

This tells us the binary is packed using **UPX**.

While continuing to inspect the output, we find:

```text
SecLeaf{str1ngs_1s_4ll_y0u_n33d}
```

The flag was directly embedded inside the binary.

---

### Step 3 — Understanding the Trick

The challenge description mentions:

- anti-debugging
- integrity verification
- decryption

These were mostly distractions.

The real intended solution was simply using:

```bash
strings
```

to extract readable data from the executable.

This is a common beginner reverse engineering challenge pattern.

---

## Exploitation Steps

```bash
# Identify binary type
file vaultcore

# Extract readable strings
strings vaultcore

# Filter only possible flags
strings vaultcore | grep SecLeaf
```

Output:

```text
SecLeaf{str1ngs_1s_4ll_y0u_n33d}
```

---

## Python Solve Script

```python
import re
import subprocess

output = subprocess.check_output(["strings", "vaultcore"]).decode()

flag = re.search(r"SecLeaf\{.*?\}", output)

if flag:
    print(flag.group())
```

---

## Explanation

This challenge focused on basic static binary analysis.

The executable was packed with **UPX**, which can sometimes make reverse engineering harder.

However, the flag itself was still stored as a plain readable string inside the binary.

Using the `strings` utility allowed us to quickly extract all printable strings from the executable and locate the flag immediately.

The challenge teaches an important beginner lesson:

> Always check `strings` before doing complex reverse engineering.

---

## Final Flag

```text
SecLeaf{str1ngs_1s_4ll_y0u_n33d}
```
