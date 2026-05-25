# CTF Writeup: Force Push Won't Save You

## Challenge Description
A developer force-pushed several times before the repository was archived. Sensitive data may still exist somewhere in the project history, and some objects may no longer be referenced.
Flag format: `SecLeaf{}`

---

## Approach

### Step 1 — Extract the Archive
The challenge provided a `.zip` containing a Git repository named `force-push-wont-save-you`. Extracted it and entered the repo directory.

### Step 2 — Enumerate Git History
Running `git log --all --oneline` revealed a suspicious commit chain:
5b33ec1 initial commit
0fbc9b5 temporary env file        ← .env added
b7d5c13 remove sensitive file     ← .env deleted
2877024 urgent cleanup
3859618 final cleanup
a197682 backup before rewrite     ← secrets.txt added (dangling)
04352ce fix
ebcdf22 final

The commit messages themselves told the story — the developer added a `.env`, panicked, deleted it, then tried to rewrite history.

### Step 3 — Check Obvious Locations (Red Herrings)

Checked the most obvious spots first:

| Location | Content |
|---|---|
| `.env` in commit `0fbc9b5` | `SecLeaf{not_the_real_flag}` |
| `secrets.txt` in commit `a197682` | `SecLeaf{still_fake}` |
| Dangling blob in `lost-found/` | `SecLeaf{fake_dangling_flag}` |
| Stash (`git stash show`) | `AWS_SECRET=AKIAFAKEKEY` (also fake) |

The challenge was intentionally seeded with decoys to mislead investigators who stop at the obvious places.

### Step 4 — Go Deeper Into Git Internals

Enumerated every blob object in the repo:

```bash
git cat-file --batch-all-objects --batch-check | grep blob
```

Still only decoy flags. Expanded the search to Git's local config files:

```bash
cat .git/config
cat .git/description
cat .git/info/exclude
```

### Step 5 — Flag Found

The `.git/info/exclude` file — which normally stores local gitignore-style rules — contained the flag.

---

## Key Takeaway

`.git/info/exclude` is a **local-only** file. It's never committed, never pushed, and rarely audited. A developer trying to hide something there would go completely unnoticed by anyone only scanning commit history or tracked files.

**The lesson: "deleting" from Git history isn't enough. The entire `.git/` directory is a forensic goldmine.**

---

## Flag
`SecLeaf{history_was_the_trap}`
