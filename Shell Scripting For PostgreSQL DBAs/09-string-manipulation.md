# Chapter 9: String Manipulation

## 1. String length

```bash
#!/bin/bash
str="PostgreSQL"
echo "${#str}"        # 10
```

## 2. Substrings

```bash
#!/bin/bash
str="PostgreSQL Database"
echo "${str:0:10}"     # PostgreSQL   (start at 0, length 10)
echo "${str:11}"        # Database     (from position 11 to end)
echo "${str: -8}"        # Database     (last 8 characters — note the space before -8)
```

## 3. Search and replace

```bash
#!/bin/bash
str="I love MySQL, MySQL is great"

echo "${str/MySQL/PostgreSQL}"    # replace first occurrence
echo "${str//MySQL/PostgreSQL}"    # replace ALL occurrences
```
Output:
```
I love PostgreSQL, MySQL is great
I love PostgreSQL, PostgreSQL is great
```

## 4. Removing prefixes/suffixes (pattern trimming)

```bash
#!/bin/bash
file="backup_2026-08-16.sql.gz"

echo "${file#*_}"        # 2026-08-16.sql.gz     (remove shortest match from front)
echo "${file##*_}"        # 2026-08-16.sql.gz     (remove longest match from front)
echo "${file%.*}"          # backup_2026-08-16.sql (remove shortest match from back)
echo "${file%%.*}"          # backup_2026-08-16     (remove longest match from back)
```

| Syntax | Meaning |
|---|---|
| `${var#pattern}` | Remove shortest match from the **start** |
| `${var##pattern}` | Remove longest match from the **start** |
| `${var%pattern}` | Remove shortest match from the **end** |
| `${var%%pattern}` | Remove longest match from the **end** |

This is extremely useful for pulling extensions or prefixes off filenames without calling
external tools.

## 5. Case conversion (Bash 4+)

```bash
#!/bin/bash
name="postgresql"
echo "${name^}"      # Postgresql   (capitalize first letter)
echo "${name^^}"       # POSTGRESQL   (uppercase all)

name="POSTGRESQL"
echo "${name,}"       # pOSTGRESQL   (lowercase first letter)
echo "${name,,}"        # postgresql   (lowercase all)
```

## 6. Splitting a string into parts

```bash
#!/bin/bash
csv_line="db1,10.0.1.10,5432"
IFS=',' read -r name ip port <<< "$csv_line"
echo "Name: $name, IP: $ip, Port: $port"
```

Or into an array:
```bash
#!/bin/bash
path="/var/lib/postgresql/16/main"
IFS='/' read -ra parts <<< "$path"
for part in "${parts[@]}"; do
    [ -n "$part" ] && echo "Part: $part"
done
```

## 7. Trimming whitespace

```bash
#!/bin/bash
str="   hello world   "
trimmed=$(echo "$str" | xargs)     # simple, common trick
echo "[$trimmed]"                    # [hello world]

# Pure-bash version (no external process)
trim() {
    local s="$1"
    s="${s#"${s%%[![:space:]]*}"}"
    s="${s%"${s##*[![:space:]]}"}"
    echo "$s"
}
echo "[$(trim "  padded  ")]"        # [padded]
```

## 8. Using `sed` and `awk` inside scripts

`sed` and `awk` are external tools, but they're used inside shell scripts constantly for
anything beyond simple substitution.

```bash
#!/bin/bash
# sed: in-place replace a setting in a config file
sed -i 's/^#listen_addresses.*/listen_addresses = '\''*'\''/' postgresql.conf

# sed: extract a value
current_max_conn=$(grep '^max_connections' postgresql.conf | sed 's/[^0-9]*//g')
echo "max_connections is currently: $current_max_conn"

# awk: extract a column from command output
df -h / | awk 'NR==2 {print $5}'    # print the "Use%" column from df

# awk: sum a column
du -sm /var/log/*.log | awk '{sum += $1} END {print "Total MB:", sum}'
```

## 9. Building strings dynamically

```bash
#!/bin/bash
db="appdb"
date_str=$(date +%F)
backup_file="${db}_backup_${date_str}.sql.gz"
echo "$backup_file"     # appdb_backup_2026-08-16.sql.gz
```

### Key takeaways

- Parameter expansion (`${var:pos:len}`, `${var/old/new}`, `${var#pattern}`, `${var%pattern}`)
  handles most string tasks without spawning an external process — faster and simpler.
- `sed` for in-place file substitution, `awk` for column-based extraction/aggregation — reach
  for these when parameter expansion isn't enough.
- Building dynamic filenames/strings from variables is the backbone of most automation
  scripts (backup filenames, log filenames, dated report names).

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. Given the line `work_mem = 4MB`, extract just `4MB` and then just the number `4`, using
   parameter expansion only (no `awk`/`sed`):
   ```bash
   line=$(grep "^work_mem" /u01/pgsql/18/data/postgresql.conf)
   value="${line#*= }"
   number="${value%MB}"
   echo "Value: $value, Number: $number"
   ```

2. Build a dynamic backup filename the same way Chapter 15 (Part 2) does, and print it:
   ```bash
   db="appdb"
   echo "${db}_backup_$(date +%F).dump"
   ```

3. Use `sed` to change `max_connections = 100` to `max_connections = 200` in a **copy** of
   `postgresql.conf` (never edit the real one directly in this exercise):
   ```bash
   cp /u01/pgsql/18/data/postgresql.conf /tmp/postgresql.conf.test
   sed -i 's/max_connections = 100/max_connections = 200/' /tmp/postgresql.conf.test
   grep max_connections /tmp/postgresql.conf.test
   ```

4. Use `awk` to print just the setting name and value (columns 1 and 3) from every
   uncommented line in the config file.

**Next:** [Chapter 10 — I/O Redirection & Pipes](10-io-redirection-and-pipes.md)
**Previous:** [Chapter 8 — Arrays](08-arrays.md)
