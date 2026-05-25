# Double Trouble

## Challenge Description

> We intercepted a suspicious encoded transmission during routine monitoring.
>
> Analysts believe the message was processed through multiple transformation layers before being transmitted.
>
> Can you recover the original message?
>
> **Flag Format:** `SecLeaf{}`

---

## Analysis

The challenge provides a file:

```bash
encrypted(1).txt
```

Contents of the file:

```text
526e4a7757584a756333737759544e66655452734d325666616a526d5957
64664d3245776148523166513d3d0a
```

At first glance, the content appears to be:

- hexadecimal encoded data
- possibly layered with additional encodings

The challenge title **Double Trouble** hints that the message was encoded multiple times.

---

## Solution

### Step 1 — Identify Hex Encoding

The string contains only hexadecimal characters:

```text
0-9
a-f
```

This strongly suggests **Hex Encoding**.

Decode it using:

```bash
echo "526e4a7757584a756333737759544e66655452734d325666616a526d595764664d3245776148523166513d3d0a" | xxd -r -p
```

Output:

```text
RnJwWXJuc3swYTNfeTRsM2VfajRmYWdfM2EwaHR1fQ==
```

---

### Step 2 — Identify Base64 Encoding

The new string:

```text
RnJwWXJuc3swYTNfeTRsM2VfajRmYWdfM2EwaHR1fQ==
```

looks like **Base64** because:

- It contains valid Base64 characters
- Ends with `==`
- Common layered encoding technique

Decode it:

```bash
echo "RnJwWXJuc3swYTNfeTRsM2VfajRmYWdfM2EwaHR1fQ==" | base64 -d
```

Output:

```text
FrpYrns{0a3_y4l3e_j4fag_3a0htu}
```

---

### Step 3 — Identify ROT13 Cipher

The decoded text still looks strange:

```text
FrpYrns{0a3_y4l3e_j4fag_3a0htu}
```

However:

```text
FrpYrns
```

looks very similar to:

```text
SecLeaf
```

This is a classic sign of **ROT13** encoding.

ROT13 shifts letters by 13 positions.

Decode it using:

```bash
echo "FrpYrns{0a3_y4l3e_j4fag_3a0htu}" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

Output:

```text
SecLeaf{0n3_l4y3r_w4snt_3n0ugh}
```

---

## Exploitation Steps

```bash
# View encrypted content
cat encrypted\(1\).txt

# Decode Hex
echo "526e4a7757584a756333737759544e66655452734d325666616a526d595764664d3245776148523166513d3d0a" | xxd -r -p

# Decode Base64
echo "RnJwWXJuc3swYTNfeTRsM2VfajRmYWdfM2EwaHR1fQ==" | base64 -d

# Decode ROT13
echo "FrpYrns{0a3_y4l3e_j4fag_3a0htu}" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

---

## Python Solve Script

```python
import base64
import codecs

# Original encrypted data
data = "526e4a7757584a756333737759544e66655452734d325666616a526d595764664d3245776148523166513d3d0a"

# Step 1: Hex Decode
hex_decoded = bytes.fromhex(data).decode().strip()

# Step 2: Base64 Decode
b64_decoded = base64.b64decode(hex_decoded).decode()

# Step 3: ROT13 Decode
flag = codecs.decode(b64_decoded, 'rot_13')

print(flag)
```

---

## Explanation

This challenge used **multiple encoding layers**.

### Layer 1 — Hex Encoding

The original message was first converted into hexadecimal.

### Layer 2 — Base64 Encoding

The hex-decoded content revealed a Base64 string.

### Layer 3 — ROT13 Cipher

The Base64-decoded output still appeared scrambled.

The keyword:

```text
FrpYrns
```

was a major clue because applying ROT13 transforms it into:

```text
SecLeaf
```

After decoding all layers, the original flag was recovered.

---

## Final Flag

```text
SecLeaf{0n3_l4y3r_w4snt_3n0ugh}
```
