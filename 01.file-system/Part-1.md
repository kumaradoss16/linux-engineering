# Chapter 3: Linux Filesystem

## Chapter assumptions

The prompt left the input fields blank. This chapter uses these values. Change them if your setup differs.

| Input | Value used |
|---|---|
| Audience | Junior sysadmins and aspiring security analysts, comfortable typing commands |
| Lab environment | Ubuntu 24.04 Server VM, normal user `student` with sudo, one optional second virtual disk (used from Part 3) |
| Filesystem type | ext4 |
| Length | About 7,000 words across five parts |
| Output format | Markdown |

Run every command in a throwaway VM. Create `~/fs-lab` and keep all test files there. Take a VM snapshot before Part 3 and before Part 4. Output blocks are sample output. Your inode numbers, dates, sizes and device names will differ.

Command classes used in this chapter:

- **Read-only**: changes nothing.
- **Changes practice data**: changes files inside `~/fs-lab` only. An undo step is given.
- **Needs sudo, changes system**: changes the system. An undo step is given.

---

## The vital few

The 80/20 rule here means selecting commands by how often they solve real problems: full disks, missing files, permission failures, broken links and unexplained processes. The commands below cover most daily work and most troubleshooting. Each has one line of justification. Commands not listed go in the "Learn later" table in Part 5.

### The 24 commands

| # | Command | Problem it solves | Taught in |
|---|---|---|---|
| 1 | `pwd`, `cd` | Know where you are and move; path mistakes cause most beginner errors | 3.1 |
| 2 | `ls -lah` | Shows type, permissions, owner, size and time at once | 3.1 |
| 3 | `stat` | Shows the full inode record, including all timestamps and link count | 3.1 |
| 4 | `file` | Identifies content by inspection, not by name | 3.1 |
| 5 | `df -h` | First check for a full filesystem | 3.2 |
| 6 | `df -i` | Detects inode exhaustion, the cause of "No space left" with free space | 3.2 |
| 7 | `du -sh`, `du -xh --max-depth=1` | Finds which directory consumes space | 3.2 |
| 8 | `find` | Locates files by name, type, age, size, owner and permission | 3.2 |
| 9 | `ln` | Creates hard links | 3.3 |
| 10 | `ln -s` | Creates symbolic links, the basis of many deployment layouts | 3.3 |
| 11 | `readlink -f` | Resolves a link to its final target, and finds broken links | 3.3 |
| 12 | `mount` | Attaches a filesystem to the tree | 3.4 |
| 13 | `findmnt` | Shows the mount tree clearly and verifies fstab | 3.4 |
| 14 | `lsblk` | Confirms device names before any destructive command | 3.4 |
| 15 | `blkid` | Shows filesystem UUIDs for stable fstab entries | 3.4 |
| 16 | `lsof` | Finds deleted files still held open and what uses a path | 3.5 |
| 17 | `ps` | Maps a process to its PID for `/proc/<PID>` inspection | 3.5 |
| 18 | `chmod` | Fixes permission failures | 3.5 |
| 19 | `chown` | Fixes ownership failures | 3.5 |
| 20 | `umask` | Explains why new files get the permissions they get | 3.5 |
| 21 | `getfacl`, `setfacl` | Grants access to specific users without changing ownership | Part 3 |
| 22 | `tar`, `rsync` | Backups that preserve metadata | Part 3 |
| 23 | `journalctl`, `dmesg` | Reads filesystem errors and mount failures | Part 3 |
| 24 | `cat /proc/<PID>/...` | Inspects process state without extra tools | 3.4 |

### The 9 concepts

1. **Inode**: the record that stores a file's metadata and the location of its data.
2. **Directory entry**: a name mapped to an inode number.
3. **Path resolution**: the kernel walks a path one component at a time.
4. **Link count**: how many directory entries point to an inode.
5. **Mount point**: a directory where a filesystem attaches to the tree.
6. **Permission bits and ownership**: the owner, group and other triplets that gate access.
7. **Space versus inodes**: a filesystem can run out of either one.
8. **Virtual filesystem**: files that present kernel state, not disk data.
9. **Timestamps**: mtime, ctime and atime each answer a different question.

