# Chapter 13: Linux Cheat Sheet & Roadmap

Top 50 essential Linux commands, for beginners to advanced.

## 1. Linux Learning Roadmap

1. Introduction
2. Linux Architecture
3. File System
4. Navigation Commands
5. File & Directory Commands
6. File Permissions
7. Searching & Text Processing
8. Process Management
9. Disk & Memory Commands
10. Networking Commands
11. Package Management
12. Shell Scripting
13. User Management
14. Cron Jobs (Scheduling)
15. System Administration
16. Bash Projects

## 2. Top 50 Linux Commands

| # | Command | Purpose | # | Command | Purpose |
|---|---|---|---|---|---|
| 1 | `pwd` | Print current directory | 26 | `top` | Real-time process monitor |
| 2 | `ls` | List directory contents | 27 | `htop` | Enhanced process monitor |
| 3 | `cd` | Change directory | 28 | `kill` | Kill process by PID |
| 4 | `mkdir` | Make new directory | 29 | `pkill` | Kill process by name |
| 5 | `rmdir` | Remove empty directory | 30 | `df` | Disk space usage |
| 6 | `touch` | Create empty file | 31 | `du` | Disk usage of files/dirs |
| 7 | `cp` | Copy files/directories | 32 | `free` | Memory usage |
| 8 | `mv` | Move/rename files | 33 | `lsblk` | List block devices |
| 9 | `rm` | Remove files/directories | 34 | `ping` | Check network connectivity |
| 10 | `cat` | Display file contents | 35 | `ip` | Show/modify IP addr & routes |
| 11 | `less` | View file content (pager) | 36 | `ss` | Socket statistics |
| 12 | `head` | Show first lines of file | 37 | `netstat` | Network statistics |
| 13 | `tail` | Show last lines / follow | 38 | `traceroute` | Trace network path |
| 14 | `nano` | Text editor (nano) | 39 | `curl` | Transfer data from/to server |
| 15 | `vim` | Text editor (vim) | 40 | `wget` | Download files from web |
| 16 | `find` | Search files in system | 41 | `apt` | Package manager (Debian/Ubuntu) |
| 17 | `locate` | Find files by name | 42 | `yum` | Package manager (RHEL/CentOS) |
| 18 | `grep` | Search text pattern | 43 | `dnf` | Next-gen package manager |
| 19 | `sort` | Sort lines in file | 44 | `systemctl` | Control systemd services |
| 20 | `uniq` | Remove duplicate lines | 45 | `service` | Manage services (SysV) |
| 21 | `wc` | Count lines/words/bytes | 46 | `journalctl` | View system logs |
| 22 | `chmod` | Change file permissions | 47 | `tar` | Archive files |
| 23 | `chown` | Change file owner | 48 | `zip` | Compress files |
| 24 | `chgrp` | Change group ownership | 49 | `unzip` | Extract zip files |
| 25 | `ps` | Show running processes | 50 | `bash` | Bash shell |

## 3. Command Categories

- File Management · Permissions · Search · Process Management · Disk & Memory · Networking
- Package Management · Shell Scripting · User Management

## 4. Linux File System (FHS) — quick recall

```
/
├── bin   Essential user binaries
├── boot  Boot loader files
├── dev   Device files
├── etc   System configuration files
├── home  User home directories
├── lib   System libraries
├── media Mount points for removable media
├── opt   Optional applications
├── proc  Process & kernel information
├── root  Root user's home directory
├── run   Runtime data
├── sbin  System binaries
├── srv   Data for services provided by system
├── tmp   Temporary files
├── usr   User programs & data
└── var   Variable data (logs, cache, spool)
```
Everything in Linux starts from root (`/`).

## 5. Quick Tips

- Use `man <command>` for help.
- Press **Tab** for auto-complete.
- Use `history` to view previous commands.
- `Ctrl+C` stops a running process.
- `Ctrl+R` searches command history.
- Use `clear` to clear the terminal.
- `!!` repeats the last command.

## 6. Example Workflow (day to day)

```
Search File → View Content → Edit File → Run/Execute → Check Logs
find . -name "file.txt" → cat/less file.txt → nano/vim file.txt → bash script.sh or ./a.out → tail -f /var/log/messages
```

## 7. Handy Shortcuts

| Shortcut | Action |
|---|---|
| Ctrl+A | Move to beginning of line |
| Ctrl+E | Move to end of line |
| Ctrl+U | Cut from start to cursor |
| Ctrl+K | Cut from cursor to end |
| Ctrl+L | Clear screen |
| Ctrl+D | Logout / exit |
| Tab | Auto-complete |
| Up/Down | Navigate history |

