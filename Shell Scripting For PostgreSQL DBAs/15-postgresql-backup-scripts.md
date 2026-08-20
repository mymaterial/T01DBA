# Chapter 15: PostgreSQL Backup Automation Scripts

This chapter writes complete, runnable scripts for the three most common PostgreSQL backup
patterns: logical backup (`pg_dump`), physical base backup (`pg_basebackup`), and WAL
archiving. Every script uses techniques from Part 1 — cited inline so you can see exactly
where each concept comes from.

## 1. Logical backup with `pg_dump` — single database

```bash
#!/bin/bash
#
# pg_backup.sh — logical backup of a single PostgreSQL database with retention cleanup
#
set -euo pipefail                                    # Ch.11 — fail fast, catch unset vars

# --- Configuration -------------------------------------------------------
DB_NAME="${1:?Usage: $0 <database_name>}"             # Ch.3  — required positional arg
BACKUP_DIR="/backups/postgres"
RETENTION_DAYS=7
DATE=$(date +%F_%H%M%S)                                 # Ch.2  — command substitution
BACKUP_FILE="${BACKUP_DIR}/${DB_NAME}_${DATE}.dump"     # Ch.9  — dynamic filename
LOG_FILE="/var/log/pg_backup.log"

# --- Functions -------------------------------------------------------------
log() { echo "[$(date '+%F %T')] $*" | tee -a "$LOG_FILE"; }   # Ch.10 — tee
die() { log "ERROR: $*"; exit 1; }

# --- Pre-flight checks (Ch.5, Ch.12) --------------------------------------
mkdir -p "$BACKUP_DIR"                                    # Ch.12 — mkdir -p, idempotent
command -v pg_dump &> /dev/null || die "pg_dump not found on PATH"

# --- Main ------------------------------------------------------------------
log "Starting backup of database: $DB_NAME"

if pg_dump -Fc -d "$DB_NAME" -f "$BACKUP_FILE"; then      # Ch.5 — if on command exit status
    log "Backup successful: $BACKUP_FILE ($(du -h "$BACKUP_FILE" | cut -f1))"
else
    die "pg_dump failed for $DB_NAME"
fi

# --- Retention cleanup (Ch.6, Ch.12) ---------------------------------------
log "Cleaning up backups older than $RETENTION_DAYS days"
find "$BACKUP_DIR" -name "${DB_NAME}_*.dump" -mtime +"$RETENTION_DAYS" -print -delete \
    | while IFS= read -r old; do log "Removed old backup: $old"; done

log "Backup job complete"
```

Run it:
```bash
chmod +x pg_backup.sh
./pg_backup.sh appdb
```

## 2. Backing up ALL databases in a loop (Ch.6 loops + Ch.8 arrays)

```bash
#!/bin/bash
set -euo pipefail

BACKUP_DIR="/backups/postgres"
DATE=$(date +%F)
LOG_FILE="/var/log/pg_backup_all.log"

log() { echo "[$(date '+%F %T')] $*" | tee -a "$LOG_FILE"; }

mkdir -p "$BACKUP_DIR"

# Get all non-template databases into an array (Ch.8 — mapfile)
mapfile -t databases < <(psql -tAc \
    "SELECT datname FROM pg_database WHERE datistemplate = false;")

log "Found ${#databases[@]} databases to back up"     # Ch.8 — array length

failures=0
for db in "${databases[@]}"; do                          # Ch.6 — for loop over array
    file="${BACKUP_DIR}/${db}_${DATE}.dump"
    log "Backing up $db -> $file"
    if pg_dump -Fc -d "$db" -f "$file"; then
        log "  OK: $db ($(du -h "$file" | cut -f1))"
    else
        log "  FAILED: $db"
        failures=$((failures + 1))                         # Ch.4 — arithmetic
    fi
done

if [ "$failures" -gt 0 ]; then                             # Ch.5 — conditional
    log "$failures database(s) failed to back up"
    exit 1
fi
log "All databases backed up successfully"
```

## 3. Physical backup with `pg_basebackup` (for point-in-time recovery / standbys)

