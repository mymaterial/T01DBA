# Chapter 6: Linux File Permissions

## 1. What are File Permissions?

File permissions control **who** can read, write, or execute a file or directory. This is
the mechanism the Linux kernel uses to enforce access control across a multiuser system —
and it's the single most common source of PostgreSQL startup failures on a fresh install.

## 2. Permission Types

| Symbol | Meaning | Description |
|---|---|---|
| `r` | Read (4) | Allows reading the file, or listing the directory |
| `w` | Write (2) | Allows modifying the file or directory |
| `x` | Execute (1) | Allows executing the file, or entering/accessing the directory |

## 3. Permission Structure

Every file has **three sets** of permissions: Owner, Group, Others.

```
-  rwx  r-x  r--
   owner group others
    7     5    4
```

```
r = 4
w = 2
x = 1
Total permission = r + w + x
```

## 4. Numeric (Octal) Representation

| Permission | Binary | Octal | Meaning |
|---|---|---|---|
| `r--` | 100 | 4 | Read |
| `-w-` | 010 | 2 | Write |
| `--x` | 001 | 1 | Execute |
| `rw-` | 110 | 6 | Read + Write |
| `r-x` | 101 | 5 | Read + Execute |
| `-wx` | 011 | 3 | Write + Execute |
| `rwx` | 111 | 7 | Read + Write + Execute |
| `---` | 000 | 0 | No permission |

## 5. Reading `ls -l` Output

```
-rw-r--r--  1  user  group  1024  May 25 10:30  file.txt
 ↓          ↓  ↓     ↓      ↓     ↓              ↓
 perms      links owner group size date          filename
```

Breakdown of the permission string:
- 1st char: file type (`-` = regular file, `d` = directory)
- Next 3: owner (`rw-`)
- Next 3: group (`r--`)
- Last 3: others (`r--`)

## 6. Changing Permissions

```bash
chmod [options] mode filename
```

| Example | Result |
|---|---|
| `chmod 755 file.sh` | `rwxr-xr-x` |
| `chmod 644 file.txt` | `rw-r--r--` |
| `chmod 700 script.sh` | `rwx------` |
| `chmod -R 755 myfolder/` | Recursive change |

**Tip:** directories need the `x` permission to be entered and listed. Files need `r` to
read, `w` to modify, `x` to execute.

## 7. Special Permissions

| Permission | Symbol | Octal | Meaning |
|---|---|---|---|
| Setuid | `u+s` | 4 | Run as the file's owner |
| Setgid | `g+s` | 2 | Run as the file's group |
| Sticky bit | `o+t` | 1 | Restrict deletion (commonly used on `/tmp`) |

### Quick example

If permission is `rwxr-xr--`:
- Owner (`rwx`) = 4+2+1 = 7
- Group (`r-x`) = 4+0+1 = 5
- Others (`r--`) = 4+0+0 = 4
- Numeric value = **754**

### Key points

- Use `chmod` to change permissions.
- Use `chown` to change the owner.
- Use `chgrp` to change the group.

---

## Where a PostgreSQL DBA uses this

This chapter maps almost 1:1 onto real PostgreSQL error messages.

### The classic startup failure

PostgreSQL **refuses to start** if `PGDATA` permissions are too open:

```
FATAL: data directory "/var/lib/postgresql/15/main" has group or world access
DETAIL: Permissions should be u=rwx (0700).
```

Fix:
```bash
chmod 700 /var/lib/postgresql/15/main
chown -R postgres:postgres /var/lib/postgresql/15/main
```
PostgreSQL enforces `0700` on `PGDATA` on purpose — the directory contains raw table/index
data and WAL, and if group/other could read it, any user on the box could read your data
files directly on disk, bypassing all of PostgreSQL's own authentication.

### `pg_hba.conf` and `postgresql.conf`

These should be owned by the `postgres` user and not world-writable:
```bash
chown postgres:postgres /etc/postgresql/15/main/pg_hba.conf
chmod 640 /etc/postgresql/15/main/pg_hba.conf
```

### Unix socket directory

If clients connect via Unix socket and get `could not connect to server: Permission denied`,
check the socket directory permissions (commonly `/var/run/postgresql`), and confirm the
connecting OS user has execute (`x`) permission to traverse into it.

### SSL private key

PostgreSQL requires the SSL private key file to be tightly restricted, and will refuse to
start otherwise:
```bash
chmod 600 server.key
chown postgres:postgres server.key
```

### Backup and script permissions

```bash
chmod 700 /usr/local/bin/pg_backup.sh   # only the owner (e.g. postgres) can read/run it
chown postgres:postgres /usr/local/bin/pg_backup.sh
```
Backup scripts often contain credentials or connection strings, so `700` (owner-only) is the
standard, not `755`.

### Quick diagnostic sequence for "permission denied" issues

```bash
ls -ld $PGDATA                       # check PGDATA itself: must be drwx------ postgres postgres
ls -l $PGDATA/postgresql.conf        # config files: postgres-owned, not world-writable
stat $PGDATA                          # detailed owner/permission/timestamp info
```

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. **Check PGDATA's permissions on a real (or simulated) instance.**
   ```bash
   ls -ld /u01/pgsql/18/data          # on a real lab instance
   ```
   or, if you don't have PostgreSQL installed yet, simulate it:
   ```bash
   mkdir -p ~/pg_practice/data
   chmod 700 ~/pg_practice/data
   ls -ld ~/pg_practice/data
   ```
   Confirm the permission string reads `drwx------` — this is the exact `0700` PostgreSQL
   enforces on its real data directory.

2. **Reproduce the classic startup error, on purpose.**
   ```bash
   chmod 750 ~/pg_practice/data
   ls -ld ~/pg_practice/data
   ```
   This now shows `drwxr-x---` — group-readable. If this were a real `PGDATA`, PostgreSQL
   would refuse to start with `FATAL: data directory has group or world access`. Fix it:
   ```bash
   chmod 700 ~/pg_practice/data
   ```

3. **Permission math practice.** Without running a command, compute the octal value for:
   - `rwxr-x---` → ?
   - `rw-r-----` → ?
   - `rwx------` → ?
   Then verify with:
   ```bash
   chmod 750 somefile; ls -l somefile
   chmod 640 somefile; ls -l somefile
   chmod 700 somefile; ls -l somefile
   ```

4. **Owner check.** If you have a real `postgres` OS user on your lab:
   ```bash
   ls -l /u01/pgsql/18/data/postgresql.conf
   ```
   Confirm the owner and group are both `postgres`, not `root`. If you installed PostgreSQL
   as `root`, this is exactly the kind of mismatch that later causes "permission denied"
   errors when the `postgres` service user tries to start the server.

**Next:** [Chapter 7 — Searching & Text Processing](07-linux-searching-text-processing.md)
**Previous:** [Chapter 5 — File & Directory Commands](05-linux-file-directory-commands.md)
