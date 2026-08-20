# Chapter 14: Best Practices & Advanced Techniques

This chapter pulls together everything from Chapters 1–13 into the shape a real,
production-grade shell script takes.

## 1. Standard script skeleton

```bash
#!/bin/bash
#
# script_name.sh — one-line description of what this does
# Usage: ./script_name.sh [-v] [-e environment] <required_arg>
#
set -euo pipefail

# --- Configuration -----------------------------------------------------
SCRIPT_NAME=$(basename "$0")
LOG_FILE="/var/log/${SCRIPT_NAME%.sh}.log"
VERBOSE=false

# --- Functions -----------------------------------------------------------
log()   { echo "[$(date '+%F %T')] $*" | tee -a "$LOG_FILE"; }
die()   { log "ERROR: $*"; exit "${2:-1}"; }
usage() { echo "Usage: $SCRIPT_NAME [-v] [-e environment] <required_arg>"; exit 1; }

cleanup() {
    log "Cleaning up..."
    rm -f /tmp/${SCRIPT_NAME}.$$.*
}
trap cleanup EXIT

# --- Argument parsing ------------------------------------------------------
while getopts "ve:" opt; do
    case $opt in
        v) VERBOSE=true ;;
        e) ENVIRONMENT="$OPTARG" ;;
        \?) usage ;;
    esac
done
shift $((OPTIND - 1))

[ $# -ge 1 ] || usage
required_arg="$1"

# --- Main logic ---------------------------------------------------------
main() {
    log "Starting $SCRIPT_NAME with arg: $required_arg"
    # ... actual work goes here ...
    log "Finished successfully"
}

main
```

This skeleton is worth memorizing — it's the same shape used by nearly every production
shell script: config at the top, small functions, `trap`-based cleanup, argument parsing,
then a `main` that ties it together.

## 2. Logging with levels

```bash
#!/bin/bash
LOG_LEVEL="${LOG_LEVEL:-INFO}"   # can be overridden by environment

log() {
    local level="$1"; shift
    local levels=(DEBUG INFO WARN ERROR)
    local msg="$*"
    # only print if this message's level is >= configured LOG_LEVEL
    echo "[$(date '+%F %T')] [$level] $msg"
}

log INFO "Service starting"
log WARN "Disk usage is high"
log ERROR "Connection failed"
```

## 3. Argument parsing with both flags and positional args

```bash
#!/bin/bash
verbose=false
output_dir="."

while [[ $# -gt 0 ]]; do
    case $1 in
        -v|--verbose) verbose=true; shift ;;
        -o|--output)  output_dir="$2"; shift 2 ;;
        -h|--help)    echo "Usage: $0 [-v] [-o dir] <file>"; exit 0 ;;
        -*) echo "Unknown option: $1" >&2; exit 1 ;;
        *)  break ;;
    esac
done

file="$1"
echo "verbose=$verbose output_dir=$output_dir file=$file"
```

## 4. Idempotency — scripts safe to run more than once

```bash
#!/bin/bash
# Bad: fails the second time it's run
mkdir /data/archive

# Good: idempotent
mkdir -p /data/archive

# Bad: duplicate cron entries if run twice
echo "0 2 * * * /script.sh" >> /etc/crontab

# Good: check before adding
grep -qF "/script.sh" /etc/crontab || echo "0 2 * * * /script.sh" >> /etc/crontab
```
Idempotent scripts (safe to re-run without side effects) are essential for automation and
configuration management — a deployment or setup script should never fail just because it
already ran once.

## 5. Shellcheck — static analysis for shell scripts

```bash
sudo apt install shellcheck    # or: dnf install ShellCheck
shellcheck myscript.sh
```
`shellcheck` catches an enormous class of real bugs (unquoted variables, unused variables,
common globbing mistakes) before they ever run. Running it on every script before deploying
it is standard practice on any serious team.

## 6. Version and environment guards

```bash
#!/bin/bash
if ((BASH_VERSINFO[0] < 4)); then
    echo "This script requires Bash 4 or newer" >&2
    exit 1
fi

if [ "$(id -u)" -ne 0 ]; then
    echo "This script must be run as root" >&2
    exit 1
fi
```

## 7. Configuration via external file

```bash
#!/bin/bash
CONFIG_FILE="${CONFIG_FILE:-/etc/myapp/config.env}"

if [ -f "$CONFIG_FILE" ]; then
    # shellcheck disable=SC1090
    source "$CONFIG_FILE"
else
    echo "Config file not found: $CONFIG_FILE" >&2
    exit 1
fi

echo "Connecting to $DB_HOST:$DB_PORT"
```
**`config.env`:**
```bash
DB_HOST=dbhost.internal
DB_PORT=5432
DB_NAME=appdb
```
Keeping environment-specific values out of the script itself (in a sourced config file)
means the same script can run in dev, staging, and prod without modification.

## 8. Notifications from scripts

```bash
#!/bin/bash
notify_slack() {
    local message="$1"
    curl -s -X POST -H 'Content-type: application/json' \
        --data "{\"text\":\"$message\"}" \
        "$SLACK_WEBHOOK_URL" > /dev/null
}

notify_slack "Backup completed successfully on $(hostname)"
```

### Key takeaways

- Use the standard skeleton (config → functions → trap → argument parsing → main) for any
  script meant to run unattended or be maintained by a team.
- Make scripts **idempotent** — safe to re-run without breaking anything.
- Run `shellcheck` on every script before relying on it.
- Externalize environment-specific configuration rather than hardcoding it.
- These fourteen chapters are the complete toolkit — Part 2 now applies every one of them
  directly to PostgreSQL operations.

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. Rewrite your `pg_conf_lookup.sh` (from Chapters 3–13) using the full standard skeleton
   from section 1 of this chapter: config block, `log()`/`die()` functions, `trap` cleanup,
   `getopts` argument parsing, and a `main()` function at the bottom.

2. Run `shellcheck` on your rewritten script and fix every warning it reports.

3. Externalize the PostgreSQL version and data path into a sourced config file
   (`pg_env.sh`), so the same script works against PG 16, 17, or 18 without code changes:
   ```bash
   # pg_env.sh
   PG_VERSION=18
   PGDATA="/u01/pgsql/${PG_VERSION}/data"
   ```

4. Add a Slack (or email) notification call to your script so it alerts you if
   `postgresql.conf` is ever missing or unreadable.

You now have every tool needed for Part 2 — three chapters of complete, real PostgreSQL
automation scripts using exactly these techniques.

**Next:** [Chapter 15 — PostgreSQL Backup Automation Scripts](15-postgresql-backup-scripts.md)
**Previous:** [Chapter 13 — Scheduling Scripts](13-scheduling-scripts.md)
