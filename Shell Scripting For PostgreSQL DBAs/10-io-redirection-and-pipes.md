# Chapter 10: I/O Redirection & Pipes

Every process has three standard streams: **stdin** (0, input), **stdout** (1, normal
output), and **stderr** (2, error output). Redirection lets you control where each goes.

## 1. Output redirection

```bash
#!/bin/bash
echo "Backup started" > backup.log      # overwrite (create or replace) the file
echo "Backup finished" >> backup.log     # append to the file
```

```bash
$ cat backup.log
Backup started
Backup finished
```

## 2. Input redirection

```bash
#!/bin/bash
while IFS= read -r line; do
    echo "Read: $line"
done < input.txt
```

## 3. Error redirection

```bash
#!/bin/bash
ls /nonexistent 2> error.log        # send stderr only to a file
ls /nonexistent 2>> error.log        # append stderr

ls /nonexistent > output.log 2>&1    # send BOTH stdout and stderr to the same file
ls /nonexistent &> combined.log       # shorthand for the same thing

ls /nonexistent 2>/dev/null           # discard errors entirely (silence them)
```

**Order matters** in `> output.log 2>&1`: it means "point stdout at the file, then point
stderr at wherever stdout currently points." Writing `2>&1 > output.log` (reversed) does
*not* do what you want.

## 4. Pipes — connecting commands

```bash
#!/bin/bash
ps aux | grep postgres | wc -l         # count postgres processes
cat access.log | grep "ERROR" | sort | uniq -c | sort -nr | head
```
Each command's stdout becomes the next command's stdin.

## 5. Here-documents (`<<`)

A here-document feeds multi-line text as input to a command, without a separate file.

```bash
#!/bin/bash
cat <<EOF
This is line one.
This is line two.
Today is $(date +%F).
EOF
```

Commonly used to feed SQL to `psql` directly from a script:

```bash
#!/bin/bash
psql -d mydb <<SQL
SELECT count(*) FROM users;
SELECT count(*) FROM orders;
SQL
```

**Quoting the delimiter** (`<<'EOF'`) disables variable expansion inside the block — useful
when the content itself contains `$` characters you don't want interpreted:

```bash
#!/bin/bash
cat <<'EOF'
This $variable will NOT be expanded.
EOF
```

## 6. Here-strings (`<<<`)

Feeds a single string as input, without a full here-document block.

```bash
#!/bin/bash
grep "error" <<< "$log_line"
read -r name age <<< "Alice 30"
echo "$name is $age years old"
```

## 7. `tee` — write to a file and stdout simultaneously

```bash
#!/bin/bash
echo "Deploying version 2.1..." | tee deploy.log
df -h | tee -a disk_report.log      # -a appends instead of overwriting
```
`tee` is invaluable when you want to both watch a script's output live *and* keep a log of
it, e.g. `./backup.sh | tee -a /var/log/backup.log`.

## 8. Redirecting inside a function or block

```bash
#!/bin/bash
{
    echo "=== Report ==="
    date
    df -h
} > report.txt
```
Grouping commands with `{ ...; }` and redirecting once is cleaner than redirecting every
line individually.

### Key takeaways

- `>` overwrites, `>>` appends, `2>` captures errors, `&>`/`2>&1` capture everything.
- Pipes (`|`) chain commands — combine with `grep`, `sort`, `uniq -c`, `awk` for powerful
  one-liners.
- Here-documents (`<<EOF`) are the standard way to feed multi-line SQL or config text into a
  command from inside a script.
- `tee` lets you watch output live while also saving it to a log file.

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. Redirect the output of a settings check to a report file, appending each time:
   ```bash
   echo "=== Check at $(date) ===" >> /tmp/pg_settings_report.txt
   grep work_mem /u01/pgsql/18/data/postgresql.conf >> /tmp/pg_settings_report.txt
   ```

2. Capture both stdout and stderr when checking a path that might not exist:
   ```bash
   cat /u01/pgsql/99/data/postgresql.conf > /tmp/check.log 2>&1
   cat /tmp/check.log
   ```

3. Feed SQL to `psql` via a here-document to check three settings from inside PostgreSQL
   itself (not just the file):
   ```bash
   psql -tAc <<SQL
   SHOW work_mem;
   SHOW shared_buffers;
   SHOW max_connections;
   SQL
   ```

4. Use `tee` so a config check is both printed to screen and saved:
   ```bash
   grep -v '^#' /u01/pgsql/18/data/postgresql.conf | grep -v '^$' | tee /tmp/active_settings.txt
   ```

**Next:** [Chapter 11 — Error Handling & Debugging](11-error-handling-and-debugging.md)
**Previous:** [Chapter 9 — String Manipulation](09-string-manipulation.md)
