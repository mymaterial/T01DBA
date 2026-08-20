# Day 3 — Linux File Operations, Text Processing & Storage

## 1. File and Directory Operations

### List files

```bash
ls
ls -l
ls -la
ls -lrt
```

- `ls` — list directory contents
- `ls -l` — long listing
- `ls -a` — include hidden files
- `ls -t` — sort by modification time
- `ls -r` — reverse order
- `ls -lrt` — long listing, oldest modified files first

### Current directory

```bash
pwd
```

### Create a file

```bash
touch file.txt
```

### Create a directory

```bash
mkdir dir1
```

### Remove an empty directory

```bash
rmdir dir1
```

### Copy a file

```bash
cp source.txt destination.txt
```

### Copy a directory recursively

```bash
cp -r source_dir destination_dir
```

### Move / rename a file

```bash
mv old.txt new.txt
mv file.txt /tmp/
```

### Remove a file

```bash
rm file.txt
```

### Remove a directory recursively

```bash
rm -r directory
```

> Be very careful with `rm` and especially `rm -r`. Deleted files may not be recoverable.

---

# 2. File Contents

### Display a file

```bash
cat file.txt
```

### Display a long file page by page

```bash
less file.txt
```

### First 10 lines

```bash
head file.txt
head -n 10 file.txt
```

### Last 10 lines

```bash
tail file.txt
tail -n 10 file.txt
```

### Follow a file continuously

```bash
tail -f application.log
```

Useful for PostgreSQL logs.

---

# 3. vi Editor

Open/create a file:

```bash
vi file.txt
```

### Important vi modes

```text
Command Mode
Insert Mode
Last Line / Command Mode
```

### Enter insert mode

```text
i
a
o
```

### Return to command mode

```text
Esc
```

### Save

```text
:w
```

### Quit

```text
:q
```

### Save and quit

```text
:wq
```

or:

```text
ZZ
```

### Quit without saving

```text
:q!
```

### Search

```text
/pattern
```

Search backward:

```text
?pattern
```

---

# 4. grep — Search Text

### Search for a pattern

```bash
grep "pattern" file.txt
```

### Case-insensitive search

```bash
grep -i "pattern" file.txt
```

### Show line numbers

```bash
grep -n "pattern" file.txt
```

### Recursive search

```bash
grep -r "pattern" /path/
```

### Invert the match

```bash
grep -v "pattern" file.txt
```

### Multiple useful options

```bash
grep -rin "error" /var/log/
```

Meaning:

```text
-r  recursive
-i  case insensitive
-n  line number
```

---

# 5. Pipes

A pipe sends the output of one command as input to another command.

Syntax:

```bash
command1 | command2
```

Example:

```bash
ps -ef | grep postgres
```

More examples:

```bash
ls -l | grep ".log"

ps -ef | grep postgres

df -h | grep "/"

du -sh * | sort -h
```

Useful concept:

```text
Command 1
   |
   | output
   v
Command 2
```

---

# 6. sort

Sort lines:

```bash
sort file.txt
```

Reverse sort:

```bash
sort -r file.txt
```

Human-readable numeric sorting:

```bash
du -sh * | sort -h
```

Reverse human-readable sorting:

```bash
du -sh * | sort -hr
```

---

# 7. uniq

Remove adjacent duplicate lines:

```bash
uniq file.txt
```

Count occurrences:

```bash
uniq -c file.txt
```

Common combination:

```bash
sort file.txt | uniq -c
```

---

# 8. wc

Count lines:

```bash
wc -l file.txt
```

Count words:

```bash
wc -w file.txt
```

Count bytes:

```bash
wc -c file.txt
```

Count lines, words and bytes:

```bash
wc file.txt
```

---

# 9. cut

Extract fields/columns.

Example:

```bash
cut -d: -f1 /etc/passwd
```

Meaning:

```text
-d:   delimiter is :
-f1   first field
```

---

# 10. Redirection

## Standard output — overwrite

```bash
command > output.log
```

