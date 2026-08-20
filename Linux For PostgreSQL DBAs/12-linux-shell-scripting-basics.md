# Chapter 12: Linux Shell Scripting Basics

Shell scripting lets you automate tasks by writing a series of commands in a file that the
shell can execute.

## 1. Overview

- A shell script is a text file with a list of commands.
- The most common shell is **Bash** (Bourne Again SHell).
- Script files usually end with `.sh`.
- Make it executable, then run it:

```bash
$ nano myscript.sh       # create script
$ chmod +x myscript.sh   # make executable
$ ./myscript.sh          # run script
```

## 2. Hello World Script

```bash
#!/bin/bash
# This is a comment
echo "Hello, World!"
echo "Welcome to Shell Scripting"
```

The first line is called the **shebang**. It tells the system which interpreter to use
(here: `/bin/bash`).

## 3. Variables

| Example | Description |
|---|---|
| `name="Linux"` | Assign string to variable |
| `age=25` | Assign number |
| `echo $name` | Use variable |
| `echo "Age: $age"` | Print variable |
| `unset name` | Delete variable |

**No spaces around `=`.** Variable names are case-sensitive.

## 4. Read Input From User

```bash
#!/bin/bash
echo -n "Enter your name: "
read username
echo "Hello, $username!"
```

| Element | Explanation |
|---|---|
| `read var` | Reads a line of input into `var` |
| `-n` | Do not move to the next line (for `echo`) |
| `-p "msg"` | Display prompt (`read -p "Name: " name`) |

## 5. Comments

```bash
# This is a single line comment

: <<'COMMENT'
This is a
multi-line comment
COMMENT
```

## 6. Arithmetic Operations

| Example | Description |
|---|---|
| `a=$((5+3))` | Addition |
| `b=$((10-2))` | Subtraction |
| `c=$((4*3))` | Multiplication |
| `d=$((20/4))` | Division |
| `e=$((20%3))` | Modulus (remainder) |

```bash
echo $a $b $c $d $e   # Output: 8 8 12 5 2
```

## 7. Conditional Statements

```bash
#!/bin/bash
read -p "Enter a number: " num
if [ $num -gt 10 ]; then
    echo "Number is greater than 10"
elif [ $num -eq 10 ]; then
    echo "Number is equal to 10"
else
    echo "Number is less than 10"
fi
```

### Common comparison operators

| Operator | Meaning |
|---|---|
| `-eq` | Equal to |
| `-ne` | Not equal |
| `-gt` | Greater than |
| `-lt` | Less than |
| `-ge` | Greater or equal |
| `-le` | Less or equal |

## 8. Loops

**For loop:**
```bash
#!/bin/bash
for i in 1 2 3 4 5
do
    echo "Number: $i"
done
```

**While loop:**
```bash
#!/bin/bash
count=1
while [ $count -le 5 ]
do
    echo "Count: $count"
    count=$((count+1))
done
```

**Until loop:**
```bash
#!/bin/bash
count=1
until [ $count -gt 5 ]
do
    echo "Count: $count"
    count=$((count+1))
done
```

## 9. Functions

```bash
#!/bin/bash
greet() {
    echo "Hello, $1!"
}

greet "Linux User"
greet "Developer"
```

- Define a function with `name() { ... }`.
- `$1`, `$2`, ... are positional parameters (arguments).
- Call the function by its name.

## 10. Command Line Arguments

```bash
#!/bin/bash
echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Total arguments: $#"
echo "All arguments: $@"
```

Run: `./script.sh arg1 arg2 arg3`. `$#` gives the total number of arguments.

## 11. Exit Status

| Example | Description |
|---|---|
| `command` | Run command |
| `echo $?` | Get exit status of last command |
| `exit 0` | Exit script with status 0 (success) |
| `exit 1` | Exit script with status 1 (failure) |

### Common exit codes

| Code | Meaning |
|---|---|
| 0 | Success |
| 1 | General error |
| 2 | Misuse of shell builtins |
| 126 | Command invoked cannot execute |
| 127 | Command not found |

## 12. Test Conditions (`[ ]`)

| Example | Description |
|---|---|
| `[ -f file.txt ]` | File exists |
| `[ -d dir ]` | Directory exists |
| `[ -r file ]` | File is readable |
| `[ -w file ]` | File is writable |
| `[ -x file ]` | File is executable |

| More tests | Description |
|---|---|
| `-s file` | File is not empty |
| `-z str` | String is empty |
| `-n str` | String is not empty |
| `str1 = str2` | Strings are equal |
| `str1 != str2` | Strings are not equal |

## 13. Simple Example Script

```bash
#!/bin/bash
# Simple backup script
echo "Starting backup..."
cp -r /home/user/data /backup/
if [ $? -eq 0 ]; then
    echo "Backup Successful!"
else
    echo "Backup Failed!"
    exit 1
fi
echo "Done."
```

What it does: copies data to `/backup/`, checks if the copy was successful using `$?`,
prints a success/failure message, and exits with the proper status.

### Tips

- Always put the shebang `#!/bin/bash` at the top.
- Add comments to make your script readable.
- Use meaningful variable and function names.
- Test your scripts with different inputs.
- Use `set -e` to exit immediately on error.

