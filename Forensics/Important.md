# Challenge Name

## Hidden ZIP Inside JPG

---

# Challenge Description

We are given a file named:

```bash
Important.jpg
```

At first glance, it appears to be a normal image file.  
The objective is to analyze the file and recover the hidden flag.

Flag Format:

```text
SecLeaf{}
```

---

# Analysis

The first step in any forensic or steganography challenge is identifying the real file type.

We use the `file` command:

```bash
file Important.jpg
```

Output:

```bash
Important.jpg: Zip archive data, made by v2.0 UNIX, extract using at least v2.0
```

This immediately tells us something important:

- The file extension is `.jpg`
- The actual file type is a ZIP archive
- The file is disguised to look like an image

This is a very common CTF trick where:
- a ZIP archive is renamed as an image
- or hidden data is embedded inside another file

---

# Solution

Since the file is actually a ZIP archive, we can directly extract it using `unzip`.

## Step 1 — Extract the file

```bash
unzip Important.jpg
```

Output:

```bash
Archive:  Important.jpg
inflating: flag.txt
```

A file named `flag.txt` gets extracted.

---

## Step 2 — Read the flag

```bash
cat flag.txt
```

The flag is displayed inside the file.

---

# Exploitation Steps

## Identify the real file type

Tool Used:
- `file`

Command:

```bash
file Important.jpg
```

---

## Extract hidden contents

Tool Used:
- `unzip`

Command:

```bash
unzip Important.jpg
```

---

## Read extracted file

Tool Used:
- `cat`

Command:

```bash
cat flag.txt
```

---

# Tools Used

- `file`
- `unzip`
- `cat`

---

# Explanation

This challenge uses a simple file masquerading technique.

Although the file had a `.jpg` extension, Linux identifies file types using **magic bytes/signatures** instead of extensions.

The `file` command detected the ZIP signature:

```text
PK
```

This confirms that:
- the file is actually a ZIP archive
- it was intentionally renamed to appear as an image

After extracting the archive, the hidden flag file becomes accessible.

---

# Final Flag

```text
SecLeaf{REPLACE_WITH_ACTUAL_FLAG}
```
