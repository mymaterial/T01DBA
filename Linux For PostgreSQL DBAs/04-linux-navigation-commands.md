# Chapter 4: Linux Navigation Commands

Navigation commands are used to move around the Linux file system and explore directories.

## 1. Basic Navigation Commands

| Command | Purpose | Example |
|---|---|---|
| `pwd` | Print working directory | `pwd` |
| `ls` | List files and directories | `ls`, `ls -l` |
| `cd <dir>` | Change directory | `cd Documents` |
| `cd ..` | Move one directory up | `cd ..` |
| `cd ~` | Go to home directory | `cd ~` |
| `cd -` | Go to previous directory | `cd -` |
| `clear` | Clear the terminal screen | `clear` |
| `exit` | Exit from terminal | `exit` |

## 2. Explanation with Examples

- **`pwd`** — shows your current location.
  ```bash
  $ pwd
  /home/user
  ```
- **`cd /path/to/dir`** — change to a specific directory.
  ```bash
  cd /etc
  ```
- **`cd ..`** — move back to the previous (parent) directory. If you're in
  `/home/user/docs`, `cd ..` takes you to `/home/user`.
- **`cd ~`** — directly go to your home directory, e.g. `/home/user`.
- **`cd -`** — switch to the directory you were in *just before* the current one. If you were
  in `/etc` and then went to `/var`, `cd -` takes you back to `/etc`.

## 3. Other Useful Commands

- `ls -l` → list files in long format (permissions, owner, size, date)
- `ls -a` → list all files, including hidden files (dotfiles)
- `ls -lh` → list with human-readable sizes (KB/MB/GB instead of raw bytes)
- `tree` → show directory structure as a tree view (may need installing)

## 4. Navigation Examples (practical flow)

```
Open Terminal → pwd → ls -l → cd Documents → cd .. → cd ~
```

1. Open terminal
2. `pwd` — check current directory
3. `ls -l` — list files in current directory
4. `cd Documents` — go into a directory
5. `cd ..` — go back to previous directory
6. `cd ~` — go to home directory

### Tips

- Linux is case-sensitive (`Home` ≠ `home`).
- Use the **Tab** key for auto-completion of paths and filenames.
- Use `ls` often to orient yourself before running destructive commands.

---

## Where a PostgreSQL DBA uses this

Navigation is the very first thing you do on any incident call, and it's almost always the
same handful of moves:

```bash
sudo su - postgres                 # switch to the postgres OS user
cd $PGDATA                         # or: cd /var/lib/postgresql/15/main
pwd                                 # confirm you're actually in the data directory
ls -la                              # spot postmaster.pid, base/, pg_wal/, config files
cd pg_wal && ls -la                 # check WAL segment count/growth
cd -                                # jump back to $PGDATA instantly
```

- **`cd ~postgres`** or `cd $PGDATA` is muscle memory for any PostgreSQL DBA — it's the single
  most-visited directory on a database host.
- **`cd -`** is genuinely useful when you're bouncing between the data directory and the log
  directory (`/var/log/postgresql`) while diagnosing an issue — no need to retype long paths.
- **`ls -la`** inside `PGDATA` immediately tells you a lot: is `postmaster.pid` present (is
  the instance running)? Is `recovery.signal` or `standby.signal` present (is this a
  replica)? Are permissions on `pg_hba.conf` sane?
- **Tab-completion** matters more here than almost anywhere else — PostgreSQL paths are long
  and version-numbered (`/usr/lib/postgresql/15/bin/pg_ctl`), and mistyping them under
  pressure during an incident is a real risk.

### A DBA's typical orientation sequence on a new host

```bash
pwd
whoami
cd /etc/postgresql 2>/dev/null || cd /var/lib/pgsql   # find config, whichever family
ls -la
```

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

This is the single most important drill in the whole series — the instructor spent nearly
half a live session on exactly this, because absolute vs relative paths trip up almost every
new Linux user. Do it exactly as written, don't skip steps.

### Part A — Build a nested folder maze

```bash
cd ~
mkdir -p folder1/subfolder
mkdir -p folder2/subfolder/folder22/folder33
echo "hello from folder2" > folder2/subfolder/folder2file.txt
```

You now have:
```
~/folder1/subfolder
~/folder2/subfolder/folder22/folder33
~/folder2/subfolder/folder2file.txt
```

### Part B — Absolute path drill

1. From your home directory, open `folder2file.txt` using its **absolute path** (starting
   with `/`, or `~/` from home) in one single command, without `cd`-ing anywhere first:
   ```bash
   cat ~/folder2/subfolder/folder2file.txt
   ```
2. Now `cd` into `folder1/subfolder` (a completely different branch):
   ```bash
   cd ~/folder1/subfolder
   pwd
   ```
3. From *here*, open the same file again using the exact same absolute path command from
   step 1. **It should work identically**, because an absolute path always starts from `/`
   (or `~`) regardless of where you currently are.

### Part C — Relative path drill

1. Still inside `~/folder1/subfolder`, try to open the file using only its name:
   ```bash
   cat folder2file.txt
   ```
   This **fails** — "No such file or directory" — because there's no such file *relative to
   where you are right now*.
2. `cd` to `~/folder2/subfolder` (the actual parent of the file):
   ```bash
   cd ~/folder2/subfolder
   cat folder2file.txt
   ```
   Now it works — you're using a **relative path**, and it only works because you're
   standing in the right starting location.

### Part D — Navigation shortcuts

```bash
pwd                 # confirm exactly where you are before doing anything risky
cd ..                # go up one level to ~/folder2
pwd
cd -                  # jump back to where you just were
pwd
cd                      # no argument — takes you straight home
pwd
cd folder2/subfolder/folder22   # relative path, multiple levels at once
pwd
cd ~/folder1                     # absolute path (using ~ shorthand), from anywhere
pwd
```

### Part E — The rule, in your own words

Answer without looking back at the chapter: *"Relative paths only work if you're already
inside the parent folder (or a subfolder of it). Absolute paths work from anywhere, because
they start at `/`."* Rewrite that rule in your own words — you should be able to explain it
to someone else.

### Part F — Tie it to PostgreSQL

```bash
cat /u01/pgsql/18/data/postgresql.conf   # absolute path — works from anywhere
cd /u01/pgsql/18/data
cat postgresql.conf                        # relative path — works only because you cd'd there first
cd
```
Both commands open the same file — this is the exact pattern you'll use for the rest of your
DBA career: sometimes you `cd` into `PGDATA` and work relatively; sometimes you reach into it
with one absolute-path command from wherever you happen to be.

**Next:** [Chapter 5 — File & Directory Commands](05-linux-file-directory-commands.md)
**Previous:** [Chapter 3 — Linux File System](03-linux-file-system.md)
