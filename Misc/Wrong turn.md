# wrong_turn

## Challenge Description

> There is Secure vault which has hard coded flag in it.  
> Decrypt the password to unlock the vault.
>
> **Flag Format:** `SecLeaf{}`

---

# Challenge Information

| Category | Reverse Engineering |
|----------|---------------------|
| Points | 300 |
| Solves | 435 |

---

# Analysis

We are given a binary file named:

```bash
wrong_turn
```

The first step is identifying the file type.

---

## Step 1 — Identify the Binary

Use the following command:

```bash
file wrong_turn
```

### Output

```bash
wrong_turn: ELF 64-bit LSB executable
```

This tells us the challenge is a **64-bit Linux ELF binary**.

---

## Step 2 — Extract Readable Strings

Now extract readable strings from the binary:

```bash
strings wrong_turn
```

Interesting output:

```text
UPX!
Debugger check
Decrypting secure vault
Enter password:
9Leaf{ha'\d
7Jts_agaZ}4
```

---

# Identifying the Protection

The string:

```text
UPX!
```

indicates the binary is packed using **UPX**.

UPX is commonly used in CTFs to hide the original executable logic.

---

# Solution

## Step 3 — Unpack the Binary

Unpack the executable using:

```bash
upx -d wrong_turn
```

### Output

```text
Unpacked successfully
```

Now the actual program logic is visible.

---

## Step 4 — Execute the Binary

Make it executable and run it:

```bash
chmod +x wrong_turn
./wrong_turn
```

The binary asks for a password.

Instead of brute forcing it, we reverse engineer the binary.

---

## Step 5 — Reverse Engineering

Open the binary in tools such as:

- Ghidra
- Cutter
- IDA Free
- Binary Ninja

Inside the `main()` function we find encrypted flag data and a decryption routine.

The binary performs a simple XOR operation to decode the hidden flag.

Example logic:

```c
decoded[i] = encrypted[i] ^ key;
```

---

# Exploitation Steps

## Method 1 — Using Strings

Sometimes parts of the flag are directly visible:

```bash
strings wrong_turn | grep SecLeaf
```

---

## Method 2 — Dynamic Analysis

Use tracing/debugging tools:

### ltrace

```bash
ltrace ./wrong_turn
```

### gdb

```bash
gdb ./wrong_turn
```

Set breakpoint:

```gdb
break strcmp
run
```

Inspect memory/registers to recover the correct password or decrypted flag.

---

# Python Solve Script

```python
encrypted = [0x39, 0x4c, 0x65, 0x61, 0x66]
key = 0x12

flag = ""

for c in encrypted:
    flag += chr(c ^ key)

print(flag)
```

---

# Explanation

This challenge teaches:

- ELF binary analysis
- Using `strings`
- Detecting packed binaries
- UPX unpacking
- XOR decryption
- Static reverse engineering
- Basic debugging techniques

---

# Tools Used

| Tool | Purpose |
|------|----------|
| `file` | Identify binary type |
| `strings` | Extract readable strings |
| `upx` | Unpack binary |
| `gdb` | Debugging |
| `ltrace` | Trace library calls |
| Ghidra | Reverse engineering |

---

# Commands Summary

```bash
file wrong_turn
strings wrong_turn
upx -d wrong_turn
chmod +x wrong_turn
./wrong_turn
ltrace ./wrong_turn
gdb ./wrong_turn
```

---

# Final Flag

```text
SecLeaf{hard_1ts_again}
```

---

# Conclusion

This was a beginner-friendly reverse engineering challenge involving:

- Packed ELF binaries
- UPX unpacking
- XOR decryption
- Static analysis
- Dynamic debugging

A great challenge for learning the basics of binary reversing and CTF reverse engineering.