Part 1 teaches commands 1 to 8 and concepts 1 to 4 and 7 in full.

### Two-hour path

The full path through all core exercises is assembled in Part 5. Part 1 contributes these Core items, about 45 minutes: Exercises 3.1.1 to 3.1.4, 3.2.1 to 3.2.4.

---

## 3.1 Filesystem fundamentals

**Level: Beginner**

> **The 20% that matters**
> - A file name is not the file. The name lives in a directory entry. The metadata lives in an inode.
> - Every path is either absolute (starts with `/`) or relative (starts anywhere else).
> - `ls -lah` and `stat` show nearly everything you need about a file.
> - The first character of `ls -l` output is the file type.
> - A process reaches open files through numbered file descriptors.

### Core ideas

A **file** is a named sequence of bytes. A **directory** is a special file that maps names to inode numbers. A **path** is a string that names a location. An **absolute path** starts at `/`, the root of the tree. A **relative path** starts from the current working directory. The shorthand `.` means the current directory, `..` means the parent, and `~` means your home directory.

An **inode** (index node) is a data structure on disk. It stores the file type, permissions, owner, group, size, timestamps, link count and pointers to data blocks. It does not store the file name. A **directory entry** pairs a name with an inode number. This design exists so that one file can have several names (links) and so that renaming a file does not move its data.

**Metadata** is data about a file: everything in the inode.

A **file descriptor** is a small integer a process uses to refer to an open file. Descriptor 0 is standard input, 1 is standard output and 2 is standard error. Descriptors exist so the kernel can track the position, mode and target of every open file per process.

```
Directory /home/student/fs-lab
+------------------+---------+
| name             | inode   |
+------------------+---------+
| notes.txt        | 1048722 | ---> Inode 1048722
| project          | 1048730 |      type, mode, owner, group,
+------------------+---------+      size, timestamps, link count,
                                    block pointers
                                         |
                                         v
                                    Data blocks on disk
                                    ("hello filesystem\n")
```

Path resolution for `/home/student/fs-lab/notes.txt`: the kernel starts at the root inode, finds `home` in the root directory, loads that inode, finds `student`, and so on until it reaches `notes.txt`. Each step needs execute (search) permission on the directory.

### File type characters

| Char | Type | Example |
|---|---|---|
| `-` | Regular file | `/etc/hostname` |
| `d` | Directory | `/etc` |
| `l` | Symbolic link | `/bin` on a merged-usr system |
| `c` | Character device | `/dev/null` |
| `b` | Block device | `/dev/sda` |
| `p` | Named pipe (FIFO) | Created with `mkfifo` |
| `s` | Socket | `/run/dbus/system_bus_socket` (verify on your system) |

### Lab setup

Class: changes practice data. Undo: `rm -r ~/fs-lab`.

```bash
mkdir -p ~/fs-lab/project/src ~/fs-lab/project/logs
cd ~/fs-lab
printf 'hello filesystem\n' > notes.txt
printf 'port=8080\n' > project/src/app.conf
printf 'start\n' > project/logs/app.log
```

### Command: `ls -lah`

```bash
ls -lah ~/fs-lab
```

**Purpose:** List directory contents with full metadata.

**Flags:**
- `-l` long format: type, permissions, link count, owner, group, size, modification time, name.
- `-a` include hidden entries, including `.` and `..`.
- `-h` print sizes in human-readable units.

**Sample output:**

```
total 16K
drwxr-xr-x  3 student student 4.0K Oct 13 09:10 .
drwxr-x--- 12 student student 4.0K Oct 13 09:08 ..
-rw-r--r--  1 student student   17 Oct 13 09:10 notes.txt
drwxr-xr-x  4 student student 4.0K Oct 13 09:10 project
```

**Reading the output:** The first character is the type. The next nine are three permission triplets (owner, group, other). The number after them is the link count. `project` shows 4: its own entry, its `.` entry, and the `..` entry in each of `src` and `logs`. The size of a directory is the size of its entry list, not of its contents.

