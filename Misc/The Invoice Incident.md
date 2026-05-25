# The Invoice Incident

## Challenge Description

At 09:14 AM, a finance employee reported receiving an urgent invoice email. Shortly after, suspicious activity began on the host.

We were provided with:

* Mail gateway logs
* Endpoint activity logs

Goal:
Identify the malicious attachment responsible for the compromise and retrieve the flag.

Flag Format:

```text
SecLeaf{}
```

---

# Analysis

The challenge provides two log files:

* `mail_gateway.log`
* `endpoint_events.log`

The investigation starts by checking the mail logs around the reported incident time (**09:14 AM**).

---

## Step 1 — Analyze Mail Logs

Suspicious entry found:

```log
2026-05-02 09:14:03 ALLOW from=billing@micr0soft-support.com to=finance.aarti subject=Pending Invoice Notice attachment=Invoice_April_2026.docm
```

### Why This Looks Suspicious

Several indicators suggest phishing activity:

* Fake sender domain:

  ```text
  micr0soft-support.com
  ```

  The attacker replaced the letter `o` with `0`.

* Urgent invoice-themed email

* Macro-enabled Word document:

  ```text
  .docm
  ```

Macro-enabled Office files are commonly used to deliver malware.

---

# Step 2 — Analyze Endpoint Logs

Searching endpoint activity around the same timestamp reveals:

```log
2026-05-02 09:16:11 host=FIN-WS23 process=WINWORD.EXE file=Invoice_April_2026.docm
```

This confirms the user opened the attachment.

Immediately after:

```log
2026-05-02 09:16:18 host=FIN-WS23 parent=WINWORD.EXE process=powershell.exe cmd=-enc SQBFAFgA
```

### Why This Is Malicious

* Microsoft Word spawned PowerShell
* PowerShell was launched using:

  ```powershell
  -enc
  ```

  which means Base64-encoded commands were used.

Attackers frequently use encoded PowerShell payloads to evade detection.

Then:

```log
2026-05-02 09:16:24 host=FIN-WS23 process=powershell.exe netconn=198.51.100.24:80
```

This confirms:

* outbound network activity
* likely malware download or C2 communication

---

# Solution

The malicious attachment responsible for the compromise is:

```text
Invoice_April_2026.docm
```

---

# Exploitation Steps

## Tools Used

* `grep`
* `cat`
* Any text editor
* Basic log analysis

---

## Step-by-Step

### 1. Search for suspicious emails

```bash
grep "attachment=" mail_gateway.log
```

Output:

```log
2026-05-02 09:14:03 ALLOW from=billing@micr0soft-support.com to=finance.aarti subject=Pending Invoice Notice attachment=Invoice_April_2026.docm
```

---

### 2. Investigate endpoint activity

```bash
grep "WINWORD" endpoint_events.log
```

Output:

```log
2026-05-02 09:16:11 host=FIN-WS23 process=WINWORD.EXE file=Invoice_April_2026.docm
```

---

### 3. Check for spawned malicious processes

```bash
grep "powershell" endpoint_events.log
```

Output:

```log
2026-05-02 09:16:18 host=FIN-WS23 parent=WINWORD.EXE process=powershell.exe cmd=-enc SQBFAFgA
2026-05-02 09:16:24 host=FIN-WS23 process=powershell.exe netconn=198.51.100.24:80
```

This confirms malicious execution from the document.

---

# Python Solve Script

```python
# Simple parser to identify suspicious attachment

with open("mail_gateway.log", "r") as f:
    lines = f.readlines()

for line in lines:
    if ".docm" in line.lower():
        print("[+] Suspicious Attachment Found:")
        print(line.strip())
```

---

# Explanation

## Attack Flow

1. Attacker sends phishing email
2. Victim opens malicious `.docm` file
3. Macro executes PowerShell
4. PowerShell connects to attacker infrastructure

---

## Technique Identified

### Macro-Based Malware Delivery

The `.docm` extension indicates a Microsoft Word document containing macros.

### Encoded PowerShell

The `-enc` argument is commonly used to hide malicious commands.

Example:

```powershell
powershell.exe -enc BASE64_PAYLOAD
```

This is a very common malware delivery technique.

---

# Final Flag

```text
SecLeaf{Invoice_April_2026.docm}
```
