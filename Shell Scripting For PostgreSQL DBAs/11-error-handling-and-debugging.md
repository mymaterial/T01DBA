# Chapter 11: Error Handling & Debugging

Scripts that run unattended (cron, systemd timers) need to fail loudly and safely, not
silently corrupt data or hang forever. This chapter covers the tools that make that possible.

## 1. Exit codes

Every command returns an exit status: `0` = success, non-zero = failure. `$?` holds the exit
status of the **last** command run.

```bash
#!/bin/bash
ls /tmp
echo "Exit code of ls: $?"    # 0

ls /nonexistent
echo "Exit code of ls: $?"     # 2 (or another non-zero code)
```

A script can set its own exit code with `exit N`:

```bash
#!/bin/bash
if [ ! -f "/etc/myapp.conf" ]; then
    echo "Config missing!" >&2
    exit 1
fi
echo "Config found."
exit 0
```

## 2. Checking success/failure explicitly

```bash
#!/bin/bash
cp important.db /backup/

if [ $? -eq 0 ]; then
    echo "Copy succeeded"
else
    echo "Copy FAILED" >&2
    exit 1
fi
```

Shorter, idiomatic version using `&&`/`||`:
```bash
#!/bin/bash
cp important.db /backup/ && echo "Copy succeeded" || { echo "Copy FAILED" >&2; exit 1; }
```

## 3. `set` options — the safety net

```bash
#!/bin/bash
set -e            # exit immediately if any command fails
set -u            # error on use of an undefined variable
set -o pipefail   # a pipeline fails if ANY command in it fails, not just the last one

# Commonly combined:
set -euo pipefail
```

| Option | Effect |
|---|---|
| `set -e` | Exit immediately on any command failure |
| `set -u` | Treat unset variables as an error |
| `set -o pipefail` | A pipe's exit status reflects the first failing command, not just the last |
| `set -x` | Print each command before executing it (debug tracing) |

**Why `pipefail` matters:**
```bash
#!/bin/bash
set -o pipefail
false | echo "this still ran"
echo "exit status: $?"     # without pipefail this would be 0 (echo succeeded);
                             # with pipefail it correctly reports failure
```

**Caveat with `set -e`:** it does *not* trigger inside `if cmd; then` conditions, or for
commands whose failure you're explicitly checking with `||` — this is intentional, but worth
knowing so `set -e` doesn't surprise you by NOT catching something you expected it to.

## 4. `trap` — cleanup and signal handling

`trap` runs a command when the script receives a signal or exits, which is the standard way
to guarantee cleanup (temp files, lock files, DB connections) even if the script fails
partway through.

```bash
#!/bin/bash
tmpfile=$(mktemp)

cleanup() {
    echo "Cleaning up..."
    rm -f "$tmpfile"
}
trap cleanup EXIT     # runs on ANY exit: normal, error, or signal

echo "Working with $tmpfile"
echo "some data" > "$tmpfile"
# ... script continues; cleanup() runs automatically when it exits, no matter how
```

Trapping specific signals:
```bash
#!/bin/bash
trap 'echo "Interrupted! Exiting cleanly."; exit 1' SIGINT SIGTERM

echo "Running... press Ctrl+C to test"
sleep 30
```

## 5. Debugging with `bash -x` / `set -x`

```bash
#!/bin/bash
set -x           # turn tracing on
a=5
b=3
echo $((a+b))
set +x            # turn tracing off
echo "done"
```
Output shows each command as it's executed, prefixed with `+`:
```
+ a=5
+ b=3
+ echo 8
8
+ set +x
done
```

Or trace an entire script without editing it:
```bash
bash -x myscript.sh
```

## 6. Custom error-handling function (a reusable pattern)

```bash
#!/bin/bash
set -euo pipefail

die() {
    echo "ERROR: $1" >&2
    exit "${2:-1}"
}

[ -f "/etc/myapp.conf" ] || die "Config file not found"
command -v psql &> /dev/null || die "psql is not installed" 2

echo "All checks passed"
```

## 7. Putting it together: a defensively written script

```bash
#!/bin/bash
set -euo pipefail

LOG_FILE="/var/log/myscript.log"

log()  { echo "[$(date '+%F %T')] $1" | tee -a "$LOG_FILE"; }
die()  { log "ERROR: $1"; exit "${2:-1}"; }

trap 'die "Script interrupted unexpectedly (line $LINENO)"' ERR

log "Starting job"

[ -d /data ] || die "/data directory missing"

log "Job completed successfully"
```
`trap ... ERR` combined with `set -e` gives you a single place that catches *any* unexpected
command failure and logs it with the line number, instead of the script failing silently or
half-completing.

### Key takeaways

- `set -euo pipefail` at the top of nearly every production script is the standard baseline.
- `trap cleanup EXIT` guarantees temp files/locks get cleaned up no matter how the script
  exits.
- `bash -x script.sh` or `set -x`/`set +x` around a section is the fastest way to see exactly
  what a script is doing.
- A small `log()`/`die()` function pair, used consistently, turns ad-hoc scripts into
  something safe to run unattended.

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. Add `set -euo pipefail` to your `pg_conf_lookup.sh` from Chapter 3, then intentionally
   pass a nonexistent setting name and observe what happens to the exit code.

2. Write a `die()` function and use it to fail loudly if `/u01/pgsql/18/data` doesn't exist:
   ```bash
   die() { echo "ERROR: $1" >&2; exit 1; }
   [ -d /u01/pgsql/18/data ] || die "PGDATA not found — check your PostgreSQL install"
   ```

3. Add a `trap` that logs a message whenever your script exits, whether successfully or not:
   ```bash
   trap 'echo "Script finished at $(date)"' EXIT
   ```

4. Debug a broken version of your `pg_conf_lookup.sh` by running `bash -x pg_conf_lookup.sh
   work_mem` and reading the trace output line by line.

**Next:** [Chapter 12 — Working with Files](12-working-with-files.md)
**Previous:** [Chapter 10 — I/O Redirection & Pipes](10-io-redirection-and-pipes.md)
