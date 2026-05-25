# Chaos Theme

## Challenge Description

> Free points? Yes. But not for everyone equally. The early bird gets the worm and the points. This one's a gift, but the leaderboard won't feel like it.
>
> **Note:** DECAY IS ON

---

# Analysis

This challenge was a simple community/free-points challenge from the CTF.

The important clue was:

```text
DECAY IS ON
```

This means:
- the challenge score decreases as more players solve it
- early solvers receive more points
- later solvers receive fewer points

No exploitation, reversing, cryptography, or forensic analysis was required.

The challenge directly provided the flag.

---

# Solution

The flag was already present in the challenge description.

```text
SecLeaf{free_fr33_free}
```

---

# Exploitation Steps

## Step 1 — Read the challenge description carefully

The challenge explicitly included the flag.

No files, services, or tools were needed.

---

# Tools Used

None

---

# Explanation

This was a classic “free flag” or “community” challenge commonly seen in CTF events.

The goal was simply:
- join the event
- submit the provided flag quickly
- gain maximum points before score decay reduced the value

The phrase:

```text
The early bird gets the worm
```

was a hint toward dynamic scoring/decay.

---

# Final Flag

```text
SecLeaf{free_fr33_free}
```
