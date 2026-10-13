For Linux system administration, learn `df` to check filesystem capacity and free space, and `du` to identify how much disk space files and directories consume. Both are essential when troubleshooting a full Ubuntu Server or Linux system.

## 1. Essential `df` commands

`df` stands for disk free. It reports space usage for mounted filesystems.

| Command          | Purpose                                                         |
| ---------------- | --------------------------------------------------------------- |
| `df -h`          | Show filesystem space in human-readable units (KB, MB, GB).     |
| `df -H`          | Human-readable units using powers of 1000.                      |
| `df -T`          | Include filesystem type.                                        |
| `df -Th`         | Show filesystem type and human-readable sizes.                  |
| `df -i`          | Show inode usage instead of data-block usage.                   |
| `df -ih`         | Show inode usage in human-readable form.                        |
| `df -h /`        | Check the filesystem containing the root directory.             |
| `df -h /home`    | Check the filesystem containing `/home`.                        |
| `df -h /boot`    | Check the filesystem containing `/boot`, if separately mounted. |
| `df -x tmpfs -h` | Exclude `tmpfs` filesystems from the report.                    |

### Example: Check disk space

```
df -h
```

Illustrative output:

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        50G   35G   13G  74% /
tmpfs           1.0G  1.2M  1.0G   1% /run
```

- `Size`: total filesystem capacity.
- `Used`: space currently used.
- `Avail`: available space for ordinary users; reserved blocks can affect this.
- `Use%`: percentage used.
- `Mounted on`: directory where the filesystem is accessible.

Administrator tip: If `df -h /` shows the root filesystem is nearly full, use `du` to investigate which directories are consuming space.

## 2. Essential `du` commands

`du` stands for disk usage. It estimates the space occupied by files and directories.

| Command                   | Purpose                                                                      |
| ------------------------- | ---------------------------------------------------------------------------- |
| `du -sh DIR`              | Show the total size of a directory.                                          |
| `du -h DIR`               | Show sizes of a directory and its contents at multiple levels.               |
| `du -sh *`                | Show the total size of each visible entry in the current directory.          |
| `du -sh ./*`              | Show the size of each visible entry using explicit relative paths.           |
| `du -sh .*`               | Include dot-prefixed entries; be careful because `.` and `..` can match too. |
| `du -h --max-depth=1 DIR` | Show immediate subdirectory sizes and the directory total.                   |
| `du -ah DIR`              | Show individual file and directory sizes.                                    |
| `du -sh /var/log`         | Check total disk usage under `/var/log`.                                     |
| `du -sh /home/*`          | Compare the sizes of users' home directories.                                |
| `du -xhd1 /`              | Summarize root-level directory usage without crossing filesystem boundaries. |
| `du -sh -- * \\| sort -h` | Sort visible entries by size, smallest first.                                |

### Example: Find large directories

```
sudo du -xhd1 / 2>/dev/null | sort -h
```

What the options mean:

- `sudo`: allows access to directories ordinary users may not be able to read.
- `du`: calculates disk usage.
- `-x`: stay on the same filesystem as `/`.
- `-h`: use human-readable sizes.
- `-d1`: show only one directory level beneath the starting path.
- `2>/dev/null`: discard permission-denied and other error messages.
- `sort -h`: sort sizes numerically with human-readable units.

This is a useful first investigation when your root filesystem is running out of space. Note that suppressing errors can hide directories that were not inspected.

## 3. `df` vs `du`

| Feature                | `df`                         | `du`                                                      |
| ---------------------- | ---------------------------- | --------------------------------------------------------- |
| Main purpose           | Filesystem capacity          | Space used by files and directories                       |
| Answers                | How full is this filesystem? | Which directories use the space?                          |
| Typical command        | `df -h`                      | `sudo du -xhd1 /`                                         |
| Inode check            | `df -i`                      | No direct equivalent for total filesystem inode capacity  |
| Deleted-but-open files | May still count their space  | Usually won't count the unlinked file by its old pathname |

The last row is particularly important for server troubleshooting: a process may still hold a deleted log file open, causing `df` to report more used space than `du` can account for. Investigate that case with:

```
sudo lsof +L1
```

This lists open files that have been deleted or unlinked but are still held open by processes, when `lsof` is installed.

## 4. Practical disk troubleshooting workflow

Use this sequence when a Linux server reports low disk space.

Step 1 — Check filesystem capacity

```
df -hT
```

Identify which filesystem is nearly full and its type.

Step 2 — Identify large directories

```
sudo du -xhd1 /
```

Find the major consumers on the root filesystem.

Step 3 — Investigate the relevant directory

For example:

```
sudo du -h --max-depth=1 /var
sudo du -h --max-depth=1 /var/log
```

Narrow down the location before deciding what needs attention.

Step 4 — Check inode exhaustion if necessary

```
df -ih
```

A filesystem can run out of inodes even when it still has free data space, often because it contains huge numbers of small files.

Important: `du` can take time on large filesystems. Start with a shallow scan rather than immediately running a full recursive report. Avoid deleting system files just because they appear large; identify their purpose first.
