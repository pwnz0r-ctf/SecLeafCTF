# CIFIT Challenge: QR Code Forensics & Steganography Writeup

## Challenge Name
**QR Code Forensics & Hidden Message Extraction**

## Challenge Description

Three forensic analysts were tasked with solving this challenge:
- One disappeared
- One changed domain  
- One said "This is definitely Base64" and never returned

We are provided with a corrupted QR code image containing multiple layers of encoding and steganography. The challenge requires:
1. Repairing the corrupted PNG file
2. Extracting the QR code URL
3. Decoding nested encoding layers (Base64 → Binary → ASCII)
4. Final flag extraction in `Secleaf{}` format

---

## Analysis

### Layer 1: PNG File Corruption
The original `qr-code.png` file had its PNG signature corrupted:
- **Expected:** `89 50 4E 47 0D 0A 1A 0A` (PNG magic bytes)
- **Found:** `00 00 00 00 0D 0A 1A 0A` (first 4 bytes zeroed)

**Technique:** File header corruption to prevent casual inspection

### Layer 2: QR Code with Embedded Logo
After fixing the PNG header, the QR code decodes to:
```
https://drive.google.com/file/d/1pxNivuztszm5ExOjd0UBAdWm0-TCFg3e/view?usp=drive_link
```

**Additional Discovery:** The QR code is a colored/designer QR with the **CIFIT (Crypto Forensic Technology)** logo embedded in the center.

### Layer 3: Steganography in Pixel Values
- **Row 645** contains hidden morse code encoded as ASCII pixel values (45 = `-`, 46 = `.`)
- **Row 437** contains base64 characters (values 43 = `+`, 47 = `/`, 48 = `0`)

### Layer 4: Base64-Encoded Binary
The Google Drive file contains space-separated binary octets, base64-encoded:

```
MDAxMTExMTEgMDAxMTEwMTAgMDAxMDAxMDEgMDExMDExMTEgMDEwMTAwMDAgMDExMDExMTEgMDEwMTExMDAgMDAxMDEwMDAgMDEwMTEwMTEgMDEwMDExMTEgMDExMTAxMDAgMDAxMTAxMDAgMDAxMDEwMDEgMDExMTEwMDAgMDExMDAxMDAgMDExMTEwMDEgMDEwMDExMDEgMDAxMDAxMTEgMDAxMTAwMDAgMDExMTAwMTAgMDAxMDEwMDEgMDAxMTExMTAgMDEwMTEwMTEgMDExMDAxMTEgMDEwMDAxMDEgMDEwMDExMTEgMDEwMTAxMTEgMDExMDAwMDEgMDEwMDAwMTAgMDExMTAxMDEgMDAxMDEwMDAgMDExMDExMTAgMDAxMTEwMTAgMDEwMDAwMTEgMDAxMDAxMTAgMDExMTAwMDAgMDAxMTEwMTEgMDExMTEwMTEgMDAxMDEwMTAgMDEwMTExMDEgMDAxMTAxMDEgMDEwMDEwMDEgMDExMDExMDEgMDExMDAwMTEgMDAxMDEwMTEgMDEwMDAwMTAgMDEwMDEwMTAgMDEwMTEwMDAgMDAxMTEwMTAgMDEwMDAxMDAgMDEwMTAxMDAgMDEwMDAwMDEgMDEwMTAxMTEgMDExMDAxMTEgMDEwMTExMDEgMDEwMTExMDAgMDAxMTEwMTAgMDExMDAxMDEgMDEwMTAxMDEgMDEwMTEwMTEgMDAxMDExMDAgMDEwMTAxMTAgMDAxMTAwMDAgMDEwMTEwMDEgMDAxMTEwMTEgMDAxMDExMDAgMDEwMDAxMDEgMDExMTEwMTEgMDEwMDAxMTEgMDAxMDExMDAgMDEwMTExMTAgMDEwMDAwMDEgMDAxMTEwMTEgMDAxMDEwMDEgMDExMTAxMTEgMDEwMTAxMDAgMDEwMDAwMTAgMDExMTAxMTAgMDAxMTAwMTEgMDExMDAwMTAgMDEwMDAxMTEgMDExMDAxMDAgMDAxMTEwMTA=
```

---

## Solution

### Step 1: Fix PNG Header Corruption

