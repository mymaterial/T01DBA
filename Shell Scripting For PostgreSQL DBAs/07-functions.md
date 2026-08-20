# Chapter 7: Functions

## 1. Defining and calling a function

```bash
#!/bin/bash
greet() {
    echo "Hello, $1!"
}

greet "Linux User"
greet "Developer"
```

Two equivalent syntaxes:
```bash
function greet {
    echo "Hello!"
}

greet() {
    echo "Hello!"
}
```
The second form (`name()`) is more portable; prefer it.

## 2. Function arguments

Inside a function, `$1`, `$2`, `$@`, `$#` refer to the arguments passed **to the function**,
not to the script.

```bash
#!/bin/bash
show_info() {
    echo "Function name args: $@"
    echo "Number of args: $#"
    echo "First: $1, Second: $2"
}

show_info apple banana cherry
```

## 3. Return values

Bash functions don't return arbitrary values like other languages — they return an **exit
status** (0–255) via `return`, or you print/echo a value and capture it with `$(...)`.

**Exit status pattern (for success/failure):**
```bash
#!/bin/bash
is_even() {
    if (( $1 % 2 == 0 )); then
        return 0   # success = "true"
    else
        return 1   # failure = "false"
    fi
}

if is_even 4; then
    echo "4 is even"
fi
```

**Output-capture pattern (for actual data):**
```bash
#!/bin/bash
add() {
    echo $(( $1 + $2 ))
}

result=$(add 5 3)
echo "Sum: $result"
```

## 4. Local variables and scope

Without `local`, variables inside a function are **global** by default — a common source of
bugs. Use `local` to keep function-internal state from leaking.

```bash
#!/bin/bash
counter=0

increment() {
    local step=$1     # local: only visible inside this function
    counter=$((counter + step))   # this modifies the global counter on purpose
}

increment 5
increment 3
echo "Counter: $counter"   # 8
```

## 5. Recursion

```bash
#!/bin/bash
factorial() {
    local n=$1
    if [ "$n" -le 1 ]; then
        echo 1
    else
        local sub=$(factorial $((n - 1)))
        echo $((n * sub))
    fi
}

echo "5! = $(factorial 5)"   # 120
```

## 6. Functions calling other functions (building a small library)

```bash
#!/bin/bash

log() {
    echo "[$(date '+%F %T')] $1"
}

check_disk() {
    local usage=$(df -h / | awk 'NR==2 {print $5}' | tr -d '%')
    if [ "$usage" -gt 80 ]; then
        log "WARNING: disk usage is ${usage}%"
        return 1
    else
        log "Disk usage OK: ${usage}%"
        return 0
    fi
}

main() {
    log "Starting health check"
    check_disk
    log "Health check complete"
}

main
```
This is the standard shape of a real operational script: small, single-purpose functions,
composed together in a `main` function called at the bottom.

## 7. Default parameter values inside functions

```bash
#!/bin/bash
greet() {
    local name="${1:-World}"
    echo "Hello, $name!"
}

greet          # Hello, World!
greet "Alice"  # Hello, Alice!
```

### Key takeaways

- Define with `name() { ... }`; call by name; arguments arrive as `$1`, `$2`, `$@`, `$#`.
- Use `return` for a status code (0 = success), and `echo` + `$(func)` to "return" real data.
- Always use `local` for variables that shouldn't leak into the global scope.
- Structuring a script as small functions + one `main` at the bottom is the pattern used in
  almost every production shell script.

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. Write a function `pg_setting()` that takes a setting name as an argument and echoes its
   value (not the whole line) from `postgresql.conf`, using `cut` or `awk` internally:
   ```bash
   pg_setting() {
       grep "^$1" /u01/pgsql/18/data/postgresql.conf | awk -F'=' '{print $2}' | xargs
   }
   echo "work_mem is: $(pg_setting work_mem)"
   ```

2. Write a function `pg_is_running()` that returns 0 (true) if `pg_isready` succeeds, 1
   otherwise — then use it in an `if pg_is_running; then ... fi` block.

3. Build a tiny function library file `pg_functions.sh` containing `pg_setting()` and
   `pg_is_running()`, then `source pg_functions.sh` from a second script and call both
   functions from there.

**Next:** [Chapter 8 — Arrays](08-arrays.md)
**Previous:** [Chapter 6 — Loops](06-loops.md)
