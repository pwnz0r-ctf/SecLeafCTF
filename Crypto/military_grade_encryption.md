# Encrypted Military Transmission — Writeup

## Challenge Description

> We intercepted an encrypted military transmission during routine monitoring.  
> Analysts were unable to identify the encryption scheme used.  
> Can you recover the hidden message?

Flag format: `SecLeaf{}`

---

## Analysis

We are given the following encrypted text:

```text
U2VjTGVhZntiNDUzNjRfMXNfbjB0XzNuY3J5cHQxMG59
```

The string contains:

- Uppercase letters
- Lowercase letters
- Numbers

This pattern is commonly seen in **Base64 encoding**.

---

## Decoding

We can decode the string using Linux, Python, or online tools like CyberChef.

### Linux Command

```bash
echo "U2VjTGVhZntiNDUzNjRfMXNfbjB0XzNuY3J5cHQxMG59" | base64 -d
```

### Output

```text
SecLeaf{b45364_1s_n0t_3ncrypt10n}
```

---

## Python Solve Script

```python
import base64

cipher = "U2VjTGVhZntiNDUzNjRfMXNfbjB0XzNuY3J5cHQxMG59"

decoded = base64.b64decode(cipher).decode()

print(decoded)
```

---

## Explanation

This challenge highlights a common misconception:

> Base64 is **encoding**, not encryption.

Encoding can be reversed easily without any secret key.

---

## Final Flag

```text
SecLeaf{b45364_1s_n0t_3ncrypt10n}
```