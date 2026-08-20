# Chapter 2: Linux Architecture

## 1. What is Linux Architecture?

Linux Architecture describes the layers that make up the OS and how they interact with each
other and with hardware. Linux is a **monolithic kernel** — all major OS services (process,
memory, file system, device, and network management) run in kernel space, with a modular
design (loadable kernel modules) for flexibility.

## 2. High-Level Architecture (layers, top to bottom)

```
User (people / applications)
        ↕
User Applications (browsers, editors, compilers, shell, postgres client tools)
        ↕
System Libraries (glibc, libc)
        ↕
Linux Kernel
  ┌─────────────┬──────────────┬──────────────────┬────────────────┬──────────────────┐
  │ Process Mgmt │ Memory Mgmt  │ File System Mgmt  │ Device Mgmt    │ Network Mgmt     │
  └─────────────┴──────────────┴──────────────────┴────────────────┴──────────────────┘
        Kernel Modules (loadable)
        ↕
Hardware Abstraction Layer (HAL)
        ↕
Hardware (CPU, RAM, Disk, Network, Devices)
```

| Layer | Role |
|---|---|
| User | End users interacting through applications |
| User Applications | Programs users run — editors, browsers, shells, `psql`, `pgAdmin` |
| System Libraries | Standard functions/APIs (glibc) — the interface between programs and kernel |
| Linux Kernel | Core of the OS — manages all resources and system services |
| Kernel Modules | Loadable drivers/features without rebuilding the whole kernel |
| HAL | Hides hardware differences, presents a common interface to the kernel |
| Hardware | CPU, RAM, disk, network cards, devices |

## 3. Components of the Linux Kernel

- **Process Management** — creates, schedules, and terminates processes (this is what
  spawns and supervises every `postgres` backend process).
- **Memory Management** — allocates and manages RAM for processes (governs how much room
  `shared_buffers` and OS page cache actually have).
- **File System Management** — manages files, directories, and permissions (how `PGDATA`
  is read/written and protected).
- **Device Management** — controls I/O devices via drivers (disk controllers, NICs).
- **Network Management** — handles protocols and connections (how clients reach PostgreSQL
  over TCP/IP on port 5432).
- **System Calls Interface** — the API boundary between user-space programs (like `postgres`)
  and the kernel.

## 4. Kernel Space vs User Space

| Kernel Space | User Space |
|---|---|
| Privileged mode | Unprivileged mode |
| Full access to hardware | Limited, mediated access to hardware |
| Kernel code and services run here | Applications and user programs run here |
| Examples: device drivers, file system, scheduler | Examples: browser, editors, shell, `postgres` server process |

The `postgres` server process (and every backend it forks) runs entirely in **user space**.
When it needs to read a page from disk, write WAL, or send data over a socket, it makes a
**system call** into the kernel, which is the only code allowed to actually touch hardware.

## 5. Diagram (Summary)

```
User → Applications → System Libraries → Linux Kernel → Hardware
```

---

## Where a PostgreSQL DBA uses this

Understanding this layering explains *why* several PostgreSQL behaviors and tuning knobs
exist:

- **`shared_buffers` vs OS page cache** — PostgreSQL's own memory management (user space)
  sits on top of the kernel's memory management (kernel space). Data pages often exist in
  *two* caches: PostgreSQL's `shared_buffers` and the Linux page cache. This is exactly why
  the common guidance is "don't set `shared_buffers` to all your RAM" — the kernel needs
  room to cache too, and double-caching is expected behavior, not a bug.

- **Every connection is a process** — because Linux's process management model is what
  PostgreSQL uses (one OS process per backend, not one thread), `ps aux | grep postgres`
  shows one row per connection. This is also why `max_connections` has real memory cost: each
  backend is a full process with its own OS-level overhead.

- **File System Management in the kernel** is what enforces the Unix permissions that protect
  `PGDATA` — PostgreSQL refuses to start if data directory permissions are too open (`0700`
  is required) precisely because the kernel, not PostgreSQL, is the actual enforcer of file
  access rules.

- **System calls have a cost.** `fsync()` calls (used heavily by PostgreSQL for WAL durability)
  cross from user space into kernel space and down to the storage device. Understanding this
  boundary is essential when diagnosing why `synchronous_commit` or `wal_sync_method`
  settings affect throughput — every commit potentially means a user-space → kernel-space →
  hardware round trip.

- **Network Management in the kernel** handles the actual TCP/IP stack; PostgreSQL's
  `listen_addresses` and `pg_hba.conf` operate at the application layer on top of that,
  which is why network-level problems (firewalls, `sysctl` limits) can block PostgreSQL
  connections even when `pg_hba.conf` is configured correctly.

### Practical commands tying architecture to PostgreSQL

```bash
ps -ef | grep postgres          # user-space view: one process per backend
cat /proc/<pid>/status          # kernel-exposed info about a specific postgres process
sysctl -a | grep shm            # kernel parameters affecting shared memory (shared_buffers)
```

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. **See the kernel your PostgreSQL server depends on.**
   ```bash
   uname -a
   ```
   Identify the kernel version and CPU architecture (x86_64, aarch64, etc.). PostgreSQL
   binaries you download must match this architecture.

2. **Look at the process tree, before PostgreSQL is even installed.**
   ```bash
   ps -ef | head -20
   ```
   Find `PID 1` — this is `init`/`systemd`, the ultimate parent of every process on the box,
   including (once installed) the PostgreSQL `postmaster`.

3. **User space vs kernel space, made concrete.** Run:
   ```bash
   cat /proc/meminfo | head -5
   ```
   `/proc` is a *virtual* filesystem — a live window the kernel exposes into its own memory
   management, from user space, without you needing kernel privileges to read it. This is
   architecture Chapter 2 made tangible: user-space commands (`cat`) reading kernel-space
   state.

4. **Predict, then verify.** Before installing PostgreSQL, predict: will the `postgres`
   server run as one process or many? After Chapter 11's install, revisit this with
   `ps -ef | grep postgres` and see if your prediction from the architecture model was right.

**Next:** [Chapter 3 — Linux File System](03-linux-file-system.md)
**Previous:** [Chapter 1 — Introduction to Linux](01-introduction-to-linux.md)
