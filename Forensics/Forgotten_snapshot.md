# Forgotten_snapshot

## Challenge Description

> We recovered this image from a damaged backup archive.
>
> Analysts believe the original owner attempted to conceal sensitive information before deletion.
>
> Some image data may have survived recovery.
>
> **Flag Format:** `SecLeaf{}`

---

## Analysis

The challenge provides an image file:

```bash
snapshot.jpg
```

The description hints that hidden information may still exist inside the image.

In forensic and steganography challenges, the first steps usually include:

- Checking metadata
- Inspecting EXIF information
- Extracting hidden comments
- Looking for appended data

JPEG images often contain metadata fields that can store hidden text.

---

## Solution

### Step 1 — Identify the File Type

Run:

```bash
file snapshot.jpg
```

Output:

```bash
snapshot.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, comment: "SecLeaf{metadata_never_lies}", progressive, precision 8, 528x500, components 3
```

---

### Step 2 — Observe the Embedded Comment

The important part is:

```text
comment: "SecLeaf{metadata_never_lies}"
```

The flag was directly embedded inside the JPEG metadata comment section.

---

### Step 3 — Alternative Metadata Inspection

You can also use tools like:

```bash
exiftool snapshot.jpg
```

or:

```bash
strings snapshot.jpg
```

These tools often reveal hidden metadata fields and comments.

---

## Exploitation Steps

```bash
# Identify file type and metadata
file snapshot.jpg

# Extract EXIF metadata
exiftool snapshot.jpg

# Extract readable strings
strings snapshot.jpg
```

---

## Python Solve Script

```python
from PIL import Image

img = Image.open("snapshot.jpg")

print(img.info)
```

Alternative:

```python
import subprocess

output = subprocess.check_output(["file", "snapshot.jpg"]).decode()

print(output)
```

---

## Explanation

This challenge focused on **image metadata forensics**.

JPEG images can store metadata such as:

- Comments
- Camera information
- EXIF tags
- Hidden text

The `file` command itself revealed the embedded JPEG comment:

```text
SecLeaf{metadata_never_lies}
```

No advanced steganography was required.

The challenge teaches an important forensic lesson:

> Always inspect metadata before attempting complex extraction techniques.

---

## Final Flag

```text
SecLeaf{metadata_never_lies}
```