```bash
#!/bin/bash
#
# pg_base_backup.sh — full physical base backup, compressed and timestamped
#
set -euo pipefail

BACKUP_ROOT="/backups/pg_basebackups"
DATE=$(date +%F_%H%M%S)
TARGET_DIR="${BACKUP_ROOT}/${DATE}"
LOCKFILE="/tmp/pg_basebackup.lock"
LOG_FILE="/var/log/pg_basebackup.log"

log() { echo "[$(date '+%F %T')] $*" | tee -a "$LOG_FILE"; }
die() { log "ERROR: $*"; exit 1; }

# --- Locking (Ch.12) — never allow two base backups to run at once ---------
exec 200>"$LOCKFILE"
flock -n 200 || die "Another base backup is already running"

trap 'log "Base backup script exiting"' EXIT              # Ch.11 — trap

mkdir -p "$TARGET_DIR"
log "Starting base backup into $TARGET_DIR"

pg_basebackup \
    -D "$TARGET_DIR" \
    -F tar \
    -z \
    -X stream \
    -c fast \
    -P \
    -v \
    || die "pg_basebackup failed"

log "Base backup completed: $(du -sh "$TARGET_DIR" | cut -f1)"

# Retention: keep only the 5 most recent base backups (Ch.6, Ch.9)
cd "$BACKUP_ROOT"
ls -1dt */ | tail -n +6 | while IFS= read -r old_dir; do    # Ch.6 — while read
    log "Removing old base backup: $old_dir"
    rm -rf "${old_dir:?}"
done

log "Retention cleanup complete"
```

## 4. Uploading backups to S3-compatible remote storage

```bash
#!/bin/bash
set -euo pipefail

BACKUP_FILE="${1:?Usage: $0 <backup_file>}"
S3_BUCKET="s3://company-pg-backups"
LOG_FILE="/var/log/pg_backup_upload.log"

log() { echo "[$(date '+%F %T')] $*" | tee -a "$LOG_FILE"; }
die() { log "ERROR: $*"; exit 1; }

[ -f "$BACKUP_FILE" ] || die "File not found: $BACKUP_FILE"     # Ch.12 — file test

command -v aws &> /dev/null || die "aws CLI not installed"

log "Uploading $BACKUP_FILE to $S3_BUCKET"
if aws s3 cp "$BACKUP_FILE" "${S3_BUCKET}/$(basename "$BACKUP_FILE")"; then
    log "Upload successful"
else
    die "Upload failed"
fi
```

Chain it onto the earlier backup script:
```bash
./pg_backup.sh appdb && ./pg_backup_upload.sh "/backups/postgres/appdb_$(date +%F)*.dump"
```

## 5. WAL archive command (used inside `postgresql.conf`)

PostgreSQL calls this script itself, once per WAL segment — it doesn't run on a schedule.

```bash
#!/bin/bash
#
# archive_wal.sh — copy a completed WAL segment to the archive location
# Called by PostgreSQL as: archive_command = '/usr/local/bin/archive_wal.sh %p %f'
#
set -euo pipefail

WAL_PATH="$1"    # %p — full path to the WAL file to archive
WAL_FILE="$2"    # %f — just the filename
ARCHIVE_DIR="/mnt/wal_archive"

DEST="${ARCHIVE_DIR}/${WAL_FILE}"

if [ -f "$DEST" ]; then
    # PostgreSQL requires archive_command to fail if the destination already exists
    # and differs — this check protects against silently overwriting WAL history
    echo "ERROR: $DEST already exists" >&2
    exit 1
fi

cp "$WAL_PATH" "$DEST" && sync
```
Corresponding `postgresql.conf` setting:
```
archive_mode = on
archive_command = '/usr/local/bin/archive_wal.sh %p %f'
```

### Key takeaways

- `pg_dump` scripts follow the same skeleton every time: pre-flight checks → backup →
  verify → retention cleanup, wrapped in `set -euo pipefail` and logged with `tee`.
- `flock`-based locking (Ch.12) is essential for `pg_basebackup` — running two concurrently
  against the same server can be very expensive in I/O and disk.
- `find ... -mtime +N -delete` is the standard retention pattern across all backup types.
- The WAL `archive_command` script is special: it's invoked by PostgreSQL itself per-segment,
  must be fast, and must follow PostgreSQL's exact contract for exit codes and idempotency.

## Lab Practice Tasks

1. Adjust the `PG_HOME`/`PGDATA` paths in every script in this chapter to match your lab's
   actual layout (`/u01/pgsql/18/data`, or wherever Chapter 3 of the Linux Notes series found
   it on your machine), and run `pg_backup.sh` against a real (or test) database.

2. Confirm the backup actually worked using techniques from earlier chapters:
   ```bash
   ls -lh /backups/postgres/          # Linux Notes Ch.5
   file /backups/postgres/*.dump        # confirm it's a valid pg_dump custom-format file
   ```

3. Break the script on purpose — point `BACKUP_DIR` at a location you don't have write
   access to — and confirm your `set -euo pipefail` correctly stops the script rather than
   silently continuing.

4. Extend the retention cleanup to log exactly which files it would delete *before* deleting
   them (a "dry run" mode), using an `if [ "$DRY_RUN" = true ]` guard around the `-delete`
   flag — reusing Chapter 5's conditionals and Chapter 3's argument parsing.

**Next:** [Chapter 16 — PostgreSQL Health Check & Monitoring Scripts](16-postgresql-monitoring-scripts.md)
**Previous:** [Chapter 14 — Best Practices & Advanced Techniques](14-best-practices-and-advanced-techniques.md)
