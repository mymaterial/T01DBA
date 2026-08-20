# Day 1 — Linux Fundamentals for PostgreSQL DBAs

## Goal

Get comfortable with the Linux environment, basic navigation, and essential commands used by PostgreSQL DBAs.

---

# 1. Linux Filesystem — Basic Mental Model

Linux uses `/` as the root of the filesystem.

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

Important beginner concepts:

- `/` = root of the Linux filesystem
- `/home` = user home directories
- `/root` = root user's home directory
- `/etc` = system configuration
- `/var` = variable data such as logs and database-related data
- `/tmp` = temporary files
- `/usr` = programs and libraries
- `/opt` = optional / third-party software

> These are directories in the Linux filesystem hierarchy. Some directories may be separate mount points depending on the server configuration.

---

# 2. Linux vs Windows Paths

Linux:

```text
/home/kumar/data/file.txt
```

Windows:

```text
C:\Users\Kumar\data\file.txt
```

Linux uses:

```text
/
```

Windows commonly uses:

```text
\
```

The purpose is the same:

```text
Organize → Store → Access data
```

---

# 3. pwd — Print Working Directory

Show your current location:

```bash
pwd
```

Example:

```text
/home/kumar
```

---

# 4. ls — List Files and Directories

Basic listing:

```bash
ls
```

Example:

```text
bin
etc
home
var
tmp
```

Long listing:

```bash
ls -l
```

List hidden files:

```bash
ls -la
```

---

# 5. cd — Change Directory

Change to a directory:

```bash
cd /etc
```

Go to the parent directory:

```bash
cd ..
```

Go to the home directory:

```bash
cd ~
```

Go to a specific directory:

```bash
cd /home/kumar
```

---

# 6. cat — Display File Contents

Display a file:

```bash
cat /etc/os-release
```

Example output:

```text
NAME="Rocky Linux"
VERSION="9.3 (Blue Onyx)"
...
```

`cat` displays the contents of a file in the terminal.

---

# 7. clear — Clear the Terminal

```bash
clear
```

This clears the visible terminal screen.

---

# 8. Absolute vs Relative Paths

## Absolute path

An absolute path starts from `/`.

Example:

```bash
/home/kumar/data/file.txt
```

## Relative path

A relative path starts from the current location.

Example:

```bash
data/file.txt
```

Mental model:

```text
Absolute
/
└── home
    └── kumar
        └── data
            └── file.txt
```

Relative:

```text
Current directory
└── data
    └── file.txt
```

---

# 9. Understanding the Shell Prompt

Example root prompt:

```text
[root@lab01 ~]#
```

Breakdown:

```text
root   = current user
lab01  = hostname
~      = home directory / current location
#      = root / privileged prompt
```

Example normal-user prompt:

```text
[kumar@lab01 ~]$
```

Here:

```text
kumar  = current user
lab01  = hostname
~      = home directory / current location
$      = normal-user prompt
```

---

# 10. Connecting to a Remote Linux Server

Linux servers are commonly accessed remotely using SSH.

Syntax:

```bash
ssh username@server_ip
```

Example:

```bash
ssh kumar@192.168.234.131
```

Important requirements:

- SSH must be enabled on the server.
- Use the correct username.
- Use the correct IP address.
- Authentication must succeed.

---

# 11. PostgreSQL DBA Connection

As a PostgreSQL DBA, Linux commands are used to locate and work with PostgreSQL configuration and data.

Example PostgreSQL configuration path:

```text
/var/lib/pgsql/16/data/postgresql.conf
```

The exact location varies with:

- PostgreSQL version
- operating system
- installation method
- configuration

The important skill is being able to navigate to the location and inspect the files.

---

# 12. Day 1 Command Summary

```bash
pwd
ls
ls -l
ls -la
cd <directory>
cd ..
cd ~
cat <file>
clear
ssh username@server_ip
```

---

# 13. Useful Examples

```bash
pwd
```

```bash
ls
```

```bash
ls -l
```

```bash
ls -la
```

```bash
cd /etc
```

```bash
cd ..
```

```bash
cd ~
```

```bash
cat /etc/os-release
```

```bash
clear
```

```bash
ssh kumar@192.168.234.131
```

---

# 14. Day 1 Practice Lab

Practice the following sequence:

```bash
pwd

ls

ls -l

ls -la

cd /etc

pwd

cd ..

pwd

cd ~

pwd

cat /etc/os-release

clear
```

Then practice absolute and relative navigation:

```bash
cd /home/kumar
pwd

cd ..
pwd

cd ~
pwd
```

---

# 15. Day 1 Key Takeaways

1. Linux filesystem starts at `/`.
2. `pwd` tells you where you are.
3. `ls` shows what is in the current directory.
4. `cd` changes your location.
5. `cd ..` moves to the parent directory.
6. `cd ~` moves to your home directory.
7. `cat` displays file contents.
8. Absolute paths start from `/`.
9. Relative paths start from the current location.
10. SSH is commonly used to connect to Linux servers.
11. PostgreSQL DBAs need Linux navigation skills to work with PostgreSQL files and configuration.

---

## Day 1 Mental Model

```text
Linux Server
     |
     v
Filesystem
     |
     v
pwd → Where am I?
     |
     v
ls  → What is here?
     |
     v
cd  → Move somewhere
     |
     v
cat → Read a file
     |
     v
ssh → Connect to another server
```