**Use case:** At the start of any investigation, list the target directory to see who owns what and which files changed recently.

**Class:** Read-only.

Add `-i` to show inode numbers:

```bash
ls -li ~/fs-lab/notes.txt
```

```
1048722 -rw-r--r-- 1 student student 17 Oct 13 09:10 /home/student/fs-lab/notes.txt
```

### Command: `stat`

```bash
stat ~/fs-lab/notes.txt
```

**Purpose:** Print the complete inode record of a file.

**Flags:**
- `-c FORMAT` print selected fields, for example `stat -c '%i %h %U %a %n' file`.
- `-f` report on the filesystem that holds the file, not the file itself.

**Sample output:**

```
  File: /home/student/fs-lab/notes.txt
  Size: 17        	Blocks: 8          IO Block: 4096   regular file
Device: 8,2	Inode: 1048722     Links: 1
Access: (0644/-rw-r--r--)  Uid: ( 1000/ student)   Gid: ( 1000/ student)
Access: 2026-10-13 09:10:41.123456789 +0000
Modify: 2026-10-13 09:10:41.123456789 +0000
Change: 2026-10-13 09:10:41.123456789 +0000
 Birth: 2026-10-13 09:10:41.123456789 +0000
```

**Reading the output:** `Size` is bytes of content. `Blocks` counts 512-byte units allocated, so 8 blocks is 4096 bytes on disk for 17 bytes of content. `Inode` and `Links` match `ls -li`. `Access` is atime, `Modify` is mtime (content changed), `Change` is ctime (inode changed, for example by `chmod` or rename). `Birth` is creation time and depends on the filesystem and kernel support (verify on your system).

**Use case:** Confirm whether two names are the same file by comparing inode numbers, and establish when a file's content or permissions last changed.

**Class:** Read-only.

### Command: `file`

```bash
file ~/fs-lab/notes.txt /bin /etc/passwd /dev/null
```

**Purpose:** Identify what a file contains by examining its contents and type, not its name.

**Flags:**
- `-b` omit the file name from output.
- `-i` print the MIME type.
- `-L` follow symbolic links.

**Sample output:**

```
/home/student/fs-lab/notes.txt: ASCII text
/bin:                           symbolic link to usr/bin
/etc/passwd:                    ASCII text
/dev/null:                      character special (1/3)
```

**Reading the output:** `symbolic link to usr/bin` shows that `/bin` is a link. `character special (1/3)` shows major and minor device numbers.

**Use case:** A file named `invoice.pdf` found in `/tmp` that `file` reports as an executable is suspicious.

**Class:** Read-only.

### Commands: `pwd`, `cd`, `realpath`

```bash
cd ~/fs-lab/project/src
pwd
cd ../logs
pwd
realpath ../src/app.conf
cd -
```

**Purpose:** Show and change the working directory, and resolve a relative path to an absolute one.

**Flags and forms:** `cd ..` goes up one level. `cd -` returns to the previous directory. `cd` alone goes to your home directory.

**Sample output:**

```
/home/student/fs-lab/project/src
/home/student/fs-lab/project/logs
/home/student/fs-lab/project/src/app.conf
/home/student/fs-lab/project/src
```

**Reading the output:** `realpath` expands `..` and prints the absolute path. The last line is the output of `cd -`, which prints the directory it returns to.

**Use case:** Scripts run from cron have an unpredictable working directory. Use absolute paths in them.

**Class:** Read-only.

### Exercises

**Exercise 3.1.1 (Core).** Create the lab files with the setup commands. Run `ls -lah ~/fs-lab` and identify the type, owner and link count of each entry.

**Exercise 3.1.2 (Core).** Run `stat ~/fs-lab/notes.txt`. Write down the inode number. Run `ls -li` on the same file and confirm the numbers match.

**Exercise 3.1.3 (Core).** Run `file` on `notes.txt`, `/bin`, `/etc/passwd` and `/dev/null`. For each, also read the first character of `ls -ld`.

