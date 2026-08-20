# Chapter 8: Process Management

A process is a program in execution. Linux provides many commands to view, control, and
manage running processes.

## 1. Viewing Processes

| Command | Purpose | Example |
|---|---|---|
| `ps` | Show current user processes | `ps` |
| `ps aux` | Show all running processes | `ps aux` |
| `ps -ef` | Full format listing | `ps -ef` |
| `top` | Interactive real-time process viewer | `top` |
| `htop` | Enhanced process viewer (if installed) | `htop` |

## 2. Process Information (`ps` columns)

| Column | Meaning |
|---|---|
| PID | Process ID |
| PPID | Parent Process ID |
| USER | Owner of the process |
| %CPU | CPU usage |
| %MEM | Memory usage |
| STAT | Process state |
| COMMAND | Command name |

### Process State (STAT)

| Code | Meaning |
|---|---|
| R | Running |
| S | Sleeping (waiting) |
| D | Uninterruptible sleep |
| T | Stopped |
| Z | Zombie (defunct) |
| `<` | High priority |
| N | Low priority |

## 3. Controlling Processes

| Command | Purpose | Example |
|---|---|---|
| `kill PID` | Terminate a process (gracefully) | `kill 1234` |
| `kill -9 PID` | Force kill a process (SIGKILL) | `kill -9 1234` |
| `pkill name` | Kill process by name | `pkill firefox` |
| `killall name` | Kill all processes by name | `killall chrome` |

## 4. Job Control (foreground & background processes)

| Command | Purpose | Example |
|---|---|---|
| `&` | Run command in background | `sleep 100 &` |
| `jobs` | List background jobs | `jobs` |
| `fg %n` | Bring job n to foreground | `fg %1` |
| `bg %n` | Resume job n in background | `bg %1` |
| `Ctrl+Z` | Stop current foreground job | — |

## 5. Changing Process Priority (Nice & Renice)

| Command | Purpose | Example |
|---|---|---|
| `nice -n 10 cmd` | Start a process with a nice value | `nice -n 10 python app.py` |
| `renice -n 5 -p PID` | Change priority of a running process | `renice -n 5 -p 1234` |
| `top` / `htop` | Adjust priority interactively | `top` → press `r` |

## 6. Process Hierarchy

```
init/systemd (PID 1)
├── bash (PID 100)
│   ├── vim (PID 101)
│   └── gcc (PID 102)
├── sshd (PID 200)
└── cron (PID 300)
```

`init`/`systemd` is always the parent of all processes.

## 7. Monitoring Resources

| Command | Purpose | Example |
|---|---|---|
| `top` | Monitor CPU & memory | `top` |
| `htop` | Better interactive monitor | `htop` |
| `free -h` | Show memory usage | `free -h` |
| `df -h` | Show disk usage | `df -h` |
| `uptime` | System load & uptime | `uptime` |

## 8. Useful Examples

```bash
ps aux --forest              # view all processes in tree format
ps -u username                # show processes of a specific user
ps -p 1234                    # find process by PID
ps -p 1234 -o pid,ppid,%mem   # check memory usage of a process
pkill -f python                # kill all "python" processes
```

### Tips

- Use `top` or `htop` to monitor the system in real time.
- Be careful with `kill -9` (force kill) — it skips cleanup.
- Zombie processes (`Z`) are terminated but their entry remains in the process table.
- `init`/`systemd` is the parent of all processes.

---

## Where a PostgreSQL DBA uses this

This is the chapter that matters most during a "the database is stuck" call.

### PostgreSQL's process model

Every PostgreSQL instance is a **process tree**: one `postmaster` (the main server process)
that forks a backend process per client connection, plus a set of background workers.

