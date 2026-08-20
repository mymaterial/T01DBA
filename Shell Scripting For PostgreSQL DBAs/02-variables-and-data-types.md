# Chapter 2: Variables & Data Types

## 1. Declaring and using variables

Bash has no strict data types — everything is stored as a string internally, but Bash treats
strings as numbers when doing arithmetic.

```bash
#!/bin/bash
name="PostgreSQL"
version=16
is_running=true

echo "Database: $name"
echo "Version: $version"
echo "Running: $is_running"
```

**Rules:**
- No spaces around `=` — `name = "value"` is a syntax error; `name="value"` is correct.
- Variable names are case-sensitive: `Name` and `name` are different variables.
- Reference a variable with `$name` or `${name}` (braces avoid ambiguity, e.g. `${name}_backup`).

## 2. Read-only and unset

```bash
#!/bin/bash
readonly PI=3.14159
echo "Pi is $PI"
# PI=3.14   # this line would now cause an error: readonly variable

name="temp"
unset name
echo "After unset: '$name'"   # prints empty string
```

## 3. Environment variables

Environment variables are available to child processes; regular shell variables are not.

```bash
#!/bin/bash
export PGHOST="localhost"
export PGPORT=5432

# Any command this script calls (like psql) can now see PGHOST/PGPORT automatically
psql -c "SELECT 1;"
```

Common built-in environment variables:

| Variable | Meaning |
|---|---|
| `$HOME` | Current user's home directory |
| `$USER` | Current username |
| `$PATH` | Directories searched for executables |
| `$PWD` | Current working directory |
| `$SHELL` | Path to the current shell |
| `$HOSTNAME` | Machine's hostname |

```bash
#!/bin/bash
echo "Running as $USER on $HOSTNAME, home is $HOME"
```

## 4. Special/automatic variables

| Variable | Meaning |
|---|---|
| `$0` | Script name |
| `$$` | Current process ID (PID) |
| `$?` | Exit status of last command |
| `$RANDOM` | Random integer (0–32767) |
| `$SECONDS` | Seconds since the script started |

```bash
#!/bin/bash
echo "Script: $0"
echo "PID: $$"
echo "Random number: $RANDOM"
sleep 2
echo "Elapsed: $SECONDS seconds"
```

## 5. Command substitution — assigning command output to a variable

```bash
#!/bin/bash
today=$(date +%F)
file_count=$(ls | wc -l)
echo "Today's date: $today"
echo "Files here: $file_count"
```
`$(command)` is preferred over the older backtick syntax `` `command` `` — it nests more
cleanly.

## 6. Numeric vs string context

```bash
#!/bin/bash
a=5
b=3
echo "String concat: $a$b"       # 53 (just placed next to each other)
echo "Arithmetic sum: $((a+b))"  # 8  (arithmetic context via double parentheses)
```

## 7. Default values and variable existence

```bash
#!/bin/bash
# Use a default if VAR is unset or empty
name="${1:-default_user}"
echo "Hello, $name"

# Assign a default AND save it into the variable if unset
: "${LOG_LEVEL:=INFO}"
echo "Log level: $LOG_LEVEL"
```

### Key takeaways

- No spaces around `=`; always quote variables in general use: `"$var"`.
- `export` makes a variable visible to child processes (important for `psql`, `pg_dump`, etc.).
- `$(command)` captures command output into a variable.
- `${var:-default}` is the standard pattern for safe defaults.

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. Create variables for every part of your PostgreSQL path, then combine them:
   ```bash
   PG_HOME="/u01/pgsql"
   PG_VERSION="18"
   PGDATA="${PG_HOME}/${PG_VERSION}/data"
   echo "Config file: ${PGDATA}/postgresql.conf"
   ```

2. `export PGDATA` and confirm a child process can see it:
   ```bash
   export PGDATA="/u01/pgsql/18/data"
   bash -c 'echo "Child sees PGDATA=$PGDATA"'
   ```

3. Use `${VAR:-default}` to write a one-liner that prints `PG_VERSION` if it's set, or falls
   back to `"18"` if it isn't — test it both with and without `PG_VERSION` set first.

4. Use `$?` after running `grep work_mem "$PGDATA/postgresql.conf"` to confirm whether the
   setting was found (exit 0) or not (exit 1).

**Next:** [Chapter 3 — User Input & Arguments](03-user-input-and-arguments.md)
**Previous:** [Chapter 1 — Getting Started](01-getting-started.md)
