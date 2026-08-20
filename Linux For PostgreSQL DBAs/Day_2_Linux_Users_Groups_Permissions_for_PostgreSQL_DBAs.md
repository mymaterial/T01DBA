# Day 2 — Linux Users, Groups, Permissions & Files

## Goal

Understand Linux users, groups, ownership and permissions — the foundation of system security and PostgreSQL operations.

---

# 1. Filesystem Recap

Important Linux directories:

```text
/
├── bin
├── etc
├── home
├── var
├── tmp
├── usr
├── opt
└── root
```

These are directories in the Linux filesystem hierarchy. Some may be separate mount points depending on server configuration.

---

# 2. Important Commands — Recap and New

## Current directory

```bash
pwd
```

## List files

```bash
ls
```

## Long listing

```bash
ls -l
```

## List hidden files

```bash
ls -la
```

## Long listing sorted by modification time

```bash
ls -ltr
```

`ls -ltr`:

```text
-l  = long listing
-t  = sort by modification time
-r  = reverse the order
```

This is useful when looking at logs, generated files and other DBA files.

> `ls -ltr` shows modification time, not necessarily file creation time.

---

# 3. Reading Files

```bash
cat file.txt
```

Example:

```bash
cat /etc/os-release
```

---

# 4. date

Display the system date and time:

```bash
date
```

Example:

```text
Mon May 18 10:30:45 IST 2025
```

---

# 5. hostname

Display the system hostname:

```bash
hostname
```

Example:

```text
lab01
```

---

# 6. man — Manual Pages

Linux provides manual pages for commands.

Syntax:

```bash
man <command>
```

Example:

```bash
man ls
```

Other examples:

```bash
man date
man hostname
man chmod
man chown
```

Important habit:

> If you do not remember a command option, use `man`.

---

# 7. Users, Groups and Ownership

Every file and directory has:

```text
Owner
Group
Permissions
```

Example:

```text
-rw-r--r--. 1 kumar dba 123 file.txt
```

Conceptually:

```text
Users
  |
  v
Groups
  |
  v
Files / Directories
  |
  v
Permissions
```

---

# 8. whoami

Show the current logged-in user:

```bash
whoami
```

---

# 9. id

Show user and group information:

```bash
id
```

For a specific user:

```bash
id kumar
```

PostgreSQL example:

```bash
id postgres
```

---

# 10. groupadd

Create a group:

```bash
sudo groupadd dba
```

Example:

```bash
groupadd dba
```

---

# 11. useradd

Create a user and add the user to a group:

```bash
sudo useradd -m -G dba user1
```

Meaning:

```text
-m     = create home directory
-G dba = add user to the dba group
```

---

# 12. touch

Create an empty file:

```bash
touch project.txt
```

Check it:

```bash
ls -l project.txt
```

---

# 13. chown — Change Ownership

Change the owner:

```bash
sudo chown kumar project.txt
```

Change owner and group:

```bash
sudo chown user1:dba project.txt
```

Check:

```bash
ls -l project.txt
```

Example:

```text
-rw-r--r-- 1 user1 dba 0 project.txt
```

---

# 14. chgrp — Change Group

Change the group ownership:

```bash
sudo chgrp dba project.txt
```

Check:

```bash
ls -l project.txt
```

---

# 15. Linux File Permissions

Linux permissions are applied to:

```text
Owner
Group
Others
```

Three basic permission types:

```text
r = read
w = write
x = execute
```

Numeric values:

```text
r = 4
w = 2
x = 1
- = 0
```

---

# 16. Permission Calculation

Common combinations:

```text
rwx = 7
rw- = 6
r-x = 5
r-- = 4
--- = 0
```

Examples:

```text
rwxrwx---
 7   7   0

rwxr-----
 7   4   0

rw-------
 6   0   0
```

---

# 17. Understanding ls -l Permissions

Example:

```text
-rwxr-xr--
```

Breakdown:

```text
-       = file type
rwx     = owner permissions
r-x     = group permissions
r--     = others permissions
```

For a directory:

```text
d
```

is commonly shown as the file type.

---

# 18. chmod — Change Permissions

Syntax:

```bash
chmod <mode> <file/directory>
```

Example:

```bash
chmod 770 file.txt
```

Meaning:

```text
Owner  = rwx
Group  = rwx
Others = ---
```

Example:

```bash
chmod 740 file.txt
```

Meaning:

