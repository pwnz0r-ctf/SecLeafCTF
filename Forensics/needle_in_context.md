# CTF Writeup — Forensic Log Recovery

## Challenge Description

> Several debug logs were partially reconstructed during a failed forensic recovery attempt.
>
> Investigators believed some entries may still contain fragments of sensitive data.
>
> The warning: not every recovery artifact is trustworthy.
>
> Key hint — some files are decoys. The challenge tests whether you can separate real signal from fabricated noise.
>
> **Category:** Forensics  
> **Difficulty:** Medium  
> **Flag Format:** `SecLeaf{}`

---

# Reconnaissance

## Step 1 — Inspect the Archive

First, list all files inside the archive:

```bash
unzip -l logs.zip
```

Output summary:

```text
312 files total
```

Most files followed this pattern:

```text
auth_*.log
```

Additionally, several unusual files appeared:

```text
error_29.log
cache_211.log
temp_92.log
debug_183.log
final_77.log
recovered_8.log
archive_150.log
```

There were also several:

```text
notes*.log
```

files.

This immediately suggested that the uniquely named files were likely important.

---

# Investigating the Decoys

## Step 2 — Analyze the auth_*.log Files

Reading the authentication logs revealed fake flags such as:

```text
SecLeaf{temporary_XXXXX}
```

Each file also contained the warning:

```text
entropy mismatch detected
```

This confirms the challenge hint:

> Most recovered artifacts are fake.

The large number of fake flags was intended to distract players.

---

# Metadata Clues

## Step 3 — Inspect the notes*.log Files

The notes files contained messages such as:

```text
metadata corruption confirmed
XOR patterns detected
possible ECB encryption
metadata integrity verified
```

These hints appear important initially, but they are actually red herrings.

The real challenge focuses on identifying the valid fragments among the noise.

---

# Identifying the Real Artifacts

The seven anomalous log files each contained a short hexadecimal string.

## Extracted Data

| File | Raw Line | Hex Value | Decoded |
|------|------|------|------|
| `debug_183.log` | cache fragment: | `5365634c` | `SecL` |
| `error_29.log` | recovered chunk: | `6561667b` | `eaf{` |
| `final_77.log` | orphan data: | `636f6e74` | `cont` |
| `recovered_8.log` | sync failed: | `6578745f` | `ext_` |
| `cache_211.log` | fragment recovered: | `69735f74` | `is_t` |
| `archive_150.log` | possible recovery: | `68655f72` | `he_r` |
| `temp_92.log` | last chunk: | `65616c5f656e656d797d` | `eal_enemy}` |

---

# Decoding the Fragments

Each suspicious value was hexadecimal encoded.

Example:

```bash
echo 5365634c | xxd -r -p
```

Output:

```text
SecL
```

Repeating the process for all files revealed partial flag fragments.

---

# Solving the Ordering Puzzle

At first glance, sorting by filename number seems logical:

```text
8 → 29 → 77 → 92 → 150 → 183 → 211
```

However, this produces:

```text
SecLeaf{context_he_ris_teal_enemy}
```

which clearly does not make grammatical sense.

The trick was understanding the semantic meaning of the fragments.

Correct arrangement:

```text
SecL
+ eaf{
+ cont
+ ext_
+ is_t
+ he_r
+ eal_enemy}
```

Final reconstructed message:

```text
SecLeaf{context_is_the_real_enemy}
```

---

# Exploitation Steps

## List Files

```bash
unzip -l logs.zip
```

## Read Suspicious Logs

```bash
cat debug_183.log
cat error_29.log
cat final_77.log
cat recovered_8.log
cat cache_211.log
cat archive_150.log
cat temp_92.log
```

## Decode Hex Fragments

```bash
echo 5365634c | xxd -r -p
echo 6561667b | xxd -r -p
echo 636f6e74 | xxd -r -p
echo 6578745f | xxd -r -p
echo 69735f74 | xxd -r -p
echo 68655f72 | xxd -r -p
echo 65616c5f656e656d797d | xxd -r -p
```

---

# Python Solve Script

```python
fragments = [
    "5365634c",
    "6561667b",
    "636f6e74",
    "6578745f",
    "69735f74",
    "68655f72",
    "65616c5f656e656d797d"
]

decoded = [bytes.fromhex(x).decode() for x in fragments]

flag = "".join(decoded)

print(flag)
```

Output:

```text
SecLeaf{context_is_the_real_enemy}
```

---

# Explanation

This challenge was designed to test forensic analysis and noise filtering.

Key lessons:

- Large numbers of fake artifacts often indicate deliberate decoys
- Metadata hints may themselves be distractions
- Hex-encoded fragments are common in forensic recovery challenges
- Logical ordering is not always numeric ordering
- Semantic meaning is often the real clue

The challenge intentionally tried to mislead players into:

- chasing fake flags
- focusing on encryption hints
- sorting files numerically

The real solution required identifying valid fragments and reconstructing a meaningful sentence.

---

# Final Flag

```text
SecLeaf{context_is_the_real_enemy}
```

---

# Key Takeaways

- Ignore high-volume decoy files
- Hex-decode suspicious strings
- Validate fragment ordering semantically
- Do not trust metadata hints blindly
- Always separate signal from noise
