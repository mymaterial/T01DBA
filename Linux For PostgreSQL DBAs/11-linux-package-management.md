# Chapter 11: Linux Package Management

Package management tools help you install, update, configure, and remove software on Linux
systems.

## 1. Package Management Overview

A package is a bundle of software that includes programs, configuration files,
documentation, and dependencies.

| Package Manager | Used In |
|---|---|
| APT (Advanced Package Tool) | Debian, Ubuntu, Linux Mint |
| YUM / DNF | RHEL, CentOS, Fedora |
| Zypper | openSUSE, SUSE Linux |
| Pacman | Arch Linux, Manjaro |
| APK | Alpine Linux |

## 2. APT (Debian/Ubuntu)

| Command | Purpose | Example |
|---|---|---|
| `sudo apt update` | Update package lists | `sudo apt update` |
| `sudo apt upgrade` | Upgrade all packages | `sudo apt upgrade` |
| `sudo apt install <pkg>` | Install a package | `sudo apt install nginx` |
| `sudo apt remove <pkg>` | Remove a package | `sudo apt remove nginx` |
| `sudo apt purge <pkg>` | Remove package + config files | `sudo apt purge nginx` |
| `sudo apt autoremove` | Remove unused dependencies | `sudo apt autoremove` |
| `apt search <keyword>` | Search for packages | `apt search python` |
| `apt show <pkg>` | Show package information | `apt show vim` |
| `dpkg -l` | List installed packages | `dpkg -l \| less` |

**Tip:** Always run `sudo apt update` before installing or upgrading packages.

## 3. YUM / DNF (RHEL, CentOS, Fedora)

| Command | Purpose | Example |
|---|---|---|
| `sudo yum update` | Update package lists | `sudo yum update` |
| `sudo yum upgrade` | Upgrade all packages | `sudo yum upgrade` |
| `sudo yum install <pkg>` | Install a package | `sudo yum install httpd` |
| `sudo yum remove <pkg>` | Remove a package | `sudo yum remove httpd` |
| `sudo yum autoremove` | Remove unused dependencies | `sudo yum autoremove` |
| `yum search <keyword>` | Search for packages | `yum search java` |
| `yum info <pkg>` | Show package information | `yum info nginx` |
| `rpm -qa` | List installed packages | `rpm -qa \| less` |

Modern systems (Fedora 22+, RHEL 8+) use `dnf` instead of `yum` — commands are mostly the
same, e.g. `dnf install nginx`.

## 4. Zypper (openSUSE / SUSE)

| Command | Purpose | Example |
|---|---|---|
| `sudo zypper refresh` | Refresh repositories | `sudo zypper refresh` |
| `sudo zypper update` | Update all packages | `sudo zypper update` |
| `sudo zypper install <pkg>` | Install a package | `sudo zypper install vim` |
| `sudo zypper remove <pkg>` | Remove a package | `sudo zypper remove vim` |
| `zypper search <keyword>` | Search for packages | `zypper search apache2` |
| `zypper info <pkg>` | Show package info | `zypper info nginx` |
| `zypper se -i` | List installed packages | `zypper se -i` |

## 5. Pacman (Arch Linux / Manjaro)

| Command | Purpose | Example |
|---|---|---|
| `sudo pacman -Sy` | Sync package databases | `sudo pacman -Sy` |
| `sudo pacman -S <pkg>` | Install a package | `sudo pacman -S git` |
| `sudo pacman -R <pkg>` | Remove a package | `sudo pacman -R git` |
| `sudo pacman -Rs <pkg>` | Remove + dependencies (not required by others) | `sudo pacman -Rs git` |
| `pacman -Ss <keyword>` | Search for packages | `pacman -Ss docker` |
| `pacman -Si <pkg>` | Show package info | `pacman -Si nginx` |
| `pacman -Qe` | List installed packages | `pacman -Qe` |

## 6. APK (Alpine Linux)

| Command | Purpose | Example |
|---|---|---|
| `apk update` | Update package index | `apk update` |
| `apk upgrade` | Upgrade all packages | `apk upgrade` |
| `apk add <pkg>` | Install a package | `apk add nginx` |
| `apk del <pkg>` | Remove a package | `apk del nginx` |
| `apk search <keyword>` | Search for packages | `apk search python` |
| `apk info <pkg>` | Show package info | `apk info nginx` |
| `apk list --installed` | List installed packages | `apk list --installed` |

## 7. Useful Tips

- Run update/refresh before installing new packages.
- Use autocompletion with Tab to save time.
- Check package description before installing.
- Remove unnecessary packages to free space.
- Read man pages: `man apt`, `man yum`, `man pacman`

## 8. Example Workflow (APT)