```bash
ps -ef --forest | grep postgres
```
Typical output shape:
```
postgres   1000      1  0  postgres        <- postmaster (the main server)
postgres   1001   1000  0   \_ postgres: checkpointer
postgres   1002   1000  0   \_ postgres: background writer
postgres   1003   1000  0   \_ postgres: walwriter
postgres   1004   1000  0   \_ postgres: logical replication launcher
postgres   1050   1000  0   \_ postgres: appuser appdb 10.0.0.5(54210) idle
postgres   1051   1000  0   \_ postgres: appuser appdb 10.0.0.5(54211) SELECT
```
This directly explains why `max_connections` costs memory: **each connection is a full OS
process**, not a lightweight thread.

### Finding the problem connection

```bash
ps aux | grep postgres | sort -k3 -nr | head       # highest CPU-consuming postgres processes
ps -p <pid> -o pid,ppid,%cpu,%mem,etime,cmd          # inspect one specific backend
```
Cross-reference the PID with `pg_stat_activity.pid` inside PostgreSQL itself to see exactly
which query that OS process is running.

### Killing a runaway query — the right way

Prefer PostgreSQL's own signals over blunt `kill -9`:

```sql
SELECT pg_cancel_backend(1051);   -- polite: ask the query to cancel (like Ctrl+C)
SELECT pg_terminate_backend(1051); -- forceful: end the whole backend connection
```

At the OS level these map to:
```bash
kill -SIGINT 1051    # equivalent effect to pg_cancel_backend
kill -SIGTERM 1051   # equivalent effect to pg_terminate_backend
kill -9 1051          # AVOID on a postgres backend — see warning below
```

**Never `kill -9` an individual `postgres` backend process directly.** SIGKILL bypasses
PostgreSQL's own cleanup, and because backends share memory (`shared_buffers`) with the
postmaster, a `SIGKILL` on one backend is treated as a potential corruption event —
PostgreSQL will force the *entire* instance to crash-restart and go through recovery to
guarantee data integrity. If you must forcefully stop the whole instance, use
`pg_ctl stop -m immediate` instead, which is safe and coordinated.

### Watching load in real time

```bash
top -u postgres        # filter to just the postgres user's processes
htop                     # interactive view; F4 to filter "postgres"
```

### Zombie/orphan checks after a crash

```bash
ps aux | grep 'Z ' | grep postgres    # zombie postgres processes after an unclean shutdown
```

### Niceness for maintenance jobs

Long-running maintenance (e.g. a manual `VACUUM FULL` wrapper script, or a bulk restore) is
often run with a lower CPU priority so it doesn't starve live query traffic:
```bash
nice -n 10 pg_restore -d mydb backup.dump
```

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. **Find the postmaster.** On a running instance:
   ```bash
   ps -ef | grep postgres
   ```
   Identify the single process with no parent `postgres` PID above it — that's the
   postmaster (the main server process). Note its PID.

2. **See the whole process family tree.**
   ```bash
   ps -ef --forest | grep -A 20 postgres
   ```
   Identify: `checkpointer`, `background writer`, `walwriter`, and one or more backend
   processes tied to actual client connections.

3. **Cross-reference with PostgreSQL's own view.** Connect with `psql` and run:
   ```sql
   SELECT pid, usename, state, query FROM pg_stat_activity;
   ```
   Pick one `pid` from the SQL output and confirm it matches a row in `ps -ef | grep postgres`
   — this proves to yourself that "one connection = one OS process" isn't just a claim in a
   textbook.

4. **Practice the safe way to stop a query** (never `kill -9` a backend):
   ```sql
   SELECT pg_sleep(120);   -- run this in one psql session to create a long query
   ```
   In another terminal:
   ```bash
   psql -tAc "SELECT pid FROM pg_stat_activity WHERE query LIKE 'SELECT pg_sleep%';"
   ```
   Then, from `psql`:
   ```sql
   SELECT pg_cancel_backend(<pid>);
   ```
   Confirm the sleeping session ends immediately with a cancellation error.

5. **Check resource usage of just the postgres user's processes.**
   ```bash
   top -u postgres
   ```
   Watch CPU/memory usage live while running a query in another session.

**Next:** [Chapter 9 — Disk & Memory Commands](09-linux-disk-memory-commands.md)
**Previous:** [Chapter 7 — Searching & Text Processing](07-linux-searching-text-processing.md)