**Exercise 3.1.4 (Core).** From `~/fs-lab/project/src`, reach `app.log` using a relative path, then using an absolute path. Confirm both with `cat`.

**Exercise 3.1.5 (Extension).** Show your own file descriptors. Class: read-only.

```bash
ls -l /proc/self/fd
```

```
total 0
lrwx------ 1 student student 64 Oct 13 09:30 0 -> /dev/pts/0
lrwx------ 1 student student 64 Oct 13 09:30 1 -> /dev/pts/0
lrwx------ 1 student student 64 Oct 13 09:30 2 -> /dev/pts/0
lr-x------ 1 student student 64 Oct 13 09:30 3 -> /proc/2291/fd
```

`/proc/self` refers to the process reading it, here `ls`. Descriptors 0, 1 and 2 point to your terminal. Descriptor 3 is the directory `ls` is reading. The terminal name (`pts/0`) and PID differ on your system.

**Exercise 3.1.6 (Extension).** Create a named pipe and check its type. Class: changes practice data. Undo: `rm ~/fs-lab/demo.fifo`.

```bash
mkfifo ~/fs-lab/demo.fifo
ls -l ~/fs-lab/demo.fifo
```

```
prw-r--r-- 1 student student 0 Oct 13 09:35 /home/student/fs-lab/demo.fifo
```

The leading `p` marks a pipe.

### Common mistakes

1. Treating the file name as part of the file. Renaming changes a directory entry, not the inode.
2. Using relative paths in scripts and cron jobs, where the working directory is not what you expect.
3. Reading the `Size` field as disk usage. Use `du` for allocated space.

---

## 3.2 Filesystem hierarchy

**Level: Beginner (hierarchy), Intermediate (find, du, df)**

> **The 20% that matters**
> - There is one tree rooted at `/`. Different disks attach into it at mount points.
> - `/etc` holds configuration, `/var` holds changing data, `/usr` holds programs, `/home` holds user data, `/tmp` holds temporary files.
> - On modern distributions, `/bin`, `/sbin` and `/lib` are symbolic links into `/usr`. Verify with `ls -l /`.
> - `/var` is where disks most often fill up.
> - `df` shows filesystem capacity. `du` shows what consumes it. They answer different questions.

### Why the hierarchy exists

The Filesystem Hierarchy Standard (FHS) defines where each kind of file belongs so that administrators, packages and scripts agree on locations. Separating static programs (`/usr`), variable data (`/var`), configuration (`/etc`) and user data (`/home`) lets you mount, back up, secure and size each area differently.

### Verify the merged `/usr` layout

On modern distributions, `/bin`, `/sbin`, `/lib` and usually `/lib64` are symbolic links to directories under `/usr`. This is called merged-usr. It varies by distribution and version, so verify on your system.

```bash
ls -l /
```

**Sample output (selected lines):**

```
lrwxrwxrwx   1 root root    7 Apr 22  2024 bin -> usr/bin
drwxr-xr-x   4 root root 4096 Oct 13 08:55 boot
drwxr-xr-x  19 root root 4180 Oct 13 08:55 dev
drwxr-xr-x  96 root root 4096 Oct 13 08:55 etc
drwxr-xr-x   3 root root 4096 Oct  1 10:12 home
lrwxrwxrwx   1 root root    7 Apr 22  2024 lib -> usr/lib
lrwxrwxrwx   1 root root    9 Apr 22  2024 lib64 -> usr/lib64
lrwxrwxrwx   1 root root    8 Apr 22  2024 sbin -> usr/sbin
drwxrwxrwt  12 root root 4096 Oct 13 09:40 tmp
drwxr-xr-x  12 root root 4096 Apr 22  2024 usr
drwxr-xr-x  13 root root 4096 Oct  1 10:12 var
```

**Reading the output:** A leading `l` and an arrow show a link. The `t` at the end of the `/tmp` permissions is the sticky bit, covered in 3.5.

### Hierarchy table