**Remember:** Linux is all about practice and consistency. The more you use it, the more
powerful you become.

---

## Where a PostgreSQL DBA uses this

Treat this chapter as your **incident-response quick reference** — the single page you keep
open during an on-call page when there's no time to read a full chapter.

### The 60-second PostgreSQL host triage

```bash
# 1. Is PostgreSQL running?
ps -ef | grep postgres
systemctl status postgresql

# 2. Is disk the problem?
df -h

# 3. Is memory the problem?
free -h

# 4. Is CPU the problem?
top

# 5. What does the log say?
tail -100 /var/log/postgresql/postgresql-*.log

# 6. Is the port open?
ss -tuln | grep 5432

# 7. Any recent config changes?
ls -lt /etc/postgresql/*/main/*.conf
```

### Mapping the roadmap onto a DBA skill path

| Roadmap step | DBA-specific milestone |
|---|---|
| 1–3 Fundamentals | Know the FHS well enough to find `PGDATA`/config blind on any host |
| 4–5 Navigation & files | Comfortable tailing logs, editing config, moving backups |
| 6 Permissions | Can diagnose and fix any `PGDATA`/`pg_hba.conf` permission error |
| 7 Search & text | Can mine gigabytes of PostgreSQL logs for the one relevant error |
| 8 Process mgmt | Can identify and safely terminate a runaway backend |
| 9 Disk & memory | Can size `shared_buffers`/`effective_cache_size` and catch disk-full risk early |
| 10 Networking | Can distinguish a firewall problem from a `pg_hba.conf` problem in seconds |
| 11 Package mgmt | Can install/patch PostgreSQL and extensions via the correct repo for any distro |
| 12 Shell scripting | Can automate backups, health checks, and failover logic reliably |
| 13+ (User mgmt, cron, sysadmin) | Natural next steps: OS user/group management for `postgres`, `cron` for scheduled maintenance, and broader `systemd`/log-rotation administration |

### A one-line reference card worth memorizing

```bash
sudo -u postgres psql               # connect as the postgres superuser locally
tail -f /var/log/postgresql/*.log   # watch logs live
df -h && free -h                     # disk + memory, always check both together
ps -ef --forest | grep postgres     # see the whole postmaster process tree
systemctl status postgresql          # is the service itself healthy per systemd
```

That's the roadmap end-to-end — from "what is Linux" to a working, automated PostgreSQL
operations toolkit built entirely on the commands in Chapters 1–12.

## Capstone Exercise — the actual end-of-Day-1 bootcamp homework

This is the exact assignment given at the end of the real bootcamp session, using only
commands from Chapters 4, 5, and 7. Do it cold, without looking anything up, to check how
much has stuck.

**The task, verbatim in spirit:**
> "My PostgreSQL configuration file is in `/u01/pgsql/18`. The file name is
> `postgresql.conf`. Open it and tell me the value for `work_mem`."

**Solve it now:**
```bash
cd /u01/pgsql/18
ls
cd data              # or wherever ls shows postgresql.conf actually living
ls
cat postgresql.conf
```
Scroll (or better — don't; use what you learned in Chapter 7):
```bash
grep work_mem postgresql.conf
```
**Expected answer:** `work_mem = 4MB` (the PostgreSQL default, commonly left commented-out
and therefore still in effect).

**Self-check — could you do all of this without hints?**

| Step | Command | Chapter |
|---|---|---|
| Connect to the lab machine | PuTTY with IP, username, password | 1 |
| Confirm where you are | `pwd` | 4 |
| Navigate to the target folder | `cd /u01/pgsql/18` (absolute path) | 4 |
| See what's there | `ls` | 4/5 |
| Go deeper if needed | `cd data` (relative path) | 4 |
| Open/search the file | `cat postgresql.conf` or `grep work_mem postgresql.conf` | 5/7 |
| Verify the disk it lives on | `df -h /u01` | 9 |

If every row was fast and automatic, Chapters 1–12 have done their job — you're ready to
layer real PostgreSQL administration (installation, backup, replication, tuning) on top of
this foundation, which is exactly what the bootcamp does starting Day 2.

### One more drill: repeat cold, with a twist

Do the exact same exercise again, but this time look up `shared_buffers` instead of
`work_mem`, and do it from your home directory using a **single absolute-path command** with
no `cd` at all:
```bash
grep shared_buffers /u01/pgsql/18/data/postgresql.conf
```

**Previous:** [Chapter 12 — Shell Scripting Basics](12-linux-shell-scripting-basics.md)
**Back to:** [Index](00-index.md)