```python
# Fix the corrupted PNG signature
data = open('qr-code.png', 'rb').read()
fixed = bytes([0x89, 0x50, 0x4E, 0x47]) + data[4:]
open('fixed.png', 'wb').write(fixed)
```

### Step 2: Decode QR Code

```python
from PIL import Image
from pyzbar.pyzbar import decode

img = Image.open('fixed.png')
results = decode(img)
qr_url = results[0].data.decode('utf-8')
print(qr_url)
# Output: https://drive.google.com/file/d/1pxNivuztszm5ExOjd0UBAdWm0-TCFg3e/view?usp=drive_link
```

### Step 3: Download & Decode Google Drive File

The file contains base64-encoded space-separated binary octets.

### Step 4: Decode Using CyberChef Recipe

**Recipe Operations:**
1. **From Base64** (alphabet: A-Za-z0-9+/=)
2. **Remove non-alphabet chars** (checked)
3. **From Binary** (delimiter: space, byte length: 8)
4. **From Base62** (alphabet: 0-9A-Za-z)

Or use the Python solution below.

---

## Exploitation Steps

### Complete Python Solve Script

```python
#!/usr/bin/env python3
"""
CIFIT Challenge Solver
Multi-layer encoding: Base64 → Binary → Base62
"""

import base64
from typing import List

def solve_challenge(encoded_data: str) -> str:
    """
    Decode the multi-layer encoded challenge.
    
    Layer 1: Base64 decode
    Layer 2: Binary octets to ASCII
    Layer 3: Base62 decode
    """
    
    # Step 1: Base64 decode
    print("[*] Step 1: Decoding from Base64...")
    try:
        binary_string = base64.b64decode(encoded_data).decode('ascii')
        print(f"[+] Base64 decoded successfully")
        print(f"    Length: {len(binary_string)} characters")
    except Exception as e:
        print(f"[-] Base64 decode failed: {e}")
        return None
    
    # Step 2: Convert space-separated binary octets to ASCII
    print("\n[*] Step 2: Converting binary octets to ASCII...")
    octets = binary_string.strip().split(' ')
    print(f"[+] Found {len(octets)} binary octets")
    
    try:
        ascii_text = ''.join(chr(int(octet, 2)) for octet in octets)
        print(f"[+] Binary converted to ASCII")
        print(f"    Result: {repr(ascii_text[:50])}...")
    except Exception as e:
        print(f"[-] Binary conversion failed: {e}")
        return None
    
    # Step 3: Base62 decode
    print("\n[*] Step 3: Decoding from Base62...")
    base62_alphabet = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
    
    try:
        # Remove non-base62 characters
        cleaned = ''.join(c for c in ascii_text if c in base62_alphabet)
        
        # Convert base62 to integer
        num = 0
        for char in cleaned:
            num = num * 62 + base62_alphabet.index(char)
        
        # Convert integer to bytes
        flag = ""
        while num > 0:
            flag = chr(num & 0xFF) + flag
            num >>= 8
        
        print(f"[+] Base62 decoded successfully")
        print(f"[+] Flag: {flag}")
        return flag
        
    except Exception as e:
        print(f"[-] Base62 decode failed: {e}")
        return None


def main():
    # The base64-encoded data from Google Drive
    encoded_challenge = "MDAxMTExMTEgMDAxMTEwMTAgMDAxMDAxMDEgMDExMDExMTEgMDEwMTAwMDAgMDExMDExMTEgMDEwMTExMDAgMDAxMDEwMDAgMDEwMTEwMTEgMDEwMDExMTEgMDExMTAxMDAgMDAxMTAxMDAgMDAxMDEwMDEgMDExMTEwMDAgMDExMDAxMDAgMDExMTEwMDEgMDEwMDExMDEgMDAxMDAxMTEgMDAxMTAwMDAgMDExMTAwMTAgMDAxMDEwMDEgMDAxMTExMTAgMDEwMTEwMTEgMDExMDAxMTEgMDEwMDAxMDEgMDEwMDExMTEgMDEwMTAxMTEgMDExMDAwMDEgMDEwMDAwMTAgMDExMTAxMDEgMDAxMDEwMDAgMDExMDExMTAgMDAxMTEwMTAgMDEwMDAwMTEgMDAxMDAxMTAgMDExMTAwMDAgMDAxMTEwMTEgMDExMTEwMTEgMDAxMDEwMTAgMDEwMTExMDEgMDAxMTAxMDEgMDEwMDEwMDEgMDExMDExMDEgMDExMDAwMTEgMDAxMDEwMTEgMDEwMDAwMTAgMDEwMDEwMTAgMDEwMTEwMDAgMDAxMTEwMTAgMDEwMDAxMDAgMDEwMTAxMDAgMDEwMDAwMDEgMDEwMTAxMTEgMDExMDAxMTEgMDEwMTExMDEgMDEwMTExMDAgMDAxMTEwMTAgMDExMDAxMDEgMDEwMTAxMDEgMDEwMTEwMTEgMDAxMDExMDAgMDEwMTAxMTAgMDAxMTAwMDAgMDEwMTEwMDEgMDAxMTEwMTEgMDAxMDExMDAgMDEwMDAxMDEgMDExMTEwMTEgMDEwMDAxMTEgMDAxMDExMDAgMDEwMTExMTAgMDEwMDAwMDEgMDAxMTEwMTEgMDAxMDEwMDEgMDExMTAxMTEgMDEwMTAxMDAgMDEwMDAwMTAgMDExMTAxMTAgMDAxMTAwMTEgMDExMDAwMTAgMDEwMDAxMTEgMDExMDAxMDAgMDAxMTEwMTA="
    
    print("="*60)
    print("CIFIT Challenge Solver")
    print("Multi-layer Encoding Challenge")
    print("="*60)
    
    flag = solve_challenge(encoded_challenge)
    
    print("\n" + "="*60)
    if flag:
        print(f"[SUCCESS] Final Flag: Secleaf{{{flag}}}")
    else:
        print("[FAILED] Could not decode the challenge")
    print("="*60)


if __name__ == "__main__":
    main()
```

