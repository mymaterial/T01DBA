# Chapter 9: Disk and Memory Commands

## 1. Disk Management Commands

These commands check disk usage, file system information, and manage disks.

| Command | Purpose | Example | Description |
|---|---|---|---|
| `df -h` | Show disk space usage of file systems | `df -h` | Human readable (KB/MB/GB) |
| `df -i` | Show inode usage | `df -i` | Inode info per file system |
| `du -sh <dir>` | Show size of a directory | `du -sh /home/user` | Summary of directory size |
| `du -h <dir>` | Show size of all files/subdirs | `du -h /var/log` | Detailed size list |
| `lsblk` | List block devices | `lsblk` | Shows disks and partitions |
| `blkid` | Show block device info (UUID, TYPE) | `blkid` | Displays UUID, TYPE, LABEL |
| `mount` | Show mounted file systems | `mount` | Currently mounted filesystems |
| `fdisk -l` | List all partitions | `fdisk -l` | Partition table of all disks |
| `free -h` | Show memory usage (also RAM + Swap) | `free -h` | Summary of used/free memory |
| `sync` | Sync data to disk | `sync` | Flush all pending writes to disk |

### Sample Output (`df -h`)

```
Filesystem   Size  Used  Avail  Use%  Mounted on
/dev/sda1     50G   20G    28G   42%  /
/dev/sda2    100G   60G    35G   64%  /home
tmpfs          2G     0     2G    0%  /dev/shm
```

## 2. Memory Management Commands

These commands monitor and analyze memory (RAM) usage.

| Command | Purpose | Example | Description |
|---|---|---|---|
| `free -h` | Show memory usage | `free -h` | Total, used, free, shared, buff/cache |
| `top` | Real-time process and memory usage | `top` | Interactive monitor (`q` to quit) |
| `htop` | Enhanced version of top | `htop` | User-friendly interface, if installed |
| `vmstat 1` | Virtual memory statistics | `vmstat 1` | Memory, CPU, I/O stats, updates every 1s |
| `cat /proc/meminfo` | Detailed memory information | `cat /proc/meminfo` | Shows all memory-related details |
| `swapon --show` | Show swap space usage | `swapon --show` | Displays active swap devices |
| `sar -r 1 5` | Memory usage report (5 samples, 1s apart) | `sar -r 1 5` | Historical memory stats (needs `sysstat`) |

### Sample Output (`free -h`)

```
        total  used  free  shared  buff/cache  available
Mem:     15G   5.2G  2.1G   200M      7.7G        8.9G
Swap:    2.0G  512M  1.5G      -         -        1.6G
```

### Legend

| Term | Meaning |
|---|---|
| RAM | Random Access Memory |
| Swap | Virtual memory on disk |
| Inode | Index of a file (metadata) |
| UUID | Unique identifier for a device/partition |

### Key Points

- Use `df -h` to check disk space.
- Use `du` to find large directories.
- Use `free -h` to monitor RAM and swap.
- Use `top`/`htop` for real-time monitoring.
- Use `cat /proc/meminfo` for complete memory detail.

---

## Where a PostgreSQL DBA uses this

Disk and memory monitoring for a DBA is not general housekeeping — it's directly tied to
whether the database stays up and how fast it runs.

### Preventing "disk full" outages

PostgreSQL **cannot accept writes** once its data disk fills up, and a full WAL disk can be
especially disruptive since PostgreSQL cannot even complete a checkpoint. This makes disk
monitoring one of the highest-priority recurring checks:

```bash
df -h                                   # overall disk usage across all mounts
df -h $PGDATA                            # usage specifically on the data volume
du -sh $PGDATA/pg_wal                    # WAL directory size — watch for runaway growth
du -sh $PGDATA/base/*                    # per-database size inside the cluster
```

A WAL directory that keeps growing usually means: a stuck replication slot, an archive
command that's failing, or a long-running transaction preventing WAL recycling — but the
first sign is always disk usage climbing in `du -sh pg_wal`.

### Finding what's actually eating space

```bash
du -h --max-depth=1 $PGDATA | sort -rh | head    # biggest subdirectories inside PGDATA
du -sh /var/log/postgresql/*                       # is a runaway log file the culprit?
```

### Sizing memory-related PostgreSQL settings

Two of the most important tuning parameters are set relative to *actual system RAM*, which
is exactly what `free -h` and `cat /proc/meminfo` report:

```bash
free -h
```
- `shared_buffers` — commonly ~25% of total RAM as a starting point
- `effective_cache_size` — commonly ~50–75% of total RAM (an estimate of OS+DB cache
  available for planning, not an allocation)

Both of these are meaningless to tune correctly without first knowing the real numbers from
`free -h`.

### Watching for memory pressure and swapping

```bash
vmstat 1 5           # watch 'si'/'so' (swap in/out) columns — nonzero means active swapping
swapon --show         # confirm whether swap is even configured, and how full it is
```
Heavy swapping on a database host is a red flag — it usually means `shared_buffers` plus
`work_mem` × concurrent sorts/hashes has been over-provisioned relative to actual RAM, and
it will show up as sudden, severe query latency spikes.

### Real-time triage during a performance incident

```bash
top                    # is CPU or memory the bottleneck right now?
free -h                # how much RAM is actually free/available?
df -h                  # is a disk near full, which can also cause I/O stalls?
```

### Checking block devices before choosing tablespace layout

```bash
lsblk                  # see attached disks/partitions
blkid                  # find UUIDs for /etc/fstab entries when mounting a new WAL disk
```

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. **Revisit the `/u01` partition from Chapter 3, this time for capacity.**
   ```bash
   df -h /u01
   ```
   Note total size, used, and available. This is exactly the number you'd watch in
   production to avoid a `PGDATA` disk-full outage.

2. **Size the actual PostgreSQL data directory.**
   ```bash
   du -sh /u01/pgsql/18/data
   ```
   Then break it down by subdirectory:
   ```bash
   du -h --max-depth=1 /u01/pgsql/18/data | sort -rh
   ```
   Identify which subdirectory is largest — usually `base/` (table/index data) or `pg_wal/`
   (write-ahead logs).

3. **Check system memory before touching `shared_buffers`.**
   ```bash
   free -h
   ```
   Write down total RAM. If you were setting `shared_buffers` right now, what value (~25% of
   RAM) would you choose?

4. **Confirm no swapping is happening under load.**
   ```bash
   vmstat 1 5
   ```
   Run a moderately heavy query in another session while this runs, and check whether the
   `si`/`so` (swap in/out) columns stay at zero.

5. **List block devices to understand the physical layout.**
   ```bash
   lsblk
   ```
   Identify which disk/partition backs `/u01` — this is useful context before ever asking
   "should WAL be on a separate disk from the data?"

**Next:** [Chapter 10 — Networking Commands](10-linux-networking-commands.md)
**Previous:** [Chapter 8 — Process Management](08-linux-process-management.md)
