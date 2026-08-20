# Chapter 16: PostgreSQL Health Check & Monitoring Scripts

These scripts wrap `psql -tAc` (tuples-only, unaligned, command) SQL queries in shell logic —
turning raw query output into pass/fail checks suitable for cron, systemd timers, or a
monitoring system's "run this script and check the exit code" plugin interface.

## 1. Basic connectivity check

```bash
#!/bin/bash
#
# pg_check_connectivity.sh — exits 0 if PostgreSQL is accepting connections
#
set -euo pipefail

HOST="${1:-localhost}"
PORT="${2:-5432}"

if pg_isready -h "$HOST" -p "$PORT" -q; then       # Ch.4/5 — command exit status
    echo "OK: PostgreSQL is accepting connections on $HOST:$PORT"
    exit 0
else
    echo "CRITICAL: PostgreSQL is NOT accepting connections on $HOST:$PORT" >&2
    exit 2
fi
```

## 2. Connection count / `max_connections` usage

```bash
#!/bin/bash
set -euo pipefail

WARN_PCT=80
CRIT_PCT=95

current=$(psql -tAc "SELECT count(*) FROM pg_stat_activity;")          # Ch.2 — command sub
max_conn=$(psql -tAc "SHOW max_connections;")
pct=$(( current * 100 / max_conn ))                                      # Ch.4 — arithmetic

echo "Connections: $current / $max_conn (${pct}%)"

if [ "$pct" -ge "$CRIT_PCT" ]; then                # Ch.5 — conditional, largest threshold first
    echo "CRITICAL: connection usage at ${pct}%" >&2
    exit 2
elif [ "$pct" -ge "$WARN_PCT" ]; then
    echo "WARNING: connection usage at ${pct}%" >&2
    exit 1
fi
exit 0
```

## 3. Replication lag check (run on a standby)

```bash
#!/bin/bash
#
# pg_check_replication_lag.sh — checks streaming replication lag in seconds
#
set -euo pipefail

WARN_SECONDS=30
CRIT_SECONDS=120

is_replica=$(psql -tAc "SELECT pg_is_in_recovery();" | tr -d '[:space:]')

if [ "$is_replica" != "t" ]; then
    echo "OK: this is the primary, not a replica — no lag to check"
    exit 0
fi

lag=$(psql -tAc \
  "SELECT COALESCE(EXTRACT(EPOCH FROM (now() - pg_last_xact_replay_timestamp())), 0)::int;")

echo "Replication lag: ${lag}s"

if [ "$lag" -ge "$CRIT_SECONDS" ]; then
    echo "CRITICAL: replication lag is ${lag}s (threshold ${CRIT_SECONDS}s)" >&2
    exit 2
elif [ "$lag" -ge "$WARN_SECONDS" ]; then
    echo "WARNING: replication lag is ${lag}s (threshold ${WARN_SECONDS}s)" >&2
    exit 1
fi
exit 0
```

## 4. Long-running transaction / query detector

```bash
#!/bin/bash
set -euo pipefail

THRESHOLD_MINUTES=15

long_queries=$(psql -tAc "
SELECT pid || '|' || now() - query_start AS duration || '|' || left(query, 60)
FROM pg_stat_activity
WHERE state != 'idle'
  AND now() - query_start > interval '${THRESHOLD_MINUTES} minutes'
  AND pid != pg_backend_pid();
")

if [ -z "$long_queries" ]; then                       # Ch.4 — string test, -z = empty
    echo "OK: no queries running longer than ${THRESHOLD_MINUTES} minutes"
    exit 0
fi

echo "WARNING: long-running queries detected:"
echo "$long_queries" | while IFS='|' read -r pid duration query; do   # Ch.9 — IFS split
    echo "  PID $pid running for $duration: $query"
done
exit 1
```

## 5. Disk space check for the data directory (ties into Linux Notes Ch.9)

```bash
#!/bin/bash
set -euo pipefail

WARN_PCT=80
CRIT_PCT=90
PGDATA="${PGDATA:-/var/lib/postgresql/16/main}"

usage=$(df --output=pcent "$PGDATA" | tail -1 | tr -d '% ')   # Ch.9 — parameter expansion

echo "PGDATA disk usage: ${usage}%"

if [ "$usage" -ge "$CRIT_PCT" ]; then
    echo "CRITICAL: PGDATA disk usage at ${usage}%" >&2
    exit 2
elif [ "$usage" -ge "$WARN_PCT" ]; then
    echo "WARNING: PGDATA disk usage at ${usage}%" >&2
    exit 1
fi
exit 0
```