Example:

```bash
ls -l > output.log
```

## Standard output — append

```bash
command >> output.log
```

Example:

```bash
date >> output.log
```

## Standard error

```bash
command 2> error.log
```

Example:

```bash
ls /no_such_directory 2> error.log
```

## Append standard error

```bash
command 2>> error.log
```

## Redirect stdout and stderr separately

```bash
command > output.log 2> error.log
```

## Redirect stdout and stderr to the same file

```bash
command > output.log 2>&1
```

Alternative Bash form:

```bash
command &> output.log
```

## Send output to /dev/null

```bash
command > /dev/null
```

Send both stdout and stderr to /dev/null:

```bash
command > /dev/null 2>&1
```

---

# 11. Disk Usage — df

### Filesystem usage

```bash
df -h
```

`-h` = human-readable.

### Inode usage

```bash
df -i
```

### Filesystem type

```bash
df -Th
```

Important distinction:

```text
df
 |
 +-- Filesystem-level usage
```

---

# 12. Disk Usage — du

### Current directory size

```bash
du -sh .
```

### Size of a file

```bash
du -sh file.dat
```

### Size of each item

```bash
du -sh *
```

### Size of directories

```bash
du -sh /var/*
```

### Find largest items

```bash
du -sh * | sort -hr
```

Example from the lab:

```bash
du -sh IO_WORKER_2_34017.dat
```

Output:

```text
2.5G    IO_WORKER_2_34017.dat
```

---

# 13. Creating Test Files with dd

### Create a 10 MiB file

```bash
dd if=/dev/zero of=abc.dat bs=1M count=10
```

Meaning:

```text
if=/dev/zero    input file
of=abc.dat      output file
bs=1M           block size
count=10        number of blocks
```

### Create a 100 MiB file

```bash
dd if=/dev/zero of=abc.dat bs=1M count=100
```

### Create a 200 MiB file

```bash
dd if=/dev/zero of=abc.dat bs=1M count=200
```

> Be careful with `dd`. Always verify the `of=` destination before running it.

---

# 14. zip / unzip

### Create a zip archive

```bash
zip archive.zip file.txt
```

Example:

```bash
zip abc.dat.zip abc.dat
```

### Zip multiple files

```bash
zip archive.zip file1 file2 file3
```

### Zip a directory

```bash
zip -r archive.zip directory/
```

### Extract a zip archive

```bash
unzip archive.zip
```

### Important

A normal `.dat` file is not automatically a ZIP archive.

This is incorrect:

```bash
zip IO_WORKER_2_34017.dat
```

If the source is a normal data file, specify the archive name first:

```bash
zip IO_WORKER_2_34017.dat.zip IO_WORKER_2_34017.dat
```

---

# 15. gzip / gunzip

### Compress a file

```bash
gzip abc.dat
```

This normally produces:

```text
abc.dat.gz
```

and removes the original `abc.dat`.

### Decompress

```bash
gunzip abc.dat.gz
```

This restores:

```text
abc.dat
```

Check the result:

```bash
ls -lh
du -sh *
```

---

# 16. tar — Archiving

### Create a tar archive

```bash
tar -cvf abc.tar abc1.dat abc2.dat abc3.dat
```

Options:

```text
c = create
v = verbose
f = archive file
```

### List archive contents

```bash
tar -tvf abc.tar
```

### Extract archive

```bash
tar -xvf abc.tar
```

Options:

```text
x = extract
v = verbose
f = archive file
```

---

# 17. tar + gzip

Create:

```bash
tar -czvf archive.tar.gz directory/
```

Extract:

```bash
tar -xzvf archive.tar.gz
```

Options:

```text
c = create
x = extract
z = gzip
v = verbose
f = file
```

---

# 18. tar + bzip2

Create:

```bash
tar -cjvf archive.tar.bz2 directory/
```

Extract:

```bash
tar -xjvf archive.tar.bz2
```

---

# 19. File Links

### Hard link

