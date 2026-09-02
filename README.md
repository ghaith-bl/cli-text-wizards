# Bash Notes - Text Processing

My personal study notes while working through Bash for a Junior DevOps path.
I write them as I learn, so they are notes-to-self rather than a polished tutorial:
short, practical, and heavy on the mistakes I actually made.

Everything here is filtered down to **what I use regularly**. Rare options are left
out on purpose - the official documentation is linked at the end of every file.

---

## Contents

| # | Topic | What it covers |
|---|-------|----------------|
| 01 | [Streams](01-streams.md) | `stdin`, `stdout`, `stderr`, redirection, pipes |
| 02 | [sort](02-sort.md) | ordering lines, numbers vs text, columns, dedupe |
| 03 | [sed](03-sed.md) | replacing text, deleting and printing lines, in-place edits |

---

## How I use these notes

- Each file ends with a **TL;DR** block - that's usually all I need to re-read.
- Every gotcha section is something that actually broke for me once.
- The examples are all Docker / Kubernetes / disk / systemd flavoured, because
  that's where I keep running into them.

---

## Quick reference across all three

```bash
# streams
cmd > out.txt 2> err.txt     # split normal output and errors
echo "problem" >&2           # send a message to stderr from a script
cmd 2>/dev/null              # discard errors

# sort
sort -n nums.txt             # numbers (not text!)
sort -rh sizes.txt           # human sizes, biggest first
sort -k2,2 -n data.txt       # by column 2
sort -u names.txt            # sort and dedupe

# sed
sed 's/old/new/g' file       # replace everything on each line
sed -n '/ERROR/p' app.log    # print only matching lines
sed '/^#/d' app.conf         # drop comment lines
sed -i.bak 's/a/b/g' file    # edit the file, keep a backup
```

---

## Environment

Written and tested on Linux with GNU versions of the tools
(`sed --version`, `sort --version` will say GNU).
Some options such as `sed -i` behave differently on macOS / BSD.

---

## Still to come

- `awk`
- `grep`, `cut`, `uniq`
- `getopts`
- `set -euo pipefail`
