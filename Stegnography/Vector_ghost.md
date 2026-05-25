# vector_ghost

## Challenge Description

> A corrupted vector asset was recovered from an internal branding archive.
>
> Something hidden still survives inside the image.
>
> **Flag Format:** `SecLeaf{}`

---

## Analysis

The challenge provides a vector image file.

Since the challenge title is **vector_ghost**, this strongly hints toward:

- SVG/XML analysis
- Hidden metadata
- Embedded comments
- Invisible text
- Encoded strings inside vector layers

SVG files are text-based, so the best approach is to inspect the raw source code directly.

---

## Solution

### Step 1 — Identify the File Type

Run:

```bash
file vector_ghost.svg
```

Output:

```bash
SVG Scalable Vector Graphics image
```

This confirms the file is XML-based and readable as plain text.

---

### Step 2 — Inspect the SVG Source

Open the file using:

```bash
cat vector_ghost.svg
```

or:

```bash
strings vector_ghost.svg
```

or:

```bash
nano vector_ghost.svg
```

---

### Step 3 — Search for Hidden Data

While reviewing the SVG source, look for:

- Hidden comments
- Metadata blocks
- Encoded text
- Invisible layers
- Suspicious strings

Example:

```xml
<!-- U2VjTGVhZnt2ZWN0b3JfZ2hvc3RfaGlkZGVufQ== -->
```

This immediately resembles **Base64 encoding** because:

- It uses only Base64 characters
- Ends with `==`
- Common CTF hiding technique

---

### Step 4 — Decode the Hidden String

Run:

```bash
echo "U2VjTGVhZnt2ZWN0b3JfZ2hvc3RfaGlkZGVufQ==" | base64 -d
```

Output:

```text
SecLeaf{vector_ghost_hidden}
```

---

## Exploitation Steps

```bash
# Check file type
file vector_ghost.svg

# Extract readable content
strings vector_ghost.svg

# View SVG source
cat vector_ghost.svg

# Decode Base64 data
echo "U2VjTGVhZnt2ZWN0b3JfZ2hvc3RfaGlkZGVufQ==" | base64 -d
```

---

## Python Solve Script

```python
import base64

encoded = "U2VjTGVhZnt2ZWN0b3JfZ2hvc3RfaGlkZGVufQ=="

decoded = base64.b64decode(encoded).decode()

print(decoded)
```

---

## Explanation

This challenge relied on inspecting the raw contents of an SVG vector file.

SVG files are XML documents, meaning hidden data can easily be embedded inside:

- Comments
- Metadata
- Hidden text objects
- Unused layers

A Base64-encoded string was hidden inside the file source.

The encoded value:

```text
U2VjTGVhZnt2ZWN0b3JfZ2hvc3RfaGlkZGVufQ==
```

was identified as Base64 due to:

- Character structure
- Padding (`==`)
- Typical CTF encoding pattern

After decoding the string, the flag was revealed.

---

## Final Flag

```text
SecLeaf{vector_ghost_hidden}
```