| Directory | Purpose | Example contents | Inspect with | Admin concern |
|---|---|---|---|---|
| `/` | Root of the tree | All other directories | `ls -l /` | Keep the root filesystem from filling |
| `/boot` | Boot files | Kernel images, initramfs, bootloader files | `ls -lh /boot` | Old kernels fill a small `/boot` |
| `/dev` | Device files | `null`, `sda`, `tty` | `ls -l /dev` | Wrong device name in a command destroys data |
| `/etc` | System configuration | `fstab`, `passwd`, `ssh/` | `ls /etc` | Back up before editing; track changes |
| `/home` | User home directories | `student/` | `du -sh /home/*` | Quotas, separate mount, privacy |
| `/lib` | Shared libraries, kernel modules | Link to `usr/lib` on merged systems | `ls -l /lib` | Missing libraries break programs |
| `/media` | Auto-mounted removable media | USB drives | `ls /media` | Desktop automount behavior |
| `/mnt` | Manual temporary mounts | Practice disk | `ls /mnt` | Stale mounts left behind |
| `/opt` | Add-on software | Vendor packages | `ls /opt` | Not tracked by the package manager |
| `/proc` | Process and kernel view (virtual) | `cpuinfo`, `meminfo`, `<PID>/` | `ls /proc` | Exposes process details |
| `/root` | Home of the root user | Root's shell files | `sudo ls -la /root` | Must stay mode 700 |
| `/run` | Runtime state (virtual, memory-backed) | PID files, sockets | `ls /run` | Cleared at boot |
| `/sbin` | System administration programs | Link to `usr/sbin` on merged systems | `ls -l /sbin` | Check `PATH` for non-root users |
| `/srv` | Data served by this system | Web or FTP content | `ls /srv` | Often empty; choose a layout and document it |
| `/sys` | Kernel device and driver view (virtual) | `class/`, `block/` | `ls /sys/class` | Writes can change hardware behavior |
| `/tmp` | Temporary files | Session and scratch files | `ls -ld /tmp` | World-writable; hardened with mount options |
| `/usr` | Programs, libraries, documentation | `bin/`, `lib/`, `share/` | `du -sh /usr` | Merged-usr links point here |
| `/var` | Variable data | `log/`, `lib/`, `cache/`, `spool/` | `du -xh --max-depth=1 /var` | Most common place for disk exhaustion |

Cleanup of `/tmp` at boot varies by distribution. Some clear it with systemd-tmpfiles, others mount it as tmpfs. Verify on your system.

### Command: `df`

```bash
df -h
df -i
```

**Purpose:** Report capacity of mounted filesystems, in blocks (`-h`) or inodes (`-i`).

**Flags:**
- `-h` human-readable sizes.
- `-i` inode counts instead of blocks.
- `-T` add the filesystem type column.
- `-x tmpfs` exclude a filesystem type from the list.