```text
Owner  = rwx
Group  = r--
Others = ---
```

Example:

```bash
chmod 600 file.txt
```

Meaning:

```text
Owner  = rw-
Group  = ---
Others = ---
```

Recursive example:

```bash
chmod -R 755 /home/dba
```

> Give only the minimum permissions required. Avoid unnecessarily broad permissions.

---

# 19. Hidden Files

Files beginning with `.` are hidden from a normal `ls` listing.

Examples:

```text
.bash_history
.bash_logout
.bash_profile
.bashrc
.pgpass
```

View hidden files:

```bash
ls -la
```

---

# 20. .pgpass — PostgreSQL Password File

The PostgreSQL `.pgpass` file can be used for non-interactive password authentication.

Location:

```text
~/.pgpass
```

Format:

```text
hostname:port:database:username:password
```

Example:

```text
localhost:5432:postgres:kumar:mypassword
```

Very important:

```bash
chmod 600 ~/.pgpass
```

This means:

```text
Owner = read/write
Group = no permissions
Others = no permissions
```

---

# 21. Linux Permissions and PostgreSQL

PostgreSQL normally runs under a dedicated operating-system account, commonly:

```text
postgres
```

Check:

```bash
id postgres
```

Important DBA concepts:

```text
PostgreSQL OS user
       |
       v
PostgreSQL data directory
       |
       v
File ownership
       |
       v
Linux permissions
```

PostgreSQL data files should be protected appropriately.

DBAs should avoid using `root` for routine PostgreSQL operations.

---

# 22. Common PostgreSQL Paths

Example data directory:

```text
/var/lib/pgsql/data
```

Example configuration file:

```text
/var/lib/pgsql/data/postgresql.conf
```

Example log location:

```text
/var/lib/pgsql/data/log/postgresql.log
```

Password file:

```text
~/.pgpass
```

> PostgreSQL paths vary by version, operating system and installation method. Verify the actual paths on your server.

---

# 23. Hands-on User / Group / Ownership Lab

Create a group:

```bash
sudo groupadd dba
```

Create a user:

```bash
sudo useradd -m -G dba user1
```

Create a file:

```bash
touch project.txt
```

Check ownership:

```bash
ls -l project.txt
```

Change owner and group:

```bash
sudo chown user1:dba project.txt
```

Check again:

```bash
ls -l project.txt
```

---

# 24. Hands-on Permission Lab

Create a test file:

```bash
touch file.txt
```

Set:

```bash
chmod 770 file.txt
```

Check:

```bash
ls -l file.txt
```

Try:

```bash
chmod 740 file.txt
ls -l file.txt
```

Then:

```bash
chmod 600 file.txt
ls -l file.txt
```

---

# 25. Day 2 Command Summary

```bash
pwd
ls
ls -l
ls -la
ls -ltr
cat
date
hostname
man
whoami
id
groupadd
useradd
touch
chown
chgrp
chmod
```

PostgreSQL-related examples:

```bash
id postgres
chmod 600 ~/.pgpass
```

---

# 26. Day 2 Key Takeaways

1. Every file/directory has an owner and group.
2. Permissions are applied to owner, group and others.
3. `r = 4`, `w = 2`, `x = 1`.
4. `chmod` changes permissions.
5. `chown` changes ownership.
6. `chgrp` changes group ownership.
7. `ls -l` helps inspect ownership and permissions.
8. `ls -la` displays hidden files.
9. `.pgpass` is a PostgreSQL-related hidden file.
10. `.pgpass` should use restrictive permissions such as `600`.
11. PostgreSQL normally runs under a dedicated OS account such as `postgres`.
12. Linux permissions are an important part of PostgreSQL security.

---

## Day 2 Mental Model

```text
                 Linux
                   |
        +----------+----------+
        |                     |
      Users                 Groups
        |                     |
        +----------+----------+
                   |
                   v
            Files / Directories
                   |
                   v
              Ownership
                   |
                   v
             Permissions
              /    |    \
             r     w     x
             4     2     1
                   |
                   v
                chmod
                   |
                   v
             PostgreSQL
```

---

## Day 2 DBA Connection

```text
Linux User
    |
    v
postgres OS account
    |
    v
PostgreSQL files
    |
    v
Ownership + Permissions
    |
    v
Secure PostgreSQL operation
```

> Practice these commands in a lab VM. Do not experiment with ownership or permissions on a production PostgreSQL data directory unless you understand the impact.
