# Chapter 7: Searching and Text Processing Commands

## 1. Searching Commands

These commands help search files, directories, and text inside files.

### `find`

| Command | Purpose | Example |
|---|---|---|
| `find . -name "file.txt"` | Search for a file by name | `find . -name "notes.txt"` |
| `find /home -type d` | Search for directories | `find /home -type d` |
| `find . -name "*.log"` | Search files with extension | `find . -name "*.log"` |
| `find . -size +10M` | Search files larger than 10MB | `find . -size +10M` |
| `find . -mtime -7` | Search files modified in last 7 days | `find . -mtime -7` |

### `locate` (uses the `updatedb` database — fast but can be stale)

| Command | Purpose | Example |
|---|---|---|
| `locate filename` | Quick search for a file in the whole system | `locate report.pdf` |

### `grep` (search text inside files)

| Command | Purpose | Example |
|---|---|---|
| `grep "text" file.txt` | Search for text in a file | `grep "error" log.txt` |
| `grep -i "text" file.txt` | Case-insensitive search | `grep -i "error" log.txt` |
| `grep -n "text" file.txt` | Show line number with match | `grep -n "error" log.txt` |
| `grep -r "text" /dir` | Search recursively in a directory | `grep -r "TODO" /home/user` |

### Examples

- Find all `.txt` files: `find . -name "*.txt"`
- Find files modified in last 3 days: `find . -mtime -3`
- Search text "password" in all files under `/etc`: `grep -r "password" /etc`

## 2. Text Processing Commands

These commands view, manipulate, and process text data in files.

| Command | Purpose | Example |
|---|---|---|
| `cat` | Concatenate and display file content | `cat file.txt` |
| `less` | View file content page by page | `less file.txt` |
| `head` | Display first N lines (default 10) | `head -n 20 file.txt` |
| `tail` | Display last N lines; `-f` follows live output | `tail -n 20 file.txt`, `tail -f file.txt` |
| `cut` | Extract sections (columns) from each line | `cut -d "," -f 1,3 file.csv` |
| `sort` | Sort lines of text | `sort file.txt`, `sort -r file.txt` (reverse) |
| `uniq` | Remove duplicate lines (use after `sort`) | `sort file.txt \| uniq` |
| `wc` | Count lines, words, characters | `wc file.txt` |
| `tr` | Translate or delete characters | `tr 'a-z' 'A-Z' < file.txt` |

### `less` navigation

| Key | Action |
|---|---|
| Space | Next page |
| `b` | Previous page |
| `/text` | Search forward |
| `q` | Quit |

### `head`/`tail` options

| Option | Meaning |
|---|---|
| `-n N` | Show first/last N lines |
| `-f` | Follow (live output) |

### `wc` options

| Option | Meaning |
|---|---|
| `-l` | Lines |
| `-w` | Words |
| `-c` | Characters/bytes |

### Quick pipeline example

```bash
cat file.txt | grep "linux" | sort | uniq -c | sort -nr
```
Search "linux" in file, sort results, remove duplicates, count occurrences, and sort by
count (descending).

---

## Where a PostgreSQL DBA uses this

This is the chapter you'll use the most during incident response. PostgreSQL log files are
plain text, and log mining is a core DBA skill.

### Finding the right log file

```bash
find /var/log/postgresql -name "*.log" -mtime -1     # logs touched in the last day
find $PGDATA -name "*.conf"                            # locate config files inside PGDATA
find / -iname "postgresql.conf" 2>/dev/null            # on an unfamiliar host
```

### Mining logs with `grep`

```bash
grep -i "error" /var/log/postgresql/postgresql-15-main.log
grep -i "fatal" /var/log/postgresql/postgresql-15-main.log
grep -n "deadlock detected" postgresql-15-main.log
grep -r "connection refused" /var/log/postgresql/
```

### Finding slow queries (with `log_min_duration_statement` enabled)

```bash
grep "duration:" postgresql-15-main.log | sort -t: -k4 -nr | head -20
```

### Counting error types quickly

```bash
grep "ERROR" postgresql-15-main.log | cut -d' ' -f6- | sort | uniq -c | sort -nr | head
```
This pipeline is the exact pattern from the "quick pipeline example" above, applied to a
real PostgreSQL log: extract error messages, count how often each distinct message appears,
and show the most frequent ones first — extremely useful for spotting a single misbehaving
application flooding the log.

### Watching a live incident

```bash
tail -f /var/log/postgresql/postgresql-15-main.log | grep --line-buffered -i "error\|fatal\|panic"
```

### Working with `pg_stat_activity` exports or CSV logs

```bash
cut -d',' -f1,4,7 postgres_csvlog.csv | head -20   # pull specific columns from a CSV log
wc -l postgres_csvlog.csv                            # how many log lines/events total
```

### Checking for a specific query pattern across many files

```bash
grep -rl "DROP TABLE" /var/log/postgresql/           # list files containing dangerous DDL
```

**`grep -r` across log directories, `tail -f` for live monitoring, and `sort | uniq -c` for
frequency analysis** together form the backbone of manual log-based troubleshooting before
you reach for a dedicated log analysis tool like `pgBadger`.

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

This chapter is the payoff for the "find one keyword in a huge file" homework given at the
end of Day 1 of the bootcamp. `postgresql.conf` typically has 300+ lines, mostly comments —
exactly the kind of file `grep` exists for.

### The actual bootcamp homework exercise

1. Connect to your lab instance and navigate to the PostgreSQL config location:
   ```bash
   cd /u01/pgsql/18/data
   ls
   cat postgresql.conf
   ```
   Scrolling through the whole file to find one setting is painful — that's the point.

2. Now find it the fast way:
   ```bash
   grep work_mem postgresql.conf
   ```
   Expected output includes a line like:
   ```
   #work_mem = 4MB                    # min 64kB
   ```
   **Answer the homework question: what is the value of `work_mem`?** → `4MB` (the default).
   Note it's commented out with `#` — meaning PostgreSQL is using the built-in default, not
   an explicit override.

3. Repeat for a few more real settings, to build the habit:
   ```bash
   grep shared_buffers postgresql.conf
   grep listen_addresses postgresql.conf
   grep max_connections postgresql.conf
   grep wal_level postgresql.conf
   ```

4. **Case-insensitive and line-numbered search** — useful when you're not sure of exact
   capitalization or want to jump straight to a line in an editor:
   ```bash
   grep -in "max_connections" postgresql.conf
   ```

5. **Find only the active (non-commented, non-blank) settings** — the single most useful
   `grep` pattern for any PostgreSQL config file:
   ```bash
   grep -v '^#' postgresql.conf | grep -v '^$'
   ```
   This shows you *only* what's actually been explicitly configured, filtering out the wall
   of default comments.

6. **Simulate the exercise without a real install**, if you don't have PostgreSQL yet:
   ```bash
   mkdir -p ~/pg_practice/18/data && cd ~/pg_practice/18/data
   cat > postgresql.conf <<'EOF'
   # This is a comment
   #work_mem = 4MB
   shared_buffers = 128MB
   #listen_addresses = 'localhost'
   max_connections = 100
   EOF
   grep work_mem postgresql.conf
   grep -v '^#' postgresql.conf
   ```

7. **Search across every `.conf` file at once** (a step beyond the homework, but the natural
   next question): find which config file defines a setting when you're not sure:
   ```bash
   grep -r "listen_addresses" /u01/pgsql/18/data/*.conf
   ```

**Next:** [Chapter 8 — Process Management](08-linux-process-management.md)
**Previous:** [Chapter 6 — File Permissions](06-linux-file-permissions.md)
