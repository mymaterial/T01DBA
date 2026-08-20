# Chapter 4: Operators

## 1. Arithmetic operators

Use `$(( ))` for arithmetic evaluation.

| Operator | Meaning | Example |
|---|---|---|
| `+` | Addition | `$((5+3))` → 8 |
| `-` | Subtraction | `$((10-2))` → 8 |
| `*` | Multiplication | `$((4*3))` → 12 |
| `/` | Division (integer) | `$((20/4))` → 5 |
| `%` | Modulus (remainder) | `$((20%3))` → 2 |
| `**` | Exponent | `$((2**8))` → 256 |
| `++` / `--` | Increment/decrement | `((count++))` |

```bash
#!/bin/bash
a=10
b=3
echo "Sum: $((a+b))"
echo "Diff: $((a-b))"
echo "Product: $((a*b))"
echo "Quotient: $((a/b))"
echo "Remainder: $((a%b))"

# Floating point needs bc or awk — bash arithmetic is integer-only
echo "Float division: $(echo "scale=2; $a/$b" | bc)"
```

Alternative: `let` and `expr`:
```bash
let result=5+3
echo "$result"          # 8

result=$(expr 5 + 3)     # note: spaces required around operators with expr
echo "$result"            # 8
```

## 2. Comparison (numeric) operators

Used inside `[ ]` or `[[ ]]` test expressions.

| Operator | Meaning |
|---|---|
| `-eq` | Equal |
| `-ne` | Not equal |
| `-gt` | Greater than |
| `-lt` | Less than |
| `-ge` | Greater or equal |
| `-le` | Less or equal |

```bash
#!/bin/bash
num=15
if [ $num -gt 10 ]; then
    echo "$num is greater than 10"
fi
```

## 3. String operators

| Operator | Meaning |
|---|---|
| `=` or `==` | Strings are equal |
| `!=` | Strings are not equal |
| `-z str` | String is empty |
| `-n str` | String is not empty |
| `<` / `>` | Lexicographic comparison (inside `[[ ]]`) |

```bash
#!/bin/bash
str1="postgres"
str2="mysql"

if [ "$str1" = "$str2" ]; then
    echo "Equal"
else
    echo "Not equal"
fi

if [ -z "$empty_var" ]; then
    echo "empty_var is empty"
fi
```

## 4. Logical operators

| Operator | Meaning |
|---|---|
| `&&` | AND |
| `\|\|` | OR |
| `!` | NOT |

```bash
#!/bin/bash
age=25
citizen=true

if [ $age -ge 18 ] && [ "$citizen" = true ]; then
    echo "Eligible to vote"
fi

if [ $age -lt 13 ] || [ $age -gt 65 ]; then
    echo "Discount applies"
else
    echo "Standard price"
fi

is_ready=false
if ! $is_ready; then
    echo "Not ready yet"
fi
```

Logical operators also chain **commands** directly (not just conditions), which is an
extremely common Bash idiom:

```bash
mkdir -p /tmp/backup && echo "Directory ready" || echo "Failed to create directory"
```
`cmd1 && cmd2` runs `cmd2` only if `cmd1` succeeded (exit code 0).
`cmd1 || cmd2` runs `cmd2` only if `cmd1` failed (non-zero exit code).

## 5. `[ ]` vs `[[ ]]`

`[[ ]]` is a Bash extension with fewer quoting pitfalls and extra features (pattern matching,
`&&`/`||` inside the brackets, no word-splitting surprises). Prefer `[[ ]]` in Bash-only
scripts; use `[ ]` (or `test`) if the script must stay POSIX-`sh`-compatible.

```bash
#!/bin/bash
file="backup.sql"
if [[ $file == *.sql ]]; then
    echo "This is a SQL file"
fi
```

### Key takeaways

- `$(( ))` for integer arithmetic; `bc` for floating point.
- `-eq/-ne/-gt/-lt/-ge/-le` for numbers, `=`/`!=`/`-z`/`-n` for strings.
- `&&`/`||` chain both conditions and commands.
- Prefer `[[ ]]` over `[ ]` in Bash scripts unless POSIX portability is required.

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. Extract the numeric value of `work_mem` (assume it's in MB, e.g. `4MB`) and compute how
   much memory 50 concurrent connections could use for sorts, in the worst case:
   ```bash
   work_mem_mb=4
   connections=50
   echo "Worst-case sort memory: $((work_mem_mb * connections)) MB"
   ```

2. Write a comparison: if available RAM (from `free -m`, first row, second column) is less
   than that worst-case number, print a warning.

3. Use a string operator to check whether `postgresql.conf` contains any line starting with
   `listen_addresses`:
   ```bash
   line=$(grep listen_addresses /u01/pgsql/18/data/postgresql.conf)
   if [ -n "$line" ]; then
       echo "Found: $line"
   fi
   ```

4. Chain a real check with `&&`/`||`:
   ```bash
   grep -q shared_buffers /u01/pgsql/18/data/postgresql.conf && echo "Found" || echo "Missing"
   ```

**Next:** [Chapter 5 — Conditionals](05-conditionals.md)
**Previous:** [Chapter 3 — User Input & Arguments](03-user-input-and-arguments.md)
