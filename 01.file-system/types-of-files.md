# Types of Files in Linux

In Linux, almost everything is represented through a filesystem interface, including ordinary files, directories, devices, and communication endpoints. Linux distinguishes several filesystem object types, each serving a different purpose.

As a Linux system administrator, learning these types helps you investigate storage, manage permissions, troubleshoot hardware, and understand how processes communicate.

## 1. The seven common Linux file types

Linux filesystems commonly expose these seven types:

1\. Regular file (`-`)

Stores data such as text, logs, scripts, programs, images, and configuration.

Examples: `/etc/hostname`, `/var/log/syslog`, `notes.txt`

2\. Directory (`d`)

Organizes filesystem entries and maps filenames to their associated filesystem objects.

Examples: `/home`, `/etc`, `/var/log`

3\. Symbolic link (`l`)

Stores a pathname referring to another file or directory.

Examples: `/etc/localtime` on many Linux systems, `current -> releases/v1`

4\. Block device (`b`)

Represents a device that provides block-oriented storage access.

Examples: `/dev/sda`, `/dev/loop0` where present

5\. Character device (`c`)

Represents a device interface that typically handles data as a stream of bytes.

Examples: `/dev/null`, `/dev/tty`, `/dev/zero`

6\. Named pipe (FIFO) (`p`)

Provides a filesystem-named channel for communication between processes.

Example: `/tmp/my-pipe`

7\. Unix domain socket (`s`)

Provides local inter-process communication through a socket endpoint, often used by services and applications.

Examples: `/run/docker.sock`, `/run/systemd/private` where present

These symbols are the first character commonly shown by `ls -l`. Device nodes, pipes, and sockets have special behavior; they are not ordinary files that simply store document contents.

## 2. How to identify file types in Linux

Use `ls -l` to identify the filesystem object type from the first character of the permissions field.

```
ls -l /dev/null
ls -ld /etc
ls -l /etc/localtime
```

Example output:

```
crw-rw-rw- 1 root root 1, 3 Oct 12 10:00 /dev/null
drwxr-xr-x 1 root root 4096 Oct 12 10:00 /etc
lrwxrwxrwx 1 root root 27 Oct 12 10:00 /etc/localtime -> /usr/share/zoneinfo/Etc/UTC
```

| First character | Type               | Example                        |
| --------------- | ------------------ | ------------------------------ |
| `-`             | Regular file       | `notes.txt`                    |
| `d`             | Directory          | `/etc`                         |
| `l`             | Symbolic link      | `/etc/localtime`               |
| `b`             | Block device       | `/dev/sda`, if present         |
| `c`             | Character device   | `/dev/null`                    |
| `p`             | Named pipe (FIFO)  | A pipe created with `mkfifo`   |
| `s`             | Unix domain socket | `/run/docker.sock`, if present |

The examples are system-dependent; your Ubuntu Server may have different devices, sockets, and symbolic links.

## 3. Practical lab: Create and inspect file types

Run this lab in your home directory. It creates temporary objects inside a dedicated practice directory.

### Step 1 — Create a regular file and directory

```
mkdir -p ~/linux-filesystem-labs/file-types
cd ~/linux-filesystem-labs/file-types

echo "Linux file types practice" > example.txt
mkdir documents
```

### Step 2 — Create a symbolic link

```
ln -s example.txt example-link
```

Inspect the link:

```
ls -l example-link
readlink example-link
```

Expected target:

```
example.txt
```

### Step 3 — Create a named pipe

```
mkfifo message-pipe
```

Inspect its type:

```
ls -l message-pipe
file message-pipe
```

The `ls -l` permissions should begin with `p`.

A named pipe can be used for communication between processes. Reading from it may block until a writer supplies data, so do not run a blocking read casually in a terminal.

### Step 4 — Inspect device files

```
ls -l /dev/null
ls -l /dev/zero
ls -l /dev/tty
```

These are existing system device interfaces; do not create or delete them manually.

| Device      | Purpose                                                                    |
| ----------- | -------------------------------------------------------------------------- |
| `/dev/null` | Discards data written to it                                                |
| `/dev/zero` | Supplies zero-valued bytes when read                                       |
| `/dev/tty`  | Refers to the controlling terminal for the current process, when available |

For example:

```
echo "This output is discarded" > /dev/null
```

The text is sent to `/dev/null` rather than displayed or saved as a normal file.

### Step 5 — Identify file types with `file` and `stat`

```
file example.txt
file example-link
file message-pipe

stat example.txt
stat example-link
stat message-pipe
```

`file` attempts to identify the type based on file contents and metadata. `stat` reports filesystem metadata, including the object type, permissions, inode, and ownership.

To inspect the link itself rather than its target, use:

```
stat -c '%F %N' example-link
```

## 4. Important distinction: `file` versus filesystem type

The `file` command and the first character in `ls -l` answer different questions.

Consider:

```
file example.txt
ls -l example.txt
```

Possible output:

```
example.txt: ASCII text
-rw-r--r-- 1 hytrel hytrel 27 Oct 12 10:00 example.txt
```

- `file` describes the detected content type: ASCII text.
- `ls -l` identifies the filesystem object as a regular file (`-`) and displays its permissions and other metadata.

A regular file could contain text, executable machine code, a JPEG image, or binary data. Its filesystem type remains a regular file.

## 5. Quick reference for Linux administrators

| Task                                      | Command             |
| ----------------------------------------- | ------------------- |
| Identify an object's type and permissions | `ls -l PATH`        |
| Identify content or format                | `file PATH`         |
| Display metadata                          | `stat PATH`         |
| List directories only                     | `find PATH -type d` |
| List regular files only                   | `find PATH -type f` |
| Find symbolic links                       | `find PATH -type l` |
| Find named pipes                          | `find PATH -type p` |
| Find Unix sockets                         | `find PATH -type s` |
| Find block devices                        | `find PATH -type b` |
| Find character devices                    | `find PATH -type c` |

For example, list symbolic links in your lab directory:

```
find ~/linux-filesystem-labs -type l
```

Remember: Linux's seven common filesystem object types are regular files, directories, symbolic links, block devices, character devices, named pipes, and sockets. Knowing how to identify them with `ls -l`, `file`, `stat`, and `find` is a fundamental skill for filesystem administration and troubleshooting.
