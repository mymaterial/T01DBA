# Chapter 17: PostgreSQL Maintenance & Operations Scripts

The last chapter: scripts that actively *do* maintenance work, not just check status —
vacuum/analyze automation, log rotation, a failover helper, and alert notifications tying
everything in this series together.

## 1. Scheduled VACUUM / ANALYZE for tables autovacuum is falling behind on

```bash
#!/bin/bash
#
# pg_vacuum_maintenance.sh — manually vacuum/analyze tables with high dead-tuple ratios
#
set -euo pipefail

MAX_DEAD_PCT=20
LOG_FILE="/var/log/pg_vacuum_maintenance.log"

log() { echo "[$(date '+%F %T')] $*" | tee -a "$LOG_FILE"; }

log "Scanning for tables needing manual VACUUM"

mapfile -t tables_to_vacuum < <(psql -tAc "
SELECT schemaname || '.' || relname
FROM pg_stat_user_tables
WHERE n_live_tup > 0
  AND (n_dead_tup::float / GREATEST(n_live_tup, 1)) * 100 > ${MAX_DEAD_PCT};
")

if [ ${#tables_to_vacuum[@]} -eq 0 ]; then
    log "No tables need manual vacuuming right now"
    exit 0
fi

log "Found ${#tables_to_vacuum[@]} table(s) needing VACUUM"

for table in "${tables_to_vacuum[@]}"; do
    log "Vacuuming $table..."
    if psql -c "VACUUM (ANALYZE, VERBOSE) $table;" &>> "$LOG_FILE"; then
        log "  Done: $table"
    else
        log "  FAILED: $table"
    fi
done

log "Manual vacuum pass complete"
```

## 2. Periodic REINDEX for a specific bloated index (with concurrency safety)

```bash
#!/bin/bash
#
# pg_reindex_concurrent.sh — reindex a given index without blocking writes
#
set -euo pipefail

INDEX_NAME="${1:?Usage: $0 <index_name>}"
LOG_FILE="/var/log/pg_reindex.log"

log() { echo "[$(date '+%F %T')] $*" | tee -a "$LOG_FILE"; }
die() { log "ERROR: $*"; exit 1; }

log "Starting CONCURRENT reindex of $INDEX_NAME"

# REINDEX CONCURRENTLY cannot run inside a transaction block —
# psql runs each -c statement in its own implicit transaction, which is what we want here
if psql -c "REINDEX INDEX CONCURRENTLY $INDEX_NAME;"; then
    log "Reindex successful: $INDEX_NAME"
else
    die "Reindex failed for $INDEX_NAME — check for invalid index left behind"
fi
```

## 3. PostgreSQL log rotation and compression (for setups not using `logrotate`)

```bash
#!/bin/bash
#
# pg_log_rotate.sh — compress logs older than 1 day, delete logs older than 30 days
#
set -euo pipefail

LOG_DIR="/var/log/postgresql"
COMPRESS_AFTER_DAYS=1
DELETE_AFTER_DAYS=30

echo "Compressing logs older than $COMPRESS_AFTER_DAYS day(s)..."
find "$LOG_DIR" -name "*.log" -mtime +"$COMPRESS_AFTER_DAYS" ! -name "*.gz" -print0 \
    | while IFS= read -r -d '' file; do
        gzip "$file"
        echo "Compressed: $file"
    done

echo "Deleting logs older than $DELETE_AFTER_DAYS day(s)..."
find "$LOG_DIR" -name "*.log.gz" -mtime +"$DELETE_AFTER_DAYS" -print -delete
```
(Using `-print0`/`read -d ''` instead of plain newline splitting protects against filenames
with spaces — the fully safe version of the pattern introduced in Chapter 12.)

## 4. Failover helper script (promote a standby)

**This is a high-impact operation — review carefully and test in staging before production
use. It's written defensively on purpose.**

```bash
#!/bin/bash
#
# pg_promote_standby.sh — safely promote a PostgreSQL standby to primary
#
set -euo pipefail

PGDATA="${PGDATA:?PGDATA must be set}"
LOG_FILE="/var/log/pg_promote.log"
LOCKFILE="/tmp/pg_promote.lock"

log() { echo "[$(date '+%F %T')] $*" | tee -a "$LOG_FILE"; }
die() { log "ERROR: $*"; exit 1; }

exec 200>"$LOCKFILE"
flock -n 200 || die "A promotion is already in progress"

# --- Safety checks before doing anything irreversible -----------------------
is_replica=$(psql -tAc "SELECT pg_is_in_recovery();" | tr -d '[:space:]')
[ "$is_replica" = "t" ] || die "This server is NOT a standby — refusing to promote"

read -p "Confirm promotion of $(hostname) to PRIMARY? Type 'yes' to continue: " confirm
[ "$confirm" = "yes" ] || die "Promotion cancelled by operator"

lag=$(psql -tAc \
  "SELECT COALESCE(EXTRACT(EPOCH FROM (now() - pg_last_xact_replay_timestamp())), 0)::int;")
log "Current replication lag before promotion: ${lag}s"

if [ "$lag" -gt 5 ]; then
    read -p "Lag is ${lag}s, some data may be lost. Continue anyway? (yes/no): " force
    [ "$force" = "yes" ] || die "Promotion cancelled — lag too high"
fi

# --- Promote ------------------------------------------------------------
log "Promoting standby..."
if pg_ctl promote -D "$PGDATA"; then
    log "Promotion command issued successfully"
else
    die "pg_ctl promote failed"
fi

# --- Wait and verify ------------------------------------------------------
log "Waiting for promotion to complete..."
for i in {1..30}; do                                     # Ch.6 — for loop with range
    still_replica=$(psql -tAc "SELECT pg_is_in_recovery();" | tr -d '[:space:]')
    if [ "$still_replica" = "f" ]; then
        log "Promotion CONFIRMED — this server is now the primary"
        exit 0
    fi
    sleep 1
done

die "Promotion did not complete within 30 seconds — check manually!"
```

