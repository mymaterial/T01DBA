# Chapter 5: Linux File and Directory Commands

In Linux, everything is treated as a file. Files are the basic data units, and directories
organize files in a tree-like structure.

## 1. File Commands

| Command | Purpose | Example | Notes |
|---|---|---|---|
| `touch` | Create an empty file | `touch demo.txt` | Creates the file if it doesn't exist |
| `cat` | Display file contents | `cat demo.txt` | Dumps whole content to terminal |
| `less` | View file content page by page | `less demo.txt` | Press `q` to quit |
| `head` | Show first lines of a file | `head -n 10 demo.txt` | Default: first 10 lines |
| `tail` | Show last lines of a file | `tail -n 10 demo.txt` | Default: last 10 lines |
| `cp` | Copy a file | `cp a.txt b.txt` | `b.txt` becomes a copy of `a.txt` |
| `mv` | Move or rename a file | `mv a.txt new.txt` | Renames `a.txt` to `new.txt` |
| `rm` | Remove (delete) a file | `rm a.txt` | Permanently deletes — no recycle bin |
| `ln` | Create a link to a file | `ln a.txt link.txt` | `link.txt` is a hard link to `a.txt` |

## 2. Directory Commands

| Command | Purpose | Example | Notes |
|---|---|---|---|
| `pwd` | Print working directory | `pwd` | Shows current path |
| `ls` | List files and directories | `ls` | Basic listing |
| `ls -l` | List in long format | `ls -l` | Shows permissions, size, owner, etc. |
| `ls -a` | List all (including hidden) | `ls -a` | Hidden files start with `.` |
| `mkdir` | Create a new directory | `mkdir mydir` | |
| `cd` | Change directory | `cd mydir` | Go into `mydir` |
| `cd ..` | Go one level up | `cd ..` | Back to parent |
| `rmdir` | Remove empty directory | `rmdir emptydir` | Directory must be empty |
| `rm -r` | Remove directory recursively | `rm -r mydir` | Deletes directory **and all contents** |
| `tree` | Show directory tree structure | `tree` | May need installing |

## Example Directory Structure

```
/ (root)
├── home
│   ├── user1
│   └── user2
├── etc
├── var
│   ├── log
│   └── cache
├── usr
└── tmp
```

## Key Points

- Files store data.
- Directories help organize files.
- Everything in Linux starts from root (`/`).
- Permissions control access to files and directories (Chapter 6).

### Tips

- Use **Tab** for auto-completion.
- Use `ls -lh` for human-readable sizes.
- Be careful with `rm -r` — it removes everything inside, with no undo.

---

## Where a PostgreSQL DBA uses this

These are the exact commands used to inspect and manage PostgreSQL configuration, logs, and
backups on a daily basis.

```bash
# Reading configuration
cat /etc/postgresql/15/main/postgresql.conf | grep -v '^#' | grep -v '^$'
less /etc/postgresql/15/main/pg_hba.conf

# Watching logs live (tail -f is one of the most-used commands in this whole series)
tail -f /var/log/postgresql/postgresql-15-main.log
tail -n 200 /var/log/postgresql/postgresql-15-main.log | grep ERROR

# Checking the first lines of a large CSV export before loading with COPY
head -n 5 export.csv

# Safely editing configuration: always copy before you touch it
cp postgresql.conf postgresql.conf.bak.$(date +%F)

# Renaming a rotated log or a completed backup file
mv base_backup.tar base_backup_2026-08-16.tar

# Removing old WAL archives that are safely past your retention window (be very careful!)
rm /mnt/wal_archive/000000010000000000000012

# Creating a symlink so a script always points at "the current" backup without renaming files
ln -s /backups/2026-08-16 /backups/latest
```

- **`tail -f`** on the PostgreSQL log is arguably the single most-used Linux command in a
  DBA's daily workflow — it's how you watch startup, checkpoints, replication status changes,
  and errors happen in real time.
- **`cat` + `grep -v '^#'`** is the fastest way to see the *effective* (non-comment,
  non-blank) settings in a huge `postgresql.conf` file without scrolling past hundreds of
  commented-out defaults.
- **`cp` before you edit** is a non-negotiable habit for `postgresql.conf` and `pg_hba.conf` —
  a single bad edit to `pg_hba.conf` can lock out every client, including you.
- **`rm -r` is dangerous around `PGDATA`.** Deleting the wrong directory (or the right
  directory on the wrong server) is one of the most common causes of real outages. Many teams
  alias `rm` to `rm -i` (interactive confirm) on production database hosts for exactly this
  reason.
- **`ln -s`** (symbolic links) are commonly used to point a stable path (e.g. `/backups/latest`)
  at the most recent dated backup directory, so restore scripts don't need to know the exact
  date.

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. **Read a real install log.** If your lab has a PostgreSQL install log (a common one to
   create yourself during installation is `install.log`), practice:
   ```bash
   cd ~
   echo "Successfully installed PostgreSQL 18" > install.log
   cat install.log
   ```
   This is literally the first command demonstrated in the bootcamp: `cat` a file to prove
   something happened (install succeeded).

2. **Open a system file two ways.**
   ```bash
   cd /etc
   cat os-release          # relative — works because you're standing in /etc
   cd
   cat /etc/os-release       # absolute — works from home too
   ```

3. **Build the PostgreSQL config folder path from scratch.**
   ```bash
   mkdir -p ~/pg_practice/18/data
   touch ~/pg_practice/18/data/postgresql.conf
   echo "work_mem = 4MB" >> ~/pg_practice/18/data/postgresql.conf
   echo "shared_buffers = 128MB" >> ~/pg_practice/18/data/postgresql.conf
   cat ~/pg_practice/18/data/postgresql.conf
   ```
   You've just recreated, in miniature, the exact file you'll inspect for real once
   PostgreSQL is installed at `/u01/pgsql/18/data/postgresql.conf`.

4. **Practice head/tail on a config file.**
   ```bash
   head -n 1 ~/pg_practice/18/data/postgresql.conf
   tail -n 1 ~/pg_practice/18/data/postgresql.conf
   ```

5. **Back up before you touch anything (a lifelong DBA habit, starting now).**
   ```bash
   cp ~/pg_practice/18/data/postgresql.conf ~/pg_practice/18/data/postgresql.conf.bak
   ls -l ~/pg_practice/18/data/
   ```

6. **Clean up.**
   ```bash
   rm -r ~/pg_practice
   ```

**Next:** [Chapter 6 — File Permissions](06-linux-file-permissions.md)
**Previous:** [Chapter 4 — Navigation Commands](04-linux-navigation-commands.md)