## 6. Table/index bloat quick check

```bash
#!/bin/bash
set -euo pipefail

MIN_SIZE_MB=100
MAX_DEAD_PCT=20

bloated=$(psql -tAc "
SELECT relname || '|' || n_dead_tup || '|' || n_live_tup
FROM pg_stat_user_tables
WHERE n_live_tup > 0
  AND pg_total_relation_size(relid) > ${MIN_SIZE_MB} * 1024 * 1024
  AND (n_dead_tup::float / GREATEST(n_live_tup,1)) * 100 > ${MAX_DEAD_PCT};
")

if [ -z "$bloated" ]; then
    echo "OK: no significantly bloated tables found"
    exit 0
fi

echo "WARNING: bloated tables detected (dead tuple ratio > ${MAX_DEAD_PCT}%):"
echo "$bloated" | while IFS='|' read -r table dead live; do
    pct=$(( dead * 100 / (live + dead) ))
    echo "  $table: $dead dead / $live live (${pct}%)"
done
```

## 7. A combined health-check runner (functions + array, Ch.7 + Ch.8)

```bash
#!/bin/bash
#
# pg_healthcheck.sh — runs all checks, summarizes pass/fail
#
set -uo pipefail   # note: no -e here, we want to run every check even if one fails

CHECK_SCRIPTS=(
    "/opt/pg-scripts/pg_check_connectivity.sh"
    "/opt/pg-scripts/pg_check_connections.sh"
    "/opt/pg-scripts/pg_check_replication_lag.sh"
    "/opt/pg-scripts/pg_check_disk.sh"
)

failures=0
for check in "${CHECK_SCRIPTS[@]}"; do                    # Ch.8 — array iteration
    echo "--- Running $(basename "$check") ---"
    if "$check"; then
        echo "PASS"
    else
        echo "FAIL (exit code $?)"
        failures=$((failures + 1))
    fi
    echo
done

echo "======================================"
if [ "$failures" -eq 0 ]; then
    echo "All checks passed"
    exit 0
else
    echo "$failures check(s) failed"
    exit 1
fi
```

Schedule it every 5 minutes (Ch.13):
```cron
*/5 * * * * postgres /opt/pg-scripts/pg_healthcheck.sh >> /var/log/pg_healthcheck.log 2>&1
```

### Key takeaways

- `psql -tAc "query"` (tuples-only, unaligned) is how shell scripts pull single values or
  simple rows out of PostgreSQL for use in Bash logic.
- Standard monitoring exit-code convention: `0` = OK, `1` = WARNING, `2` = CRITICAL — this
  matches what Nagios-style monitoring systems expect.
- Two-threshold checks (`WARN`/`CRIT`) with the critical check ordered first in an `if/elif`
  chain is the standard shape for every monitoring script in this chapter.
- The combined runner pattern (array of scripts + loop + failure counter) scales cleanly as
  you add more checks over time.

## Lab Practice Tasks

1. Run `pg_check_connectivity.sh` against your lab instance, then stop PostgreSQL
   (`systemctl stop postgresql-18` or `pg_ctl stop -D /u01/pgsql/18/data`) and run it again —
   confirm it correctly reports CRITICAL.

2. Add a new check script, `pg_check_work_mem.sh`, that warns if `work_mem` is set below
   `4MB` — tying directly back to the Chapter 7 (Linux Notes) `grep` exercise:
   ```bash
   #!/bin/bash
   value=$(psql -tAc "SHOW work_mem;" | tr -d ' ')
   echo "work_mem is currently: $value"
   ```

3. Generate a long-running query on purpose (`SELECT pg_sleep(1000);`) and confirm
   `pg_check_long_queries.sh` catches it within your configured threshold.

4. Add your new script to the `CHECK_SCRIPTS` array in `pg_healthcheck.sh` and confirm the
   combined runner picks it up automatically.

**Next:** [Chapter 17 — PostgreSQL Maintenance & Operations Scripts](17-postgresql-maintenance-scripts.md)
**Previous:** [Chapter 15 — PostgreSQL Backup Automation Scripts](15-postgresql-backup-scripts.md)
