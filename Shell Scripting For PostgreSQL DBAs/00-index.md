# Linux Complete Notes — for PostgreSQL DBAs

A chapter-by-chapter Linux tutorial, rebuilt from a visual "Linux Complete Notes" cheat-sheet
into full explanations, worked examples, and — for every topic — a section on **where a
PostgreSQL DBA actually uses this on the job**. PostgreSQL runs on Linux in the vast majority
of production deployments, and nearly every day-2 DBA task (checking WAL disk usage, tailing
logs, fixing `pg_hba.conf` permissions, finding a runaway backend, tuning `shared_buffers`
against available RAM) is really a Linux task wearing a PostgreSQL hat. This series treats
Linux as the DBA's primary tool, not a side skill.

## Chapters

| # | Chapter | What it covers | Core DBA use case |
|---|---------|-----------------|--------------------|
| 1 | [Introduction to Linux](01-introduction-to-linux.md) | What Linux is, kernel/shell/filesystem, distros | Choosing/understanding the OS PostgreSQL runs on |
| 2 | [Linux Architecture](02-linux-architecture.md) | Kernel space vs user space, process/memory/file/network management | Understanding why `shared_buffers`, I/O, and `postgres` processes behave the way they do |
| 3 | [Linux File System](03-linux-file-system.md) | FHS layout, `/etc`, `/var`, `/opt`, mount points, filesystem types | Locating `PGDATA`, config files, binaries, WAL, sockets |
| 4 | [Navigation Commands](04-linux-navigation-commands.md) | `pwd`, `cd`, `ls`, shortcuts | Moving around `PGDATA`, log directories, tablespaces |
| 5 | [File & Directory Commands](05-linux-file-directory-commands.md) | `touch`, `cat`, `less`, `head`, `tail`, `cp`, `mv`, `rm`, `ln` | Reading `postgresql.conf`, tailing logs, backing up config |
| 6 | [File Permissions](06-linux-file-permissions.md) | `rwx`, octal notation, `chmod`, `chown`, `chgrp` | Fixing PGDATA/`pg_hba.conf` permission errors, socket access |
| 7 | [Searching & Text Processing](07-linux-searching-text-processing.md) | `find`, `grep`, `sort`, `uniq`, `wc`, `cut`, `awk`, `sed` | Mining PostgreSQL logs for slow queries, errors, lock waits |
| 8 | [Process Management](08-linux-process-management.md) | `ps`, `top`, `htop`, `kill`, `nice`, job control | Finding/killing runaway `postgres` backends, checking the postmaster tree |
| 9 | [Disk & Memory Commands](09-linux-disk-memory-commands.md) | `df`, `du`, `free`, `vmstat`, `lsblk` | Watching WAL/disk growth, sizing `shared_buffers`/`effective_cache_size` |
| 10 | [Networking Commands](10-linux-networking-commands.md) | `ip`, `ping`, `ss`, `netstat`, `nmap`, `scp` | Diagnosing "connection refused", checking port 5432, replication links |
| 11 | [Package Management](11-linux-package-management.md) | `apt`, `yum`/`dnf`, `zypper`, `pacman`, `apk` | Installing PostgreSQL, extensions, `pg_repack`, minor version upgrades |
| 12 | [Shell Scripting Basics](12-linux-shell-scripting-basics.md) | Variables, conditionals, loops, functions, exit codes | Automating backups, health checks, failover scripts |
| 13 | [Cheat Sheet & Roadmap](13-linux-cheatsheet-roadmap.md) | Top 50 commands, learning roadmap, shortcuts | Quick reference during incidents |

## How to use this series

- Read chapters 1–3 once for the mental model (architecture + filesystem layout).
- Keep chapters 4–11 open as reference while working at the terminal.
- Chapter 12 turns everything into repeatable automation — the natural next step once the
  individual commands are second nature.
- Chapter 13 is a single-page cheat sheet for incident response, when you don't have time to
  read a full chapter.

Each chapter is self-contained and can be read out of order.
