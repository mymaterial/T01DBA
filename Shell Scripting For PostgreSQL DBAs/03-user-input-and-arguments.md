# Chapter 3: User Input & Command-Line Arguments

## 1. Reading input interactively with `read`

```bash
#!/bin/bash
echo -n "Enter your name: "
read username
echo "Hello, $username!"

read -p "Enter your age: " age
echo "You are $age years old."

read -sp "Enter password: " password   # -s = silent (hides typed characters)
echo
echo "Password length: ${#password}"
```

| Option | Meaning |
|---|---|
| `-p "prompt"` | Show a prompt before reading |
| `-s` | Silent mode (for passwords) |
| `-a arr` | Read into an array |
| `-t N` | Timeout after N seconds |
| `-n N` | Read exactly N characters |

```bash
#!/bin/bash
read -t 5 -p "Quick! Type something (5s): " answer
if [ -z "$answer" ]; then
    echo "Too slow!"
else
    echo "You typed: $answer"
fi
```

## 2. Command-line arguments (`$1`, `$2`, ...)

```bash
#!/bin/bash
# usage: ./greet.sh Alice Engineer
echo "Script name: $0"
echo "First arg (name): $1"
echo "Second arg (role): $2"
echo "All args: $@"
echo "Number of args: $#"
```

```bash
$ ./greet.sh Alice Engineer
Script name: ./greet.sh
First arg (name): Alice
Second arg (role): Engineer
All args: Alice Engineer
Number of args: 2
```

## 3. `$@` vs `$*`

Both expand to all arguments, but they behave differently when quoted:

```bash
#!/bin/bash
for arg in "$@"; do
    echo "arg (\$@): [$arg]"
done
for arg in "$*"; do
    echo "arg (\$*): [$arg]"
done
```
With input `./script.sh "a b" c`:
- `"$@"` preserves each argument separately: `[a b]`, `[c]`
- `"$*"` joins everything into one string: `[a b c]`

**Rule of thumb:** use `"$@"` when looping over arguments individually — it's almost always
what you want.

## 4. Validating argument count

```bash
#!/bin/bash
if [ $# -lt 2 ]; then
    echo "Usage: $0 <database_name> <backup_dir>"
    exit 1
fi

db_name=$1
backup_dir=$2
echo "Backing up $db_name into $backup_dir"
```

## 5. Parsing named options with `getopts`

Positional arguments (`$1`, `$2`) are fine for simple scripts, but real tools take flags like
`-d dbname -h host`. `getopts` handles that cleanly:

```bash
#!/bin/bash
# usage: ./connect.sh -h myhost -p 5432 -d mydb

while getopts "h:p:d:" opt; do
    case $opt in
        h) host="$OPTARG" ;;
        p) port="$OPTARG" ;;
        d) dbname="$OPTARG" ;;
        \?) echo "Invalid option: -$OPTARG"; exit 1 ;;
    esac
done

echo "Host: $host, Port: $port, DB: $dbname"
```

```bash
$ ./connect.sh -h dbhost -p 5432 -d appdb
Host: dbhost, Port: 5432, DB: appdb
```

A colon after a letter in `"h:p:d:"` means that option **requires a value** (`-h dbhost`); no
colon means it's a flag (`-v` for verbose, with no argument).

## 6. Combining defaults, arguments, and interactive prompts

```bash
#!/bin/bash
# Accept a name as an argument, or prompt for it if not given
name="${1:-}"
if [ -z "$name" ]; then
    read -p "Enter a name: " name
fi
echo "Hello, $name!"
```

### Key takeaways

- `read` gets input interactively; `-p`, `-s`, `-t` cover the common cases.
- `$1`…`$9`, `$@`, `$#` handle simple positional arguments.
- `"$@"` (quoted) is the safe way to loop over arguments.
- `getopts` is the standard way to build scripts with real `-flag value` options.

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. Write `pg_conf_lookup.sh` that takes a PostgreSQL setting name as `$1` and greps for it:
   ```bash
   #!/bin/bash
   setting="$1"
   grep "$setting" /u01/pgsql/18/data/postgresql.conf
   ```
   Run it as `./pg_conf_lookup.sh work_mem` and `./pg_conf_lookup.sh shared_buffers`.

2. Add argument validation: if no argument is given, print a usage message and exit 1.

3. Rewrite it using `getopts` to accept `-v <version> -s <setting>`, so it can look up a
   setting for any PostgreSQL version, not just 18:
   ```bash
   ./pg_conf_lookup.sh -v 18 -s work_mem
   ```

4. Use `read -p` to prompt for the setting name interactively if it wasn't passed as an
   argument at all — combine `$1` with a `read` fallback, as shown in section 6 of this
   chapter.

**Next:** [Chapter 4 — Operators](04-operators.md)
**Previous:** [Chapter 2 — Variables & Data Types](02-variables-and-data-types.md)
