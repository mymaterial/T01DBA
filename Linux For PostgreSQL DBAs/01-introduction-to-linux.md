# Chapter 1: Introduction to Linux

## 1. What is Linux?

Linux is a free, open-source operating system originally created by **Linus Torvalds in
1991**. Technically, "Linux" refers to the **kernel** — the core program that talks to
hardware — while a full "Linux distribution" (distro) bundles the kernel with a shell,
system libraries, and applications. Linux is based on the design principles of **Unix**.

It's used everywhere: web servers, cloud infrastructure, container platforms, embedded
devices, Android phones, and supercomputers. It is also, overwhelmingly, the operating
system PostgreSQL runs on in production.

## 2. Why Linux? (Features)

| Feature | What it means |
|---|---|
| Open Source | Source code is public; free to use, modify, redistribute |
| Multiuser | Many users can work on the system at once, each with isolated permissions |
| Multitasking | Many processes run concurrently, scheduled by the kernel |
| Secure | Strong permission model, SELinux/AppArmor, minimal default attack surface |
| Stable | Can run for months/years without a reboot |
| Portable | Runs on x86, ARM, and many other architectures |

## 3. Components

| Component | Role |
|---|---|
| **Kernel** | The core — manages CPU, memory, devices, and processes |
| **Shell** | The command-line interface that interprets what you type (bash, zsh, etc.) |
| **File System** | Organizes everything — files, directories, and even devices — as files |
| **Applications** | User-facing programs, e.g. `postgres`, `psql`, `pg_dump` |

## 4. Popular Distributions

| Distro | Family | Typical PostgreSQL relevance |
|---|---|---|
| Ubuntu | Debian-based | Very common on cloud VMs; uses `apt` |
| Debian | Debian-based | Conservative, stable; common base image |
| Fedora / RHEL / CentOS / Rocky / Alma | RPM-based | Common in enterprise; uses `yum`/`dnf` |
| Kali Linux | Debian-based (security) | Rarely used for production DB hosts |
| Linux Mint | Debian-based | Desktop-oriented, rarely a server OS |

Which distro you're on determines your **package manager** (Chapter 11) and some filesystem
conventions (Chapter 3) — e.g. PostgreSQL data lives under `/var/lib/postgresql/<version>/main`
on Debian/Ubuntu, but under `/var/lib/pgsql/<version>/data` on RHEL/CentOS.

## 5. Advantages

- Free — no licensing cost for the OS itself
- Secure — least-privilege by default, granular permissions
- Fast and resource-efficient — no OS "tax" competing with `shared_buffers`
- Reliable — long uptimes suit long-running database servers
- Highly customizable — you tune the kernel, filesystem, and services around the workload

## 6. Basic Commands (preview)

| Command | Purpose |
|---|---|
| `pwd` | Present working directory |
| `ls` | List files |
| `cd` | Change directory |
| `mkdir` | Create folder |
| `rm` | Remove files |
| `cp` | Copy files |
| `mv` | Move/rename files |
| `cat` | View file contents |

These are covered in full in Chapters 4–5.

## 7. Linux File System Structure (preview)

```
/
├── home    → User files
├── etc     → System configuration files
├── var     → Variable data (logs, PostgreSQL data on Debian/Ubuntu)
├── usr     → User programs
└── bin     → Essential user commands
```

Full detail in Chapter 3.

---

## Where a PostgreSQL DBA uses this

- **Distro choice affects everything downstream.** Before you `apt install postgresql` or
  `dnf install postgresql-server`, you need to know which family you're on — it changes the
  data directory path, the service name (`postgresql` vs `postgresql-15`), and the package
  repository you add (PGDG apt repo vs PGDG yum repo).

  ```bash
  cat /etc/os-release        # confirms distro + version, works on almost every distro
  ```

- **Multiuser model is why PostgreSQL runs as its own OS user** (`postgres` by default). The
  database process never needs to run as `root`; Linux's permission model is what makes that
  safe and enforceable.

- **Stability and uptime** are exactly why Linux is the default choice for database hosts —
  PostgreSQL instances routinely run for months between kernel patch windows, and Linux is
  built for that.

- **"Everything is a file"** (a Unix/Linux philosophy point) is directly why Unix-domain
  sockets work: by default, `psql` connects to PostgreSQL through a file-like socket
  (`/var/run/postgresql/.s.PGSQL.5432`) rather than a network port. Understanding that Linux
  treats devices, pipes, and sockets as files demystifies a lot of PostgreSQL's local
  connection behavior.

### Quick check for a new PostgreSQL host

```bash
uname -a              # kernel version and architecture
cat /etc/os-release    # distro name/version — determines package manager & paths
whoami                 # confirm which OS user you're operating as
```

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

These mirror how a real PostgreSQL DBA bootcamp opens Day 1: get a terminal, understand what
you're logged into, and connect the OS to the database before typing a single `psql` command.

**Setup:** Connect to your lab VM via PuTTY (or any SSH client) using its IP address, e.g.
`192.168.234.131`, with your assigned username and password. Once connected you'll see a
prompt like:
```
root@lab01:~#
```
This tells you: **user** = `root`, **hostname** = `lab01`, **location** = `~` (home
directory), and `#` means you're logged in as a privileged (root) user (a regular user would
see `$`).

1. **Identify your OS family.** Run:
   ```bash
   cat /etc/os-release
   ```
   Is this box Fedora-family (RHEL/CentOS/OEL/Rocky/Alma) or Debian-family (Ubuntu/Debian)?
   Write down the exact distro and version — you'll need this later to pick the correct
   PostgreSQL package repository (Chapter 11).

2. **Why does the flavor matter?** Explain in one sentence why a command that works on
   CentOS is guaranteed to work on Oracle Enterprise Linux (OEL) and RHEL, but is *not*
   guaranteed to work unchanged on Ubuntu.

3. **Resume-readiness check.** A real interview question: "What operating system is your
   PostgreSQL hosted on?" Practice answering it for your own lab environment using what
   `cat /etc/os-release` told you — e.g. "I've worked with PostgreSQL on RHEL/OEL and on
   Ubuntu."

4. **Confirm your identity and machine.** Run `whoami` and `hostname`. Confirm both match
   what the shell prompt already told you (`root` and `lab01` in the example above).

> **Why this matters for PostgreSQL:** every later exercise in this series assumes you can
> answer "what am I logged into, and what OS family is this?" in five seconds. That's the
> whole point of Chapter 1.

**Next:** [Chapter 2 — Linux Architecture](02-linux-architecture.md)