```bash
ln source.txt hardlink.txt
```

### Symbolic link

```bash
ln -s source.txt symlink.txt
```

Check:

```bash
ls -li
```

---

# 20. scp — Secure Copy

### Copy local file to remote server

```bash
scp abc.tar 192.168.234.133:/root
```

### Copy to a specific user

```bash
scp abc.tar user@192.168.234.133:/home/user/
```

### Copy from remote server

```bash
scp user@192.168.234.133:/root/abc.tar .
```

### Copy a directory recursively

```bash
scp -r directory user@192.168.234.133:/root/
```

---

# 21. ssh

Connect to a remote server:

```bash
ssh 192.168.234.133
```

Connect as a specific user:

```bash
ssh user@192.168.234.133
```

Exit:

```bash
exit
```

---

# 22. Practical DBA Examples

## Find PostgreSQL processes

```bash
ps -ef | grep postgres
```

## Search PostgreSQL logs

```bash
grep -i "error" postgresql.log
```

## Follow PostgreSQL logs

```bash
tail -f postgresql.log
```

## Find large files

```bash
du -ah /var/lib/pgsql | sort -hr | head
```

## Check filesystem space

```bash
df -h
```

## Check inode exhaustion

```bash
df -i
```

## Find large files

```bash
find /var/lib/pgsql -type f -size +1G -ls
```

## Find recent PostgreSQL files

```bash
find /var/lib/pgsql -type f -mtime -1
```

---

# 23. Important PostgreSQL DBA Connections

Linux file skills are directly useful for:

```text
PostgreSQL data directory
WAL files
PostgreSQL logs
Backup files
Configuration files
Archive directories
Temporary files
Mount points
Disk capacity
Inode exhaustion
Backup transfer
Compression
Automation
```

Typical investigation:

```text
PostgreSQL problem
       |
       v
df -h
       |
       v
Which filesystem is full?
       |
       v
du -sh *
       |
       v
Which directory is consuming space?
       |
       v
find
       |
       v
Which files are large/old?
```

---

# 24. Commands Practiced in Day 3

```text
ls
pwd
cd
touch
mkdir
rmdir
cp
mv
rm
cat
less
head
tail
vi
grep
sort
uniq
wc
cut
|
>
>>
2>
2>>
2>&1
/dev/null
df
du
dd
zip
unzip
gzip
gunzip
tar
ln
scp
ssh
find
```

---

# 25. Day 3 Key Takeaways

1. `ls -lrt` helps inspect files by modification time.
2. `cat`, `less`, `head`, and `tail` help inspect files.
3. `vi` is an important Linux editor for DBAs.
4. `grep` searches text efficiently.
5. Pipes connect commands together.
6. Redirection separates or combines command output and errors.
7. `df` shows filesystem usage.
8. `du` shows file/directory usage.
9. `dd` is useful for controlled lab file generation but must be used carefully.
10. `zip`/`gzip` compress data; `tar` primarily creates archives.
11. `scp` transfers files between Linux servers.
12. These skills are directly useful for PostgreSQL logs, WAL, backups, storage and troubleshooting.

---

## Day 3 DBA Lab

Use a lab VM and practice:

```bash
mkdir ~/day3_lab
cd ~/day3_lab

touch file1.txt file2.txt

echo "PostgreSQL Linux DBA" > file1.txt
echo "Performance tuning" >> file1.txt

cat file1.txt

grep "PostgreSQL" file1.txt

cp file1.txt file2.txt

mv file2.txt file3.txt

ls -lrt

du -sh *

df -h

dd if=/dev/zero of=test.dat bs=1M count=100

du -sh test.dat

zip test.dat.zip test.dat

tar -cvf test.tar test.dat

ls -lh

rm -f test.dat test.dat.zip test.tar file1.txt file3.txt

cd ..
rmdir ~/day3_lab
```

> Run destructive commands only inside your dedicated lab directory. Never experiment with `rm`, `dd`, or archive commands against a production PostgreSQL data directory.
