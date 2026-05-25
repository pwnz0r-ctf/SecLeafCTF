# Hidden_panel

## Challenge Description

> We discovered a partially exposed internal web portal during reconnaissance.
>
> Developers claimed sensitive endpoints were “properly hidden.”
>
> Can you discover what was left behind?
>
> **Challenge Link:** `https://s3.secleaf.tech/`
>
> **Flag Format:** `SecLeaf{}`

---

## Analysis

This challenge focuses on **web reconnaissance** and hidden endpoint discovery.

A very common first step during web enumeration is checking:

```text
/robots.txt
```

The `robots.txt` file often contains:

- Hidden directories
- Administrative paths
- Backup locations
- Developer notes
- Sensitive comments

Many developers mistakenly assume these paths are hidden simply because they are excluded from search engines.

---

## Solution

### Step 1 — Visit robots.txt

Open:

```text
https://s3.secleaf.tech/robots.txt
```

The file contained:

```txt
User-agent: *

Disallow: /admin/
Disallow: /backup/
Disallow: /panel-final/

# temporary developer note:
# SecLeaf{r0b0ts_sh0uldnt_t4lk}
```

---

### Step 2 — Analyze the Contents

Several sensitive directories were exposed:

```txt
/admin/
/backup/
/panel-final/
```

However, the important discovery was the developer comment:

```txt
# SecLeaf{r0b0ts_sh0uldnt_t4lk}
```

The flag was accidentally left inside the `robots.txt` file itself.

---

## Exploitation Steps

### Access robots.txt

```bash
curl https://s3.secleaf.tech/robots.txt
```

or open directly in the browser:

```text
https://s3.secleaf.tech/robots.txt
```

---

### Extract the Flag

Located inside the developer comment:

```txt
# SecLeaf{r0b0ts_sh0uldnt_t4lk}
```

---

## Python Solve Script

```python
import requests
import re

url = "https://s3.secleaf.tech/robots.txt"

response = requests.get(url)

flag = re.search(r"SecLeaf\{.*?\}", response.text)

if flag:
    print(flag.group())
```

---

## Explanation

The challenge demonstrates a classic web security mistake.

Developers often place sensitive paths inside:

```text
robots.txt
```

believing this hides them from users.

However:

- `robots.txt` is publicly accessible
- Search engines can read it
- Attackers always check it during reconnaissance

In this case, the developer accidentally left the flag directly inside a comment.

The title **Hidden_panel** hints toward hidden web endpoints and reconnaissance techniques.

---

## Final Flag

```text
SecLeaf{r0b0ts_sh0uldnt_t4lk}
```

---

## Key Takeaways

- Always check `robots.txt` during web enumeration
- Hidden paths are not secure paths
- Comments inside web files may leak sensitive data
- Security through obscurity is ineffective
- Reconnaissance is one of the most important phases in web security testing
