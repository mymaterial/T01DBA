# Chapter 5: Conditional Statements

## 1. `if` / `elif` / `else`

```bash
#!/bin/bash
read -p "Enter a number: " num

if [ $num -gt 10 ]; then
    echo "Greater than 10"
elif [ $num -eq 10 ]; then
    echo "Equal to 10"
else
    echo "Less than 10"
fi
```

## 2. Nested conditionals

```bash
#!/bin/bash
read -p "Enter username: " user
read -p "Enter password: " pass

if [ "$user" = "admin" ]; then
    if [ "$pass" = "secret" ]; then
        echo "Access granted"
    else
        echo "Wrong password"
    fi
else
    echo "Unknown user"
fi
```

## 3. `case` statements

`case` is cleaner than a long `if/elif` chain when matching one variable against several
patterns.

```bash
#!/bin/bash
read -p "Enter a fruit: " fruit

case $fruit in
    apple)
        echo "Apples are $1.50/lb"
        ;;
    banana|plantain)
        echo "Bananas are $0.60/lb"
        ;;
    orange)
        echo "Oranges are $1.20/lb"
        ;;
    *)
        echo "Unknown fruit"
        ;;
esac
```

`case` also supports wildcard-style patterns:

```bash
#!/bin/bash
read -p "Enter a filename: " file

case $file in
    *.txt) echo "Text file" ;;
    *.sh)  echo "Shell script" ;;
    *.sql) echo "SQL file" ;;
    *)     echo "Unknown file type" ;;
esac
```

## 4. File and directory test conditions

| Test | Meaning |
|---|---|
| `-e file` | File exists |
| `-f file` | Regular file exists |
| `-d dir` | Directory exists |
| `-r file` | File is readable |
| `-w file` | File is writable |
| `-x file` | File is executable |
| `-s file` | File exists and is not empty |
| `file1 -nt file2` | file1 is newer than file2 |
| `file1 -ot file2` | file1 is older than file2 |

```bash
#!/bin/bash
config="/etc/myapp/config.conf"

if [ -f "$config" ]; then
    echo "Config file found"
    if [ -r "$config" ]; then
        echo "And it's readable"
    fi
else
    echo "Config file missing!"
fi
```

## 5. Combining multiple conditions

```bash
#!/bin/bash
read -p "Enter age: " age
read -p "Have ticket? (yes/no): " ticket

if [ "$age" -ge 18 ] && [ "$ticket" = "yes" ]; then
    echo "Entry allowed"
else
    echo "Entry denied"
fi
```

## 6. A practical example: environment-aware script

```bash
#!/bin/bash
env="${1:-dev}"

case $env in
    prod)
        db_host="prod-db.internal"
        log_level="WARN"
        ;;
    staging)
        db_host="staging-db.internal"
        log_level="INFO"
        ;;
    dev)
        db_host="localhost"
        log_level="DEBUG"
        ;;
    *)
        echo "Unknown environment: $env" >&2
        exit 1
        ;;
esac

echo "Connecting to $db_host with log level $log_level"
```

### Key takeaways

- `if/elif/else` for general branching; `case` for matching one value against many patterns.
- File-test operators (`-f`, `-d`, `-r`, `-w`, `-x`) are essential for safe scripts that check
  before acting.
- Always quote variables inside test expressions: `[ "$var" = "value" ]`, to avoid errors
  when the variable is empty or contains spaces.

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. Write a script that checks whether `/u01/pgsql/18/data/postgresql.conf` exists; if not,
   check `/u01/pgsql/17/data/postgresql.conf`, then `/etc/postgresql/16/main/postgresql.conf`
   — using `if/elif/else` to support multiple possible install layouts.

2. Write a `case` statement that takes a distro name (`rhel`, `centos`, `ubuntu`, `debian`)
   as `$1` and prints the correct PostgreSQL config path convention for that distro (see
   Linux Notes Chapter 3 for the two conventions).

3. Combine file tests: write a check that PGDATA (`-d`), `postgresql.conf` (`-f`), and
   `pg_hba.conf` (`-f`) all exist, printing a clear message for whichever one is missing.

4. Extend exercise 3 into a `case` on file extension: given any filename under PGDATA, print
   whether it's a `.conf` file, a `.pid` file, or something else.

**Next:** [Chapter 6 — Loops](06-loops.md)
**Previous:** [Chapter 4 — Operators](04-operators.md)