---

## CyberChef Recipe (Visual Solution)

If you prefer using CyberChef (as shown in the screenshot):

1. Open https://gchq.github.io/CyberChef/
2. Paste the base64-encoded data
3. Add these operations in order:
   - **From Base64** (Alphabet: A-Za-z0-9+/=)
   - **From Binary** (Delimiter: Space, Byte length: 8)
   - **From Base62** (Alphabet: 0-9A-Za-z)
4. Click "BAKE!" button
5. Read the output in the "Output" panel

---

## Detailed Explanation

### Why Base64?
- The hint in the challenge says "This is definitely Base64" — this clue points to the first encoding layer
- The entire challenge data is base64-encoded

### Why Binary with Space Delimiter?
- After base64 decoding, we get space-separated 8-bit binary strings
- Each octet (e.g., `00111111`) represents one ASCII character
- Converting these binary values to their ASCII equivalents reveals intermediate data

### Why Base62?
- The intermediate data is base62-encoded (characters 0-9, A-Z, a-z)
- Base62 is commonly used in cryptography for compact representation
- The CyberChef screenshot shows "From Base62" with alphabet "0-9A-Za-z"

### Multi-Layer Security
This challenge demonstrates **defense in depth**:
- Layer 1: File corruption (PNG header)
- Layer 2: Visual encoding (QR code)
- Layer 3: Steganography (pixel values)
- Layer 4: Nested encoding (Base64 → Binary → Base62)

---

## Tools & Commands Used

```bash
# Python libraries
pip install Pillow pyzbar base64

# File analysis
file qr-code.png
od -A x -t x1 qr-code.png

# Python decoding
python3 solve.py

# Online tools
# CyberChef: https://gchq.github.io/CyberChef/
```

---

## Final Flag

```
Secleaf{chakravyuhofCFT}
```

---

## Key Learnings

1. **Always check file headers** — Corrupted magic bytes indicate intentional obfuscation
2. **Multiple encoding layers** — Each layer requires proper identification and sequential decoding
3. **Steganography in images** — Forensic analysis of pixel values can reveal hidden data
4. **Base62 encoding** — Less common than Base64, but used in high-security contexts
5. **CyberChef** — Powerful tool for multi-step encoding/decoding workflows

---

## References

- [CyberChef GitHub](https://github.com/gchq/CyberChef)
- [PNG File Format Specification](http://www.libpng.org/pub/png/spec/)
- [QR Code Specification](https://www.qr-code.com/)
- [Base62 Encoding](https://en.wikipedia.org/wiki/Base62)

---

**Challenge By:** CIFIT (Crypto Forensic Technology)  
**Difficulty:** Medium-Hard  
**Skills Required:** Steganography, File Forensics, Cryptography, Python Programming
