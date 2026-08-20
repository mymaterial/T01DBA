# Chapter 12: Working with Files in Scripts

## 1. Testing files before acting on them

```bash
#!/bin/bash
file="/etc/myapp.conf"

if [ -e "$file" ]; then
    echo "Exists"
fi
if [ -f "$file" ]; then
    echo "Is a regular file"
fi
if [ -d "/etc" ]; then
    echo "Is a directory"
fi
if [ -r "$file" ] && [ -w "$file" ]; then
    echo "Readable and writable"
fi
if [ -s "$file" ]; then
    echo "Not empty"
fi
```

## 2. Creating, copying, moving, deleting safely

```bash
#!/bin/bash
mkdir -p /backup/2026-08-16      # -p: no error if it already exists, creates parents too
touch /backup/2026-08-16/.keep

cp -v source.txt /backup/2026-08-16/        # -v: verbose, show what's happening
cp -r /data /backup/2026-08-16/data          # -r: recursive, for directories

mv old_name.txt new_name.txt

rm -i old_report.txt        # -i: ask for confirmation before deleting
rm -rf /tmp/scratch_work/   # -rf: recursive + force — use with real caution
```

## 3. Reading a file line by line

```bash
#!/bin/bash
while IFS= read -r line; do
    echo "Processing: $line"
done < "servers.txt"
```

## 4. Writing/generating files from a script

```bash
#!/bin/bash
report_file="/tmp/report_$(date +%F).txt"

{
    echo "Report generated: $(date)"
    echo "Disk usage:"
    df -h
    echo "Memory usage:"
    free -h
} > "$report_file"

echo "Report saved to $report_file"
```

## 5. Finding files and acting on each

```bash
#!/bin/bash
# Find and process each matching file
find /var/log -name "*.log" -mtime +30 | while read -r old_log; do
    echo "Archiving $old_log"
    gzip "$old_log"
done

# Find and delete in one step (careful!)
find /tmp -name "*.tmp" -mtime +7 -delete
```

## 6. Checking free space before writing a large file

```bash
#!/bin/bash
required_mb=500
available_mb=$(df --output=avail -m /data | tail -1 | tr -d ' ')

if [ "$available_mb" -lt "$required_mb" ]; then
    echo "Not enough disk space (need ${required_mb}MB, have ${available_mb}MB)" >&2
    exit 1
fi
echo "Enough space available, proceeding..."
```

## 7. Locking a file to prevent concurrent script runs

A very common real-world need: prevent the same script from running twice at once (e.g. two
overlapping cron executions of a backup job).

```bash
#!/bin/bash
LOCKFILE="/tmp/mybackup.lock"

if [ -e "$LOCKFILE" ]; then
    echo "Script already running (lockfile exists). Exiting."
    exit 1
fi

trap 'rm -f "$LOCKFILE"' EXIT
touch "$LOCKFILE"

echo "Doing work..."
sleep 5
echo "Done."
```
More robust version using `flock` (handles crashes/stale locks better):
```bash
#!/bin/bash
exec 200>/tmp/mybackup.lock
flock -n 200 || { echo "Already running"; exit 1; }

echo "Doing work..."
sleep 5
echo "Done."
```

## 8. Comparing files

```bash
#!/bin/bash
if diff -q file1.txt file2.txt > /dev/null; then
    echo "Files are identical"
else
    echo "Files differ"
fi

if [ "file1.txt" -nt "file2.txt" ]; then
    echo "file1.txt is newer"
fi
```

### Key takeaways

- Always test before acting: `-f`, `-d`, `-r`, `-w`, `-s` prevent scripts from failing on
  missing or unreadable files.
- `while IFS= read -r line; do ... done < file` is the correct, safe way to process a file
  line by line.
- Check available disk space *before* writing large files, not after.
- Use a lockfile (or `flock`) in any script that must never run twice concurrently — this
  matters enormously for backup and maintenance jobs.

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. Before ever touching `postgresql.conf`, write a small safety-check script:
   ```bash
   CONF="/u01/pgsql/18/data/postgresql.conf"
   [ -f "$CONF" ] || { echo "Missing: $CONF"; exit 1; }
   [ -r "$CONF" ] || { echo "Not readable: $CONF"; exit 1; }
   [ -w "$CONF" ] || echo "Note: not writable by current user"
   echo "Safe to proceed"
   ```

2. Practice the "always copy before you edit" habit as an actual script:
   ```bash
   cp -v /u01/pgsql/18/data/postgresql.conf \
        /u01/pgsql/18/data/postgresql.conf.bak.$(date +%F)
   ```

3. Write a `while read` loop that reads a list of setting names from a text file (one per
   line) and greps each from `postgresql.conf`.

4. Implement a lockfile around a (simulated) config-edit script, so it refuses to run if
   another copy is already in progress — reuse the `flock` pattern from this chapter.

**Next:** [Chapter 13 — Scheduling Scripts](13-scheduling-scripts.md)
**Previous:** [Chapter 11 — Error Handling & Debugging](11-error-handling-and-debugging.md)
