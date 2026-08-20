# Chapter 1: Getting Started with Shell Scripting

## 1. What is a shell script?

A shell script is a plain text file containing a sequence of commands that the shell (Bash,
in this series) executes in order, exactly as if you'd typed them one by one at the
terminal. Scripting turns repetitive manual work into a single, repeatable command.

## 2. The shebang line

The first line of a script tells the OS which interpreter should run it:

```bash
#!/bin/bash
```

This is called a **shebang** (`#!`). Without it, the script runs using whatever shell is
currently active, which can behave differently. Always include it as line 1.

Other common shebangs you'll see:
```bash
#!/bin/sh          # POSIX shell — fewer features, more portable
#!/usr/bin/env bash # finds bash wherever it is in $PATH — more portable across systems
```

## 3. Writing and running your first script

```bash
#!/bin/bash
# hello.sh — my first shell script
echo "Hello, World!"
echo "Today is $(date)"
```

Three ways to run it:

```bash
chmod +x hello.sh   # make it executable (one-time)
./hello.sh           # run it directly

bash hello.sh         # run it by explicitly invoking bash (no chmod needed)

sh hello.sh            # run with the sh interpreter
```

Sample output:
```
Hello, World!
Today is Sun Aug 16 09:03:11 UTC 2026
```

## 4. Comments

```bash
#!/bin/bash
# This is a single-line comment — everything after # on the line is ignored

echo "This runs"   # inline comment works too

: <<'END_COMMENT'
This is a multi-line comment block.
Nothing between the two markers is executed.
END_COMMENT

echo "This also runs"
```

Good scripts are commented like documentation: what the script does, what arguments it
expects, and why any non-obvious line exists.

## 5. Script file conventions

- Extension `.sh` is a strong convention (not required by Bash itself, but expected by
  humans and tools).
- File permissions: `chmod +x script.sh` gives execute permission (see Chapter 6 of the
  Linux Notes series for the full permissions model).
- Store operational scripts somewhere consistent, e.g. `/usr/local/bin/` or
  `/opt/scripts/`, so they're on `$PATH` and easy to find later.

## 6. A slightly larger first script

```bash
#!/bin/bash
# system_info.sh — print a short system summary

echo "=== System Info ==="
echo "Hostname : $(hostname)"
echo "Uptime   : $(uptime -p)"
echo "User     : $(whoami)"
echo "Date     : $(date '+%Y-%m-%d %H:%M:%S')"
echo "===================="
```

Run it:
```bash
chmod +x system_info.sh
./system_info.sh
```

Output:
```
=== System Info ===
Hostname : db-primary-01
Uptime   : up 14 days
User     : postgres
Date     : 2026-08-16 09:05:02
====================
```

### Key takeaways

- Always start scripts with `#!/bin/bash`.
- `chmod +x` before you can run a script with `./script.sh`.
- Comments (`#`) make scripts maintainable — use them generously.
- `$(command)` runs a command and substitutes its output inline — you'll use this constantly.

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. Write `pg_greeting.sh` — a script with a proper shebang that prints your hostname, the
   current date, and the fixed string `"PostgreSQL config lives at /u01/pgsql/18/data"`.
   `chmod +x` it and run it with `./pg_greeting.sh`.

2. Modify the script so it also runs `cat /u01/pgsql/18/data/postgresql.conf | wc -l` and
   prints "Config file has N lines" using `$(...)` command substitution.

3. Run your script three different ways — `./pg_greeting.sh`, `bash pg_greeting.sh`,
   `sh pg_greeting.sh` — and note whether the output changes.

**Next:** [Chapter 2 — Variables & Data Types](02-variables-and-data-types.md)
