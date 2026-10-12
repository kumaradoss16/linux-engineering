Here are the common and useful Linux commands for inspecting a file such as `example.txt`. These commands are especially helpful for Linux system administrators when checking logs, configuration files, and application output.

## 1. Common file inspection commands

| Command                        | Purpose                                                       | Example use                                              |
| ------------------------------ | ------------------------------------------------------------- | -------------------------------------------------------- |
| `cat example.txt`              | Display the entire file.                                      | Read a short text file.                                  |
| `less example.txt`             | View file contents page by page.                              | Navigate large logs or configuration files.              |
| `more example.txt`             | Display file contents one screen at a time.                   | Basic page-by-page viewing.                              |
| `head example.txt`             | Display the first 10 lines by default.                        | Check the beginning of a file.                           |
| `head -n 20 example.txt`       | Display the first 20 lines.                                   | Inspect initial configuration entries.                   |
| `tail example.txt`             | Display the last 10 lines by default.                         | Check the latest log entries.                            |
| `tail -n 20 example.txt`       | Display the last 20 lines.                                    | Review recent application messages.                      |
| `tail -f example.txt`          | Follow new lines as they are appended.                        | Monitor a growing log file.                              |
| `tail -F example.txt`          | Follow a log by filename and retry if it is recreated.        | Monitor logs during log rotation.                        |
| `file example.txt`             | Identify the file type.                                       | Check whether a file is text, binary, or another format. |
| `stat example.txt`             | Display detailed file metadata.                               | Inspect size, permissions, inode, and timestamps.        |
| `wc example.txt`               | Count lines, words, and bytes.                                | Measure file contents.                                   |
| `wc -l example.txt`            | Count newline characters, generally corresponding to lines.   | Count log entries or text lines.                         |
| `nl -ba example.txt`           | Display lines with line numbers, including blank lines.       | Reference specific lines during troubleshooting.         |
| `grep "error" example.txt`     | Search for matching lines.                                    | Find errors in logs.                                     |
| `grep -in "error" example.txt` | Search case-insensitively and display line numbers.           | Find error messages with their locations.                |
| `sort example.txt`             | Sort lines alphabetically or lexically.                       | Organize text output.                                    |
| `uniq example.txt`             | Remove adjacent duplicate lines.                              | Identify repeated adjacent entries.                      |
| `diff file1.txt file2.txt`     | Compare two files.                                            | Identify changes between configuration versions.         |
| `du -h example.txt`            | Show disk space used by the file.                             | Estimate allocated storage.                              |
| `ls -lh example.txt`           | Display file permissions, ownership, and human-readable size. | Quickly inspect file attributes.                         |

## 2. How to use `less`, `head`, and `tail`

Assume `example.txt` contains server log entries.

### `less example.txt`

```
less example.txt
```

Useful keys while viewing:

| Key      | Action                     |
| -------- | -------------------------- |
| `Space`  | Move forward one screen    |
| `b`      | Move backward one screen   |
| `/error` | Search forward for `error` |
| `n`      | Find the next match        |
| `N`      | Find the previous match    |
| `g`      | Go to the beginning        |
| `G`      | Go to the end              |
| `q`      | Quit                       |

### `head example.txt`

```
head example.txt
head -n 5 example.txt
```

Displays the first 10 lines by default, or the first 5 lines with `-n 5`.

### `tail example.txt`

```
tail example.txt
tail -n 50 example.txt
tail -f example.txt
```

Displays the final 10 lines by default, the final 50 lines with `-n 50`, or follows newly appended lines with `-f`.

For system logs, you can also use:

```
sudo tail -n 50 /var/log/syslog
```

On Ubuntu systems that use the systemd journal, this is another useful option:

```
journalctl -n 50
```

## 3. Understand `file` and `stat`

### `file example.txt`

Example output:

```
example.txt: ASCII text
```

This identifies the file's content type based on file signatures and other checks. The `.txt` extension alone does not guarantee that the file contains text.

### `stat example.txt`

Example output:

```
  File: example.txt
  Size: 1250        Blocks: 8          IO Block: 4096 regular file
Device: 8,1         Inode: 123456       Links: 1
Access: (0644/-rw-r--r--)  Uid: (1000/hytrel)  Gid: (1000/hytrel)
Access: 2026-10-12 10:00:00.000000000 +0000
Modify: 2026-10-12 09:55:00.000000000 +0000
Change: 2026-10-12 09:55:00.000000000 +0000
 Birth: 2026-10-12 09:50:00.000000000 +0000
```

The values above are illustrative; actual output varies.

| Field         | Meaning                                                      |
| ------------- | ------------------------------------------------------------ |
| `Size`        | Logical file size in bytes                                   |
| `Blocks`      | Number of allocated filesystem blocks reported by the system |
| `Inode`       | Filesystem object identifier                                 |
| `Links`       | Hard-link count                                              |
| `Access`      | Last access time (`atime`)                                   |
| `Modify`      | Last content modification time (`mtime`)                     |
| `Change`      | Last inode metadata change time (`ctime`)                    |
| `Birth`       | File creation time, if supported                             |
| `Uid` / `Gid` | Numeric user and group ownership                             |
| `0644`        | Owner can read/write; group and others can read              |

Note that `ctime` is not the file creation time; it records the last change to inode metadata, including changes to permissions or ownership.

## 4. Practical command sequence

Run these commands in the directory containing your sample file:

```
less example.txt
head -n 10 example.txt
tail -n 10 example.txt
file example.txt
stat example.txt
wc -l example.txt
grep -in "error" example.txt
ls -lh example.txt
```

If the file does not exist, create a small practice file first:

```
printf 'Linux filesystem lab\nINFO: Service started\nERROR: Connection failed\nINFO: Service recovered\n' > example.txt
```

Recommendation: Learn `less`, `head`, `tail`, `file`, `stat`, `grep`, and `wc` first. Together, they cover the most common everyday tasks for examining files and troubleshooting Linux servers.
