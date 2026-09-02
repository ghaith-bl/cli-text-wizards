# 03 - sed

> My study notes. `sed` can do a lot of things I will probably never need, so I
> kept only the parts I use: replacing text, deleting lines, printing specific
> lines, and editing config files.

---

## The mental model

`sed` = **stream editor**. Instead of opening a file and editing it by hand, I hand
it an editing instruction and it applies that instruction to every line.

For each line it does the same three things:

```
1. read one line into a buffer  (called the "pattern space")
2. run my command on it
3. print the result to stdout, then move to the next line
```

Two facts I had to lock in:

- It reads from a **file**, or from **stdin** if I don't give it a file.
- It **does not modify the original file** unless I pass `-i`.

```bash
sed [OPTIONS] 'SCRIPT' [FILE]
```

---

## Test files I use in these notes

```
# app.conf
host = localhost
port = 8080
debug = true
log_level = info
timeout = 30
```

```
# app.log
2026-01-10 INFO  user login ok
2026-01-10 ERROR db connection failed
2026-01-11 WARN  disk usage 85%
2026-01-11 ERROR timeout on api call
2026-01-12 INFO  backup done
```

---

## The substitute command (90% of what I use)

```
s/PATTERN/REPLACEMENT/FLAGS
│ │       │           │
│ │       │           └── flags: g i p ...
│ │       └────────────── what to write instead
│ └────────────────────── what to search for
└──────────────────────── s = substitute
```

```bash
sed 's/localhost/127.0.0.1/' app.conf
```

---

## Cheat sheet

**Flags** (go after the last slash)

| Flag | What it does |
|------|-------------------------------------------------|
| `g` | replace **all** matches on the line |
| `i` | ignore upper/lower case |
| `p` | print the changed line (use together with `-n`) |

**Options**

| Option | What it does |
|----------|--------------------------------------------|
| `-n` | don't print automatically |
| `-E` | use extended regex - **I always use this** |
| `-i` | edit the file for real (dangerous) |
| `-i.bak` | edit for real but keep a backup (safe) |
| `-e` | add another command |

**Commands**

| Command | What it does |
|---------|-------------------------|
| `s` | substitute |
| `d` | delete the line |
| `p` | print the line |
| `q` | quit |

---

## Gotcha 1: without `g` only the first match changes

```bash
echo "a-a-a" | sed 's/a/X/'    # X-a-a   <- only the first one
echo "a-a-a" | sed 's/a/X/g'   # X-X-X   <- what I usually want
```

I forget this constantly. If a replacement "didn't work", `g` is the first thing
I check.

---

## Gotcha 2: `-i` is permanent

```bash
sed -i 's/8080/9090/' app.conf
```

No undo, no confirmation. If my pattern is wrong, the file is gone.

Two habits that saved me:

```bash
# 1. always run it WITHOUT -i first and look at the output
sed 's/8080/9090/' app.conf

# 2. when I do use -i, keep a backup
sed -i.bak 's/8080/9090/' app.conf
```

Note: `-i` is a GNU extension. On macOS the syntax is slightly different.

---

## Gotcha 3: use `-E`, life gets easier

Without `-E`, symbols like `+` and `?` need a backslash:

```bash
sed 's/[0-9]\+/NUM/g' app.conf     # ugly
sed -E 's/[0-9]+/NUM/g' app.conf   # clean
```

I just add `-E` by default now.

---

## Choosing which lines to touch

By default the command runs on **every line**. An address before the command
narrows it down.

```bash
sed '2s/8080/9090/' app.conf         # line 2 only
sed '2,4s/=/:/' app.conf             # lines 2 to 4
sed '$s/30/60/' app.conf             # last line ($ = last line here)
sed '/debug/s/true/false/' app.conf  # only lines containing "debug"
sed '/ERROR/!d' app.log              # ! inverts: keep ONLY error lines
```

---

## Deleting and printing lines

```bash
sed '/^#/d' app.conf        # drop comment lines
sed '/^$/d' app.conf        # drop empty lines
sed '3d' app.conf           # drop line 3
sed '1d' app.conf           # drop the header line

sed -n '3p' app.log         # print line 3
sed -n '2,4p' app.log       # print lines 2 to 4
sed -n '/ERROR/p' app.log   # behaves like grep
```

`-n` matters here. Without it, matching lines get printed **twice**, because `sed`
already prints every line by default:

```bash
sed '/ERROR/p' app.log      # each error line appears twice
sed -n '/ERROR/p' app.log   # correct
```

---

## Changing the delimiter (very useful for paths)

The `/` isn't required. Whatever character comes right after `s` becomes the
delimiter.

```bash
sed 's/\/usr\/local\/bin/\/opt\/bin/' file   # unreadable
sed 's|/usr/local/bin|/opt/bin|' file        # much better
```

I use `|` whenever paths are involved.

---

## Capture groups

Put part of the pattern in `( )`, then reuse it as `\1`, `\2`, ...

```bash
echo "web-01" | sed -E 's/([a-z]+)-([0-9]+)/\2-\1/'
# 01-web
```

And `&` means "the whole match":

```bash
echo "port 8080" | sed -E 's/[0-9]+/[&]/'
# port [8080]
```

---

## Variables inside sed

Single quotes do **not** expand variables:

```bash
new_port="9090"

sed 's/port = .*/port = $new_port/' app.conf   # WRONG - writes $new_port literally
sed "s/port = .*/port = $new_port/" app.conf   # RIGHT
```

If the variable holds a path, switch the delimiter too:

```bash
new_path="/opt/app/bin"
sed "s|^path = .*|path = $new_path|" app.conf
```

---

## Snippets I actually use

```bash
# update an image tag in a kubernetes manifest
sed -i.bak 's|image: myapp:.*|image: myapp:v2.0|' deployment.yaml

# comment out a config line
sed -i '/^debug/s/^/#/' app.conf

# uncomment it again
sed -i '/^#debug/s/^#//' app.conf

# clean a config file: no comments, no empty lines
sed -e '/^#/d' -e '/^[[:space:]]*$/d' app.conf

# drop the header row from a command output
kubectl get pods | sed '1d'

# strip terminal colour codes from logs
docker logs mycontainer | sed -E 's/\x1b\[[0-9;]*m//g'

# trim leading and trailing spaces
sed -E 's/^[[:space:]]+|[[:space:]]+$//g' file.txt
```

---

## When to use what

| Job | Tool |
|--------------------------------------|------|
| search / filter lines | grep |
| replace text, delete lines | sed |
| work with columns, math, conditions | awk |

Short version: **text inside a line -> sed. Columns and calculations -> awk.**

---

## Official docs

- [GNU sed Manual](https://www.gnu.org/software/sed/manual/sed.html)
- [GNU sed Manual - by chapter](https://www.gnu.org/software/sed/manual/html_node/)
- [man7 - sed(1)](https://man7.org/linux/man-pages/man1/sed.1.html)
- [POSIX - sed](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/sed.html)

---

## TL;DR for future me

```
sed 's/old/new/g' file      the command I use 99% of the time
-g flag missing  -> only the first match per line changed
-n missing       -> matching lines printed twice with p
-E              -> always use it
-i              -> permanent. test without it first, or use -i.bak
s|old|new|      -> change the delimiter when paths are involved
\1 \2           -> capture groups
&               -> the whole match
sed reads ONE LINE at a time - it can't easily match across lines
```