## 5. Sending alerts from any script in this series (Slack example, Ch.14)

```bash
#!/bin/bash
#
# notify.sh — reusable notification helper, source this from other scripts
#
notify_slack() {
    local message="$1"
    local webhook_url="${SLACK_WEBHOOK_URL:?SLACK_WEBHOOK_URL not set}"
    curl -s -X POST -H 'Content-type: application/json' \
        --data "{\"text\":\"$message\"}" "$webhook_url" > /dev/null
}

notify_email() {
    local subject="$1"
    local body="$2"
    local to="${ALERT_EMAIL:?ALERT_EMAIL not set}"
    echo "$body" | mail -s "$subject" "$to"
}
```
Used from the health-check script in Chapter 16:
```bash
#!/bin/bash
source /opt/pg-scripts/notify.sh

if ! /opt/pg-scripts/pg_healthcheck.sh; then
    notify_slack "PostgreSQL health check FAILED on $(hostname)"
    notify_email "PG Alert: $(hostname)" "Health check failed — see /var/log/pg_healthcheck.log"
fi
```

## 6. Putting it all together: a daily maintenance orchestrator

```bash
#!/bin/bash
#
# pg_daily_maintenance.sh — the single script cron calls once a day
#
set -uo pipefail
source /opt/pg-scripts/notify.sh

LOG_FILE="/var/log/pg_daily_maintenance.log"
log() { echo "[$(date '+%F %T')] $*" | tee -a "$LOG_FILE"; }

STEPS=(
    "/opt/pg-scripts/pg_backup_all.sh"
    "/opt/pg-scripts/pg_vacuum_maintenance.sh"
    "/opt/pg-scripts/pg_log_rotate.sh"
    "/opt/pg-scripts/pg_healthcheck.sh"
)

failed_steps=()
for step in "${STEPS[@]}"; do
    log "=== Running $(basename "$step") ==="
    if ! "$step"; then
        log "!!! $(basename "$step") FAILED !!!"
        failed_steps+=("$(basename "$step")")
    fi
done

if [ ${#failed_steps[@]} -eq 0 ]; then
    log "Daily maintenance completed successfully"
else
    msg="Daily maintenance on $(hostname) had failures: ${failed_steps[*]}"
    log "$msg"
    notify_slack "$msg"
    exit 1
fi
```
Scheduled with a systemd timer (Ch.13) rather than raw cron, so failures are visible in
`journalctl -u pg-daily-maintenance.service` alongside every other service on the box.

### Key takeaways

- Maintenance scripts that make irreversible changes (promotion, `REINDEX`, mass `VACUUM`)
  should always include explicit confirmation prompts and pre-flight safety checks — the cost
  of a wasted 10 seconds asking "are you sure?" is far lower than the cost of promoting the
  wrong server.
- `REINDEX CONCURRENTLY` and similar non-transactional commands need to run as standalone
  `psql -c` calls, not batched inside a single transaction.
- A single orchestrator script (array of steps + loop + failure tracking) that calls out to
  the smaller, single-purpose scripts from Chapters 15–17 is the natural endpoint of this
  whole series — exactly how real DBA automation is structured in production.
- Every technique from Part 1 — variables, functions, arrays, loops, conditionals, string
  handling, redirection, error handling, file safety, and scheduling — shows up directly in
  these three PostgreSQL chapters. That's the whole point of learning shell scripting as a
  DBA: it's not a separate skill from database administration, it *is* database
  administration.

## Capstone Lab Practice Tasks

1. On a disposable test instance only, walk through `pg_promote_standby.sh` against a real
   streaming replica you've set up in your lab — confirm every safety check (recovery-mode
   check, confirmation prompt, lag check) actually stops you if you answer "no".

2. Build your own `pg_daily_maintenance.sh` from scratch, using only scripts you wrote
   yourself across Chapters 15–17 (not copy-pasted), pointing everything at
   `/u01/pgsql/18/data`.

3. Schedule it with a systemd timer (Chapter 13) instead of cron, and verify a full run's
   logs in `journalctl -u pg-daily-maintenance.service`.

4. **Final integration exercise** — replay the entire bootcamp arc in one script: given a
   PostgreSQL version as `$1`, your script should locate `PGDATA`, confirm the instance is
   running, print the value of `work_mem` (the original Day-1 homework), and finally run a
   backup — using variables, functions, conditionals, `grep`, and error handling from every
   chapter in this series:
   ```bash
   #!/bin/bash
   set -euo pipefail
   VERSION="${1:?Usage: $0 <pg_version>}"
   PGDATA="/u01/pgsql/${VERSION}/data"

   [ -d "$PGDATA" ] || { echo "PGDATA not found: $PGDATA" >&2; exit 1; }
   pg_isready -q || { echo "PostgreSQL is not running" >&2; exit 1; }

   echo "work_mem: $(grep '^work_mem' "$PGDATA/postgresql.conf" || echo 'using default 4MB')"
   pg_dump -Fc -d postgres -f "/tmp/postgres_$(date +%F).dump"
   echo "Done."
   ```

If you can write that last script without referring back to any chapter, you've completed
both series end to end.

**Previous:** [Chapter 16 — PostgreSQL Health Check & Monitoring Scripts](16-postgresql-monitoring-scripts.md)
**Back to:** [Index](00-index.md)
