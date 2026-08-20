# Chapter 3: Linux File System

## 1. What is a File System?

A file system is the method the OS uses to organize, store, and manage files and directories
on storage devices. In Linux, **everything is treated as a file** — including devices,
directories, and sockets.

## 2. Characteristics

- Hierarchical structure (tree-like), starting from root (`/`)
- Case-sensitive (`Data` ≠ `data`)
- Supports multiple filesystem types (ext4, xfs, btrfs, etc.)
- Every file and directory has permissions (Chapter 6)

## 3. Important Directories

| Directory | Purpose |
|---|---|
| `/` | Root directory of the entire file system |
| `/bin` | Essential user command binaries |
| `/sbin` | System binaries (superuser commands) |
| `/etc` | Configuration files |
| `/home` | Home directories of users |
| `/root` | Home directory of the root user |
| `/var` | Variable files: logs, spool, cache |
| `/usr` | User programs and applications |
| `/tmp` | Temporary files |
| `/dev` | Device files |
| `/proc` | Virtual file system exposing process/kernel info |
| `/boot` | Boot loader files |
| `/lib` | Essential shared libraries |
| `/opt` | Optional/third-party software packages |
| `/mnt` | Mount point for temporary/manual filesystem mounts |
| `/media` | Mount point for removable media |

## 4. Linux File System Structure (diagram)

```
/
├── bin   ├── boot  ├── dev   ├── etc
├── home
│   ├── user1
│   ├── user2  → Desktop, Documents, Downloads
│   └── user3
├── lib   ├── media  ├── mnt → usb
├── opt   ├── proc   ├── root  ├── run
├── sbin  ├── srv    ├── sys   ├── tmp
├── usr   └── var
```

## 5. Mount Point

A directory where a filesystem from a storage device is "attached" (mounted), e.g.
`/mnt/usb`, `/media/pendrive`, or — critically for a DBA — a separate disk mounted at
`/var/lib/postgresql` or `/pgdata` for WAL/data isolation.

## 6. Types of File Systems

| Filesystem | Notes |
|---|---|
| ext4 | Most common default in Linux; solid, well-understood journaling |
| xfs | High performance, especially with large files and parallel I/O; RHEL default |
| btrfs | Advanced features: snapshots, compression, checksums |
| ntfs | Windows filesystem (readable/writable on Linux) |
| vfat | For FAT32/exFAT partitions (USB drives, etc.) |

## 7. Summary

Linux follows a single tree-like hierarchy starting from root (`/`). All files and
directories, including your PostgreSQL data directory and configuration, are organized under
it. Knowing this layout is the fastest way to find anything on a database host.

---

## Where a PostgreSQL DBA uses this

Knowing the FHS (Filesystem Hierarchy Standard) tells you exactly where to look on any
PostgreSQL server, even one you've never logged into before:

| What you need | Typical location (Debian/Ubuntu) | Typical location (RHEL/CentOS) |
|---|---|---|
| Data directory (`PGDATA`) | `/var/lib/postgresql/<version>/main` | `/var/lib/pgsql/<version>/data` |
| Config files | `/etc/postgresql/<version>/main/postgresql.conf` | inside `PGDATA` itself |
| `pg_hba.conf` (client auth) | `/etc/postgresql/<version>/main/pg_hba.conf` | inside `PGDATA` itself |
| Logs | `/var/log/postgresql/` | `/var/lib/pgsql/<version>/data/log/` or via journald |
| Binaries (`psql`, `pg_dump`) | `/usr/lib/postgresql/<version>/bin/` | `/usr/pgsql-<version>/bin/` |
| Unix socket | `/var/run/postgresql/` | `/tmp` or `/var/run/postgresql/` |
| WAL archive (if configured) | often a separate mount, e.g. `/mnt/wal_archive` | same idea |

- **Mount points matter for I/O isolation.** A common production pattern is to put `PGDATA`
  on one disk/mount and WAL (`pg_wal`) on a separate, faster disk/mount so write-ahead log
  I/O doesn't contend with table/index I/O. Checking `df -h` (Chapter 9) against `mount`
  output tells you whether that separation actually exists on a given host.

- **`/etc` is where you'll spend real time** — `postgresql.conf`, `pg_hba.conf`, and
  `pg_ident.conf` are configuration, so on Debian-family systems they live in `/etc` (not
  inside the data directory), unlike RHEL-family systems where they stay inside `PGDATA`
  unless moved. This single FHS convention explains a very common source of confusion when
  DBAs move between Ubuntu and RHEL shops.

- **`/var/log` is your first stop during an incident** — PostgreSQL logs, syslog, and
  `dmesg`-captured kernel messages (e.g. the OOM killer terminating a `postgres` process) all
  live under `/var/log`.

- **Filesystem type affects PostgreSQL performance tuning.** `xfs` is a very common choice for
  PostgreSQL data volumes because of its strong large-file and parallel I/O performance;
  `ext4` is the safe, well-tested default. Knowing which filesystem underlies your mount
  changes recommendations around `full_page_writes`, alignment, and I/O schedulers.

### Practical commands

```bash
find / -iname "postgresql.conf" 2>/dev/null   # locate config on an unfamiliar host
mount | grep postgres                          # see if PGDATA/WAL are on dedicated mounts
df -Th                                          # filesystem type + usage per mount
```

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

This is the exact drill used to introduce mount points and partitions in a live bootcamp,
using the disk usage command before PostgreSQL is even installed.

1. **See your partitions/mount points.**
   ```bash
   df -h
   ```
   Look for a dedicated mount such as `/u01` (a very common enterprise convention — Oracle
   and PostgreSQL shops both use `/u01` for application/database installs, separate from the
   OS's own `/` partition). Note its size and how much is used.

2. **Understand "everything starts with `/`."**
   ```bash
   cd /
   ls
   ```
   List every top-level directory. For each one — `/etc`, `/home`, `/var`, `/u01` (if
   present), `/proc` — say in one line what it's for, using the FHS table earlier in this
   chapter.

3. **Find where PostgreSQL will live, before it's installed.** If your lab already has
   PostgreSQL, locate its home:
   ```bash
   ls /u01 2>/dev/null
   ls /u01/pgsql 2>/dev/null
   ```
   A typical enterprise layout is `/u01/pgsql/18` (PostgreSQL version 18 installed under a
   dedicated `/u01` mount, separate from the root filesystem). If it exists, note the exact
   path — you'll use it constantly for the rest of this series.

4. **Confirm a mount point is really a separate filesystem.**
   ```bash
   mount | grep u01
   ```
   If `/u01` is its own mount, it means PostgreSQL's data and WAL can fill up *that* disk
   without taking down the OS on `/` — a real reason enterprises isolate the database onto
   its own partition.

**Next:** [Chapter 4 — Navigation Commands](04-linux-navigation-commands.md)
**Previous:** [Chapter 2 — Linux Architecture](02-linux-architecture.md)