```
Update Repositories → Install Package → Manage/Configure (use it) → Remove if not needed
sudo apt update    → sudo apt install <pkg> → (configure & use)    → sudo apt remove <pkg>
```

## 9. Check Installed Package Version

- APT: `dpkg -s <package> | grep Version`
- YUM/DNF: `rpm -qi <package> | grep Version`
- Zypper: `zypper info <package> | grep Version`
- Pacman: `pacman -Qi <package> | grep Version`
- APK: `apk info -v <package>`

### Caution

Be careful with remove/purge commands. Important system packages should never be removed
carelessly.

---

## Where a PostgreSQL DBA uses this

This chapter is exactly how PostgreSQL itself, its extensions, and its supporting tools get
onto (and stay current on) a server.

### Installing PostgreSQL — the *right* repository matters

Distro-default repos often ship an old PostgreSQL version, so most production installs add
the official **PGDG (PostgreSQL Global Development Group)** repository first:

**Debian/Ubuntu (APT):**
```bash
sudo apt update
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh   # adds the PGDG repo
sudo apt update
sudo apt install -y postgresql-16 postgresql-client-16 postgresql-contrib-16
```

**RHEL/CentOS/Rocky (DNF/YUM):**
```bash
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo dnf -qy module disable postgresql   # disable the distro's built-in (older) module
sudo dnf install -y postgresql16-server postgresql16-contrib
```

### Installing extensions and companion tools

```bash
sudo apt install postgresql-16-postgis-3     # PostGIS
sudo apt install postgresql-16-pgaudit        # pgAudit
sudo apt install postgresql-16-repack         # pg_repack, for online table bloat cleanup
sudo apt install pgbouncer                     # connection pooler
sudo apt install pgbackrest                    # backup/restore tool
```

### Checking what version is actually installed (before an upgrade)

```bash
dpkg -l | grep postgresql          # Debian/Ubuntu
rpm -qa | grep postgresql          # RHEL/CentOS
psql --version                      # client version specifically
```

### Minor version patching (security/bugfix updates)

```bash
sudo apt update && sudo apt install --only-upgrade postgresql-16   # Debian/Ubuntu
sudo dnf update postgresql16-server                                  # RHEL/CentOS
```
Minor version upgrades (e.g. 16.2 → 16.3) are binary-compatible and don't require
`pg_upgrade` — only a service restart. Major version upgrades (15 → 16) do require
`pg_upgrade` or a dump/restore, which is a separate, much bigger procedure.

### A typical new-server bootstrap sequence

```bash
sudo apt update
sudo apt install -y postgresql-16 postgresql-contrib-16 postgresql-16-repack pgbackrest
sudo systemctl enable --now postgresql
sudo -u postgres psql -c "SELECT version();"
```

### Why the repo/package manager choice matters operationally

Knowing whether a fleet is Debian-family or RHEL-family determines:
- Which config path convention applies (see Chapter 3)
- Which service name to use with `systemctl` (`postgresql` vs `postgresql-16`)
- How you'll script fleet-wide patching in configuration management tools (Ansible, Puppet)
  that branch on `ansible_os_family` or similar

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

Given the bootcamp's lab machines are OEL/CentOS-family (Fedora flavor), these exercises use
`dnf`/`yum` and `rpm` — the same commands you'd use on RHEL or Rocky Linux in production.

1. **Confirm what's already installed.**
   ```bash
   rpm -qa | grep -i postgres
   ```
   If PostgreSQL 18 is already on your lab machine, this should list its packages
   (`postgresql18-server`, `postgresql18-contrib`, etc.).

2. **Check the exact installed version.**
   ```bash
   /u01/pgsql/18/bin/psql --version
   ```
   or, if `psql` is on your `PATH`:
   ```bash
   psql --version
   ```

3. **List everything a package installed, if PostgreSQL was installed via RPM.**
   ```bash
   rpm -ql postgresql18-server | head -20
   ```
   Confirm you can see the binary directory and the systemd unit file.

4. **Practice the search flow (safe — doesn't install anything).**
   ```bash
   dnf search postgresql
   dnf info postgresql18-server
   ```

5. **If you're on the Debian/Ubuntu machine instead** (the bootcamp's "secondary" OS for one
   day of the course), repeat with the APT equivalents:
   ```bash
   dpkg -l | grep postgresql
   apt show postgresql-16
   ```
   Compare: same underlying task, different command family — exactly the "90% same, 10%
   different across flavors" point made at the start of this series.

**Next:** [Chapter 12 — Shell Scripting Basics](12-linux-shell-scripting-basics.md)
**Previous:** [Chapter 10 — Networking Commands](10-linux-networking-commands.md)
