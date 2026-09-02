# 01 - Streams: stdin, stdout, stderr

> My study notes while learning Bash for DevOps. I wrote these the way I understood
> them, including the parts that confused me at first. If something here looks
> obvious, it wasn't obvious to me on day one.

---

## The mental model

Every command I run on Linux gets **three channels** opened for it automatically.
I don't create them, I don't open them. They are just there.

| Number | Name | What it does |
|--------|--------|--------------------------------|
| `0` | stdin | where the command reads input from |
| `1` | stdout | the normal / correct output |
| `2` | stderr | error and warning messages |

The number is called a **file descriptor**.

The thing that confused me: **stdout and stderr both print to my screen**, so they
look like one thing. They are not. They are two completely separate channels and I
can send them to two different places.

---

## Proof that they are separate

```bash
ls /tmp /does-not-exist
```

On screen I get a mix: the file listing AND an error line. Now separate them:

```bash
ls /tmp /does-not-exist > out.txt 2> err.txt

cat out.txt   # only the file listing
cat err.txt   # only the error message
```

Two different files. So two different channels. This is the moment it clicked for me.

---

## Cheat sheet

```bash
cmd > file        # stdout -> file (overwrite)
cmd >> file       # stdout -> file (append)
cmd 2> file       # stderr -> file
cmd 2>> file      # stderr -> file (append)
cmd > file 2>&1   # stdout + stderr -> same file
cmd &> file       # bash shortcut for the line above
cmd 2>/dev/null   # throw errors away
cmd >/dev/null    # throw normal output away
cmd < file        # file -> stdin
cmd1 | cmd2       # stdout of cmd1 -> stdin of cmd2
echo "msg" >&2    # write to stderr from inside a script
```

`/dev/null` is a fake file. Anything I send there disappears. It's the trash can.

---

## Gotcha 1: a pipe only carries stdout

This one cost me time.

```bash
ls /tmp /does-not-exist | wc -l
```

I expected the error to go into the pipe. It doesn't. The error still shows up on
my screen, because **a pipe only carries channel 1**.

If I want both in the pipe:

```bash
ls /tmp /does-not-exist 2>&1 | wc -l
```

---

## Gotcha 2: the order of redirections matters

```bash
cmd > file 2>&1   # correct: both end up in the file
cmd 2>&1 > file   # wrong: errors still go to the screen
```

Why? The line is read left to right.

- First version: send channel 1 to the file, then tell channel 2 "go wherever
  channel 1 goes" (which is now the file). Both land in the file.
- Second version: tell channel 2 "go wherever channel 1 goes" (still the screen at
  that point), then move channel 1 to the file. Errors stayed on the screen.

I just memorised the correct one: **redirect stdout first, then duplicate.**

---

## Gotcha 3: errors in my own scripts

This is the practical payoff of the whole topic.

**Bad** - everything goes to stdout, so errors pollute my results:

```bash
if ! systemctl is-active --quiet nginx; then
    echo "ERROR: nginx is not running"
    exit 1
fi
```

**Good** - errors go to channel 2 where they belong:

```bash
if ! systemctl is-active --quiet nginx; then
    echo "ERROR: nginx is not running" >&2
    exit 1
fi
```

Now my script plays nicely with other tools:

```bash
./check.sh > report.txt     # clean results only
./check.sh 2> errors.log    # problems only
./check.sh | grep running   # errors don't pollute the pipe
```

**Rule I follow now:** results go to stdout, messages for humans go to stderr.

---

## Snippets I actually use

```bash
# Docker: get container names, ignore any warnings
docker ps --format '{{.Names}}' > containers.txt 2>/dev/null

# Kubernetes: keep results and errors apart
kubectl get pods > pods.txt 2> kube_errors.txt

# Disk report to a file
df -h > disk_report.txt 2> disk_errors.txt

# Silence everything, I only care whether it worked
if ! git fetch origin > /dev/null 2>&1; then
    echo "ERROR: git fetch failed" >&2
    exit 1
fi
```

---

## Official docs

- [GNU Bash Manual - Redirections](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)
- [POSIX - stdin, stdout, stderr](https://pubs.opengroup.org/onlinepubs/9699919799/functions/stdin.html)
- [man7 - stdin(3)](https://man7.org/linux/man-pages/man3/stdin.3.html)
- [man7 - stdout(3)](https://man7.org/linux/man-pages/man3/stdout.3.html)

---

## TL;DR for future me

```
0 = stdin, 1 = stdout, 2 = stderr
stdout and stderr look the same on screen but are NOT the same
a pipe carries stdout only
in my scripts: results -> stdout, errors -> stderr  (echo "..." >&2)
cmd > file 2>&1   is the correct order
```
