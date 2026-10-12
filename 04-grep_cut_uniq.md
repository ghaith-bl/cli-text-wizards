# grep, cut, uniq

## Quick Reference Notes - Daily Use

No long explanations, just ready-to-apply patterns.

---

# grep

Searches for a specific pattern inside a file or command output, and returns only the matching lines.

### 1. Search inside a command's help output

Using

--help

```bash
kubectl --help | grep -i "delete"
```

### 2. Case-insensitive search with line numbers

```bash
grep -in "error" /var/log/syslog
```

---

# cut

Cuts a specific column or set of characters from each line. Needs a clear, fixed delimiter between columns (comma, tab, etc). If the delimiter is variable spacing, use awk instead.

### 1. Cut a specific field from a comma-separated file

```bash
cut -d',' -f2 data.csv
```

### 2. Cut the first N characters from each line

```bash
docker ps --format "{{.ID}}" | cut -c1-8
```

---

# uniq

Removes duplicate lines, but only if the duplicates are adjacent to each other. That's why it almost always comes right after sort.

### 1. Count how many times each value repeats

```bash
awk '{print $1}' access.log | sort | uniq -c
```

### 2. Show only values that appeared more than once

```bash
cat services.txt | sort | uniq -d
```