---

## Where a PostgreSQL DBA uses this

Shell scripting is where all the previous chapters come together into repeatable,
schedulable automation — the backbone of DBA operations.

### A real backup script

```bash
#!/bin/bash
set -e   # exit immediately if any command fails

DB_NAME="production"
BACKUP_DIR="/backups/postgres"
DATE=$(date +%F_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_${DATE}.dump"
RETENTION_DAYS=7

echo "Starting backup of $DB_NAME at $(date)"

pg_dump -Fc -d "$DB_NAME" -f "$BACKUP_FILE"

if [ $? -eq 0 ]; then
    echo "Backup successful: $BACKUP_FILE"
else
    echo "Backup FAILED for $DB_NAME" >&2
    exit 1
fi

# Delete backups older than the retention window
find "$BACKUP_DIR" -name "${DB_NAME}_*.dump" -mtime +$RETENTION_DAYS -exec rm {} \;

echo "Old backups cleaned up (older than $RETENTION_DAYS days)"
```

This uses: the shebang, `set -e`, variables, command substitution (`date`), `$?` exit-status
checking, an `if` conditional, and `find` from Chapter 7 — a direct extension of the "Simple
Example Script" pattern above.

### A replication/health-check script for cron

```bash
#!/bin/bash
# Checks if this PostgreSQL instance is in recovery (i.e. is a standby)
IS_REPLICA=$(psql -tAc "SELECT pg_is_in_recovery();")

if [ "$IS_REPLICA" = "t" ]; then
    LAG=$(psql -tAc "SELECT EXTRACT(EPOCH FROM (now() - pg_last_xact_replay_timestamp()));")
    echo "Standby replication lag: ${LAG}s"
    if (( $(echo "$LAG > 60" | bc -l) )); then
        echo "WARNING: replication lag exceeds 60 seconds" >&2
        exit 1
    fi
else
    echo "This is the primary — no lag to check."
fi
```

### A disk-space guard script (ties into Chapter 9)

```bash
#!/bin/bash
THRESHOLD=85
USAGE=$(df -h "$PGDATA" | awk 'NR==2 {print $5}' | tr -d '%')

if [ "$USAGE" -ge "$THRESHOLD" ]; then
    echo "ALERT: PGDATA disk usage is ${USAGE}% (threshold: ${THRESHOLD}%)" >&2
    exit 1
fi
echo "Disk usage OK: ${USAGE}%"
```

### Wiring scripts into automation

```bash
chmod +x /usr/local/bin/pg_backup.sh          # from Chapter 6 — must be executable
crontab -e
# 0 2 * * * /usr/local/bin/pg_backup.sh >> /var/log/pg_backup.log 2>&1
```

### Why this matters for a DBA specifically

- **`set -e` + exit-code checking** is what turns a fragile "it worked when I ran it by hand"
  script into something safe to run unattended at 2 AM.
- **Functions** let you build a personal library of reusable checks (connection test, replica
  lag check, disk check) that get sourced into many different maintenance scripts.
- **Command-line arguments (`$1`, `$2`, `$#`)** let one generic script back up *any* database
  name passed to it, rather than hardcoding one database per script.
- **Exit status conventions (0 = success, non-zero = failure)** are exactly what monitoring
  tools (cron + `mailx`, Nagios, Prometheus node exporter's `textfile` collector) key off of
  to decide whether to page someone.

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. **Write your first PostgreSQL-aware script.**
   ```bash
   cat > pg_where.sh <<'EOF'
   #!/bin/bash
   echo "Hostname: $(hostname)"
   echo "PostgreSQL config directory: /u01/pgsql/18/data"
   echo "work_mem setting:"
   grep work_mem /u01/pgsql/18/data/postgresql.conf
   EOF
   chmod +x pg_where.sh
   ./pg_where.sh
   ```
   This one script chains together Chapters 4 (paths), 7 (`grep`), and this chapter
   (scripting) into a single reusable tool.

2. **Add a conditional check.** Extend the script so it only tries to `grep` if the config
   file actually exists:
   ```bash
   #!/bin/bash
   CONF="/u01/pgsql/18/data/postgresql.conf"
   if [ -f "$CONF" ]; then
       grep work_mem "$CONF"
   else
       echo "Config file not found at $CONF"
   fi
   ```

3. **Full end-to-end exercise (replays the bootcamp's Day-1 homework as a script):** write a
   script `check_work_mem.sh` that takes a PostgreSQL version number as an argument, builds
   the path `/u01/pgsql/<version>/data/postgresql.conf`, and prints the `work_mem` line —
   applying the "Command Line Arguments" section of this chapter (`$1`) directly.

For a much deeper treatment of shell scripting — variables, functions, arrays, loops, error
handling — see the companion **Shell Scripting for PostgreSQL DBAs** series, which builds on
every exercise in this chapter.

**Next:** [Chapter 13 — Cheat Sheet & Roadmap](13-linux-cheatsheet-roadmap.md)
**Previous:** [Chapter 11 — Package Management](11-linux-package-management.md)
