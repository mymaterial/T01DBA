# Chapter 8: Arrays

## 1. Indexed arrays — creating and accessing

```bash
#!/bin/bash
fruits=("apple" "banana" "orange")

echo "${fruits[0]}"      # apple
echo "${fruits[1]}"      # banana
echo "${fruits[@]}"       # apple banana orange (all elements)
echo "${#fruits[@]}"      # 3 (number of elements)
```

## 2. Adding, updating, and removing elements

```bash
#!/bin/bash
fruits=("apple" "banana")

fruits+=("cherry")          # append
fruits[1]="blueberry"        # update index 1
unset fruits[0]               # remove index 0

echo "${fruits[@]}"           # blueberry cherry
```

## 3. Looping over an array

```bash
#!/bin/bash
servers=("db1" "db2" "db3")

for server in "${servers[@]}"; do
    echo "Checking $server..."
done

# with index
for i in "${!servers[@]}"; do
    echo "Index $i: ${servers[$i]}"
done
```

## 4. Building an array from command output

```bash
#!/bin/bash
# One element per line, safely (handles spaces correctly)
mapfile -t log_files < <(find /var/log -name "*.log")

for f in "${log_files[@]}"; do
    echo "Log: $f"
done
```
`mapfile -t` (a.k.a. `readarray -t`) is the safe way to turn command output into an array,
one line = one element, with trailing newlines stripped.

## 5. Associative arrays (key-value maps)

Requires `declare -A` (Bash 4+).

```bash
#!/bin/bash
declare -A server_roles
server_roles[db1]="primary"
server_roles[db2]="replica"
server_roles[db3]="replica"

echo "${server_roles[db1]}"        # primary

for server in "${!server_roles[@]}"; do
    echo "$server -> ${server_roles[$server]}"
done
```

## 6. Checking if a key/index exists

```bash
#!/bin/bash
declare -A config
config[timeout]=30

if [[ -v config[timeout] ]]; then
    echo "timeout is set to ${config[timeout]}"
fi

if [[ -v config[retries] ]]; then
    echo "retries is set"
else
    echo "retries is not set"
fi
```

## 7. A practical example: iterating over multiple hosts

```bash
#!/bin/bash
declare -A hosts=(
    [web1]="10.0.1.10"
    [web2]="10.0.1.11"
    [db1]="10.0.2.10"
)

for name in "${!hosts[@]}"; do
    ip="${hosts[$name]}"
    if ping -c 1 -W 1 "$ip" &> /dev/null; then
        echo "$name ($ip): UP"
    else
        echo "$name ($ip): DOWN"
    fi
done
```

### Key takeaways

- Indexed arrays: `arr=(a b c)`, access with `${arr[i]}`, all elements with `${arr[@]}`.
- Associative arrays (`declare -A`) map names to values — great for host lists, config maps.
- Always quote array expansions: `"${arr[@]}"`, to keep elements with spaces intact.
- `mapfile -t` is the safe way to load command output or file lines into an array.

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. Build an indexed array of settings you care about and loop `grep` over each:
   ```bash
   settings=(work_mem shared_buffers max_connections wal_level effective_cache_size)
   for s in "${settings[@]}"; do
       grep "^$s\|^#$s" /u01/pgsql/18/data/postgresql.conf
   done
   ```

2. Build an associative array mapping PostgreSQL version → data directory, then loop over it
   to check which versions are actually installed on your lab machine:
   ```bash
   declare -A pg_versions=(
       [16]="/u01/pgsql/16/data"
       [17]="/u01/pgsql/17/data"
       [18]="/u01/pgsql/18/data"
   )
   for ver in "${!pg_versions[@]}"; do
       [ -d "${pg_versions[$ver]}" ] && echo "PG $ver: installed" || echo "PG $ver: not found"
   done
   ```

3. Use `mapfile -t` to load every line of `postgresql.conf` matching `max_` into an array,
   then print how many such settings exist.

**Next:** [Chapter 9 — String Manipulation](09-string-manipulation.md)
**Previous:** [Chapter 7 — Functions](07-functions.md)