**Sample output (`df -h /`):**

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        20G  6.1G   13G  32% /
```

**Sample output (`df -i /`):**

```
Filesystem      Inodes  IUsed   IFree IUse% Mounted on
/dev/sda2      1310720 182340 1128380   14% /
```

**Reading the output:** `Use%` is block usage. `IUse%` is inode usage. ext4 reserves some blocks for root, so `Used` plus `Avail` is less than `Size`. A filesystem is full when either percentage reaches 100.

**Use case:** "No space left on device" with free blocks usually means `IUse%` is 100, caused by millions of small files.

**Class:** Read-only.

### Command: `du`

```bash
du -sh ~/fs-lab
sudo du -xh --max-depth=1 /var | sort -h
```

**Purpose:** Measure the space consumed by files and directories.

**Flags:**
- `-s` print one total per argument.
- `-h` human-readable sizes.
- `-x` stay on one filesystem, so mounted disks and virtual filesystems are not counted.
- `--max-depth=1` report only one level of subdirectories.

**Sample output:**

```
20K	/home/student/fs-lab
```

```
8.0K	/var/spool
36M	/var/cache
118M	/var/log
1.2G	/var/lib
1.4G	/var
```

**Reading the output:** `sort -h` orders human-readable sizes, so the largest entry is last. The total is the final line. Repeat the command inside the largest subdirectory to descend.

**Use case:** After `df` shows a full filesystem, `du -xh --max-depth=1` at `/`, then `/var`, finds the consumer in a few steps.

**Class:** Read-only. `sudo` gives complete numbers because unprivileged users cannot read every directory. It changes nothing.

### Command: `find`

```bash
find ~/fs-lab -type f
find ~/fs-lab -type f -name '*.log'
find /var/log -type f -size +10M
find ~/fs-lab -type f -mtime -1
find ~/fs-lab -type f -user student -perm 644
```

**Purpose:** Search a directory tree for files matching conditions.

**Flags:**
- `-type f|d|l` match regular files, directories or symbolic links.
- `-name 'pattern'` match the name against a pattern. Quote the pattern so the shell does not expand it.
- `-size +10M` match files larger than 10 MiB. Sizes round up to the unit, so test with care.
- `-mtime -1` match files modified less than one day ago. `-mtime +7` matches files modified eight or more days ago, because fractions are discarded.
- `-user NAME` match files owned by a user.
- `-perm 644` match exactly mode 644. `-perm -644` requires all listed bits. `-perm /644` requires any listed bit.

**Sample output (first command):**

```
/home/student/fs-lab/notes.txt
/home/student/fs-lab/project/src/app.conf
/home/student/fs-lab/project/logs/app.log
```

**Reading the output:** Each line is one match. With no match, `find` prints nothing and exits successfully, so empty output means "no files matched", not an error.

**Use case:** Find the largest recent log files during a disk-full incident:

```bash
sudo find /var/log -type f -size +100M -mtime -2
```

**Class:** Read-only. `find` with `-delete` or `-exec` can change data. Do not add them until you have run the same command without them and reviewed the list.

### Exercises

**Exercise 3.2.1 (Core).** Run `ls -l /`. List which of `/bin`, `/sbin`, `/lib` and `/lib64` are symbolic links on your VM and where each points. Record the distribution and version with `cat /etc/os-release`.

**Exercise 3.2.2 (Core).** Run `df -h` and `df -i`. Identify the filesystem that holds `/`. Compute free space and free inodes.

**Exercise 3.2.3 (Core).** Run `du -sh ~/fs-lab`. Then run `sudo du -xh --max-depth=1 /var | sort -h`. Name the largest subdirectory of `/var`.

**Exercise 3.2.4 (Core).** Run the five `find` commands above. For each, write one sentence on why it matched or did not match. Then create a second file with `touch -d '10 days ago' ~/fs-lab/old.txt` and run `find ~/fs-lab -type f -mtime +7`. Class: changes practice data. Undo: `rm ~/fs-lab/old.txt`.

**Exercise 3.2.5 (Extension).** Run `findmnt /` and `df -T /`. Confirm the filesystem type is `ext4`.

```
TARGET SOURCE    FSTYPE OPTIONS
/      /dev/sda2 ext4   rw,relatime
```

**Exercise 3.2.6 (Extension).** Create 2,000 empty files and watch `df -i` change. Class: changes practice data. Undo: `rm -r ~/fs-lab/many`.

```bash
mkdir ~/fs-lab/many
cd ~/fs-lab/many && seq 1 2000 | xargs touch && cd ~
df -i ~/fs-lab
```

`IUsed` rises by about 2,000 while `Used` blocks barely change. This is the mechanism behind inode exhaustion, which is covered in Part 3.

### Common mistakes

1. Running `du` without `-x` on `/`. It counts `/proc`, other mounts and virtual files, which gives misleading totals and slow output.
2. Reading `df` and `du` as the same measurement. `df` asks the filesystem, `du` walks directories. They disagree when deleted files are still open (Part 3).
3. Using `find` without quoting patterns. `find . -name *.log` lets the shell expand `*.log` first and can fail or return wrong results.

---

End of Part 1. Reply "continue" for Part 2: 3.3, 3.4, 3.5 and Labs 3.1 to 3.3.
