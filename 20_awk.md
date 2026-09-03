# awk

## Quick Reference Notes - Daily Use

No long explanations, just ready-to-apply patterns.

---

## The 5 Core Patterns

### 1. Extract a specific column

Image column from

docker ps

```bash
docker ps | awk '{print $2}'
```

### 2. Filter by condition, then print a column

Pods whose status is not

Running

```bash
kubectl get pods | awk '$3 != "Running" {print $1}'
```

### 3. Skip the header line

Using

NR

```bash
df -h | awk 'NR>1 {print $1, $5}'
```

### 4. Custom input delimiter

Using

-F

```bash
docker images --format "{{.Repository}}:{{.Tag}}" | awk -F':' '{print $1}'
```

### 5. Count lines matching a pattern

Using

END

```bash
systemctl list-units --type=service | awk '/failed/ {count++} END {print count}'
```

---

## Additional Daily Patterns

### 6. Sum a numeric column

Example: total memory column from

ps aux

```bash
ps aux | awk 'NR>1 {sum += $6} END {print sum}'
```

### 7. Custom output separator

Using

OFS

```bash
docker ps | awk 'BEGIN{OFS="|"} {print $1, $2}'
```

### 8. Combine a filter tool before extraction

Filter first with another tool, then extract a column

```bash
kubectl get pods | grep Error | awk '{print $1}'
```

---

## Simple Script: Disk Usage Warning

Combines pattern 3 inside an actual script, with one simple condition added.

```bash
#!/bin/bash

echo "Checking disk usage..."

df -h | awk '$5+0 > 80 {print "WARNING:", $1, "is at", $5}'

echo "Done."
```
