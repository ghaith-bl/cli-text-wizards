# 02 - sort

> My study notes. I cut out the rare options and kept only what I actually reach
> for. If I ever need the exotic stuff, the official docs are linked at the bottom.

---

## The mental model

`sort` orders lines. Three facts I had to lock in first:

1. It reads from a **file**, or from **stdin** if I don't give it a file.
2. It writes the result to **stdout**.
3. It **does not modify the original file** (unless I explicitly tell it to).

```bash
sort [OPTIONS] [FILE]
```

---

## Test file I use in these notes

```
# servers.txt
web-01 120 running
db-02 45 stopped
cache-10 9 running
web-2 300 running
api-3 45 running
```

---

## Cheat sheet

| Option | What it does |
|-----------|-----------------------------------------------|
| `-n` | numeric sort - **needed for any numbers** |
| `-h` | human sizes (2K, 5M, 1G) |
| `-V` | version sort (v1.9 before v1.10) |
| `-r` | reverse order |
| `-u` | sort and drop duplicates |
| `-k2,2` | sort by column 2 |
| `-t:` | set the column separator |
| `-f` | ignore upper/lower case |
| `-o FILE` | write to a file (safe way to sort in place) |
| `-c` | only check whether it's already sorted |

---

## The default sort is TEXT, not numbers

This is the mistake I made first.

```
# nums.txt
10
9
100
2
```

```bash
sort nums.txt
```

```
10
100
2
9
```

That looks broken, but it's correct **as text**: it compares character by
character, and the character `1` comes before `9`.

```bash
sort -n nums.txt
```

```
2
9
10
100
```

**Rule: if I'm sorting numbers, `-n` is not optional.**

---

## Sorting by a column

```bash
sort -k2,2 -n servers.txt
```

```
cache-10 9 running
db-02 45 stopped
api-3 45 running
web-01 120 running
web-2 300 running
```

The one thing worth remembering: **write the column number twice with a comma
between them** (`-k2,2`, not `-k2`). If I write it once, `sort` keeps reading past
that column and I get results I didn't ask for. I just always write it twice.

If the file isn't separated by spaces, set the separator:

```bash
sort -t: -k3,3 -n /etc/passwd   # sort users by UID
sort -t, -k2,2 -n data.csv      # a CSV file
```

---

## Removing duplicates

```bash
sort -u names.txt
```

Sorts and removes duplicates in one step. There's also a separate tool called
`uniq`, but that one only removes duplicates that are **next to each other**, so it
needs a sorted input anyway. For a plain dedupe, `sort -u` is enough.

---

## Sizes and versions

```bash
du -sh /var/log/* | sort -h    # 900K correctly before 2M
```

Without `-h` this sorts as text and puts the sizes in the wrong order.

```
# tags.txt
v1.10.0
v1.9.0
v1.2.0
```

```bash
sort tags.txt      # v1.10.0, v1.2.0, v1.9.0   <- wrong
sort -V tags.txt   # v1.2.0, v1.9.0, v1.10.0   <- right
```

Useful for image tags and release tags.

---

## Gotcha: never redirect a file onto itself

```bash
sort servers.txt > servers.txt   # DESTROYS the file
```

The shell empties the file **before** `sort` even starts, so I lose everything.
The safe way:

```bash
sort -o servers.txt servers.txt
```

---

## Gotcha: locale changes the result

The sort order depends on the system language settings, so the same script can
give different output on two machines. In automation I force byte order:

```bash
LC_ALL=C sort file
```

---

## Snippets I actually use

```bash
# top 5 biggest log directories
du -sh /var/log/* | sort -rh | head -5

# docker containers sorted by name
docker ps --format '{{.Names}}' | sort

# pods with the most restarts first
kubectl get pods --no-headers | sort -k4,4 -rn

# most repeated lines in a log file
sort app.log | uniq -c | sort -rn | head -10
```

That last one is the pattern I see everywhere: **sort, count, sort again by
count, take the top.**

---

## Official docs

- [GNU Coreutils - sort invocation](https://www.gnu.org/software/coreutils/manual/html_node/sort-invocation.html)
- [man7 - sort(1)](https://man7.org/linux/man-pages/man1/sort.1.html)
- [POSIX - sort](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/sort.html)

---

## TL;DR for future me

```
sort never edits the original file
default sort is TEXT -> "10" comes before "9"
numbers      -> -n
sizes        -> -h
versions     -> -V
reverse      -> -r
by column 2  -> -k2,2   (always write the number twice)
dedupe       -> -u
in place     -> sort -o f f    NEVER  sort f > f
```
