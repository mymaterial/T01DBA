# Chapter 6: Loops

## 1. `for` loop — over a list

```bash
#!/bin/bash
for fruit in apple banana orange; do
    echo "Fruit: $fruit"
done
```

## 2. `for` loop — over a range

```bash
#!/bin/bash
for i in {1..5}; do
    echo "Number: $i"
done

# with a step
for i in {0..20..5}; do
    echo "Step: $i"
done

# C-style
for ((i = 1; i <= 5; i++)); do
    echo "C-style: $i"
done
```

## 3. `for` loop — over files

```bash
#!/bin/bash
for file in /var/log/*.log; do
    echo "Found log: $file"
done
```

## 4. `for` loop — over command output

```bash
#!/bin/bash
for user in $(cut -d: -f1 /etc/passwd); do
    echo "System user: $user"
done
```
Note: this splits on whitespace, which breaks if values contain spaces — see Chapter 8 for
the safer array-based pattern when that matters.

## 5. `while` loop

```bash
#!/bin/bash
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    count=$((count+1))
done
```

**`while` reading a file line by line** — one of the most useful loop patterns in DBA scripting:

```bash
#!/bin/bash
while IFS= read -r line; do
    echo "Line: $line"
done < server_list.txt
```
`IFS=` and `-r` prevent word-splitting and backslash-mangling, so each line is preserved
exactly as written — always use this pattern when processing files line by line.

**Infinite loop with a break condition** (common for monitoring scripts):

```bash
#!/bin/bash
while true; do
    echo "Checking status at $(date)..."
    sleep 5
    # break condition would go here, e.g.:
    # if [ -f /tmp/stop_signal ]; then break; fi
done
```

## 6. `until` loop

Runs **until** a condition becomes true (opposite of `while`).

```bash
#!/bin/bash
count=1
until [ $count -gt 5 ]; do
    echo "Count: $count"
    count=$((count+1))
done
```

A very common DBA use: poll until a service becomes available.

```bash
#!/bin/bash
until pg_isready -h localhost -p 5432 -q; do
    echo "Waiting for PostgreSQL to accept connections..."
    sleep 2
done
echo "PostgreSQL is ready!"
```

## 7. `select` loop — simple menus

```bash
#!/bin/bash
echo "Choose an environment:"
select env in "dev" "staging" "prod" "quit"; do
    case $env in
        quit) echo "Bye"; break ;;
        *) echo "You selected: $env" ;;
    esac
done
```

## 8. Loop control: `break` and `continue`

```bash
#!/bin/bash
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        echo "Skipping 5"
        continue
    fi
    if [ $i -eq 8 ]; then
        echo "Stopping at 8"
        break
    fi
    echo "i = $i"
done
```

`break N` / `continue N` skip out of N nested loop levels, useful in nested loops.

## 9. Nested loops

```bash
#!/bin/bash
for db in appdb reportdb; do
    for table in users orders; do
        echo "Checking $db.$table"
    done
done
```

### Key takeaways

- `for` for known lists/ranges/files; `while` for condition-driven or line-by-line reading;
  `until` for "wait until ready" patterns.
- Always use `while IFS= read -r line; do ... done < file` to safely process a file line by
  line.
- `break`/`continue` give you fine control inside loops; both accept an optional level count
  for nested loops.

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. Loop over a fixed list of settings and grep each one from `postgresql.conf`:
   ```bash
   for setting in work_mem shared_buffers max_connections wal_level; do
       echo "--- $setting ---"
       grep "$setting" /u01/pgsql/18/data/postgresql.conf
   done
   ```

2. Use `while IFS= read -r line; do ... done < postgresql.conf` to count how many
   non-comment, non-blank lines the file has (compare your count against `grep -vc '^#\|^$'`).

3. Write an `until` loop that polls `pg_isready` every 2 seconds until PostgreSQL is up,
   printing "waiting..." each time it isn't.

4. Use a `for` loop over `{1..5}` combined with `psql -tAc "SELECT pg_sleep(1);"` to fire five
   quick test queries, timing the total with `$SECONDS` (Chapter 2).

**Next:** [Chapter 7 — Functions](07-functions.md)
**Previous:** [Chapter 5 — Conditionals](05-conditionals.md)
