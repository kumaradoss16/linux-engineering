# Chapter 4: Linux Command Line

## Part 1: Vital Few, Bash Basics, Command Syntax, Environment, History, Aliases, and Command Substitution

> **Lab safety**
>
> Run all practice commands only in a disposable VM or lab environment that you own or are authorized to use. Use `~/cli-lab` for practice files. Do not run destructive commands against `/etc`, `/var/log`, `/usr`, or `/`. Before every use of `rm`, especially with wildcards or recursive options, run `pwd` and `ls -lah` first.

## Lab setup

Run these commands first:

```bash
mkdir -p ~/cli-lab
cd ~/cli-lab
```

- `mkdir` creates a directory.
- `-p` creates missing parent directories and does not fail if `~/cli-lab` already exists.
- `~` expands to your home directory, usually `/home/your-user-name`.
- `cd` changes Bash’s current working directory.

Verify the location:

```bash
pwd
ls -lah
```

**Sample output:**

```text
/home/student/cli-lab
total 8.0K
drwxr-xr-x 2 student student 4.0K Oct 15 09:25 .
drwxr-xr-x 8 student student 4.0K Oct 15 09:25 ..
```

Read `pwd` as the full path where future lab files will be created. The `.` entry means the current directory. The `..` entry means the parent directory.

***

# The vital few

**Level: Beginner**

> ## The 20% that matters
>
> Master these first:
>
> - Use `pwd`, `cd`, and `ls -lah` to know where you are and what exists.
> - Use `cat`, `less`, `head`, and `tail` to inspect text safely.
> - Use `grep`, `find`, `sort`, `uniq`, `cut`, and `wc` to search and summarize data.
> - Use quotes around variable values and filenames that may contain spaces.
> - Check `$?` after an important command or use `&&` and `||` to control safe automation.

Most daily Linux command-line work comes from a small set of commands and shell rules. Learn these well before moving to less common tools.

## Vital commands

| Command | Primary use | Safety classification |
|---|---|---|
| `pwd` | Show current directory | Read-only |
| `cd` | Change current directory | Changes shell state only |
| `ls` | List files | Read-only |
| `cp` | Copy files | Changes data in practice area |
| `mv` | Move or rename files | Changes data in practice area |
| `rm` | Remove files | Changes data in practice area |
| `mkdir` | Create directories | Changes data in practice area |
| `touch` | Create files or update timestamps | Changes data in practice area |
| `cat` | Print file contents | Read-only |
| `less` | Read files one page at a time | Read-only |
| `head` | Show beginning of file | Read-only |
| `tail` | Show end of file | Read-only |
| `grep` | Search text | Read-only |
| `find` | Search filesystem paths | Read-only unless paired with an action |
| `sort` | Sort text lines | Read-only |
| `uniq` | Remove or count adjacent duplicate lines | Read-only |
| `cut` | Extract delimited fields | Read-only |
| `tr` | Translate or delete characters | Read-only |
| `wc` | Count lines, words, and bytes | Read-only |
| `xargs` | Build command arguments from input | Depends on command it runs |
| `file` | Identify file content type | Read-only |
| `stat` | Show metadata | Read-only |
| `history` | Show command history | Read-only |
| `alias` | Create or show shortcuts | Changes shell state only |
| `env` | Show environment variables | Read-only |
| `echo` | Print text or variables | Read-only |
| `which` | Show executable found through `PATH` | Read-only |
| `man` | Read manual pages | Read-only |

Bash builtins include commands such as `cd`, `echo`, `history`, `alias`, and `type`. Bash runs these inside the current shell because they need to affect its state. For example, an external `cd` program could not change the current directory of the Bash process you are using. Bash supports history, alias expansion, variables, command substitution, and command search and execution. [gnu](https://www.gnu.org/software/bash/manual/bash.pdf)

## Vital shell concepts

| Concept | Why it matters operationally |
|---|---|
| Command structure | Helps you read documentation and avoid incorrect syntax. |
| Options and arguments | Options change behavior. Arguments identify input or targets. |
| Environment variables | Configure commands and pass context to child processes. |
| `PATH` | Determines which executable Bash runs for an unqualified command. |
| Command history | Speeds repeated work and supports investigation of prior commands. |
| Aliases | Create shortcuts, but can hide the actual command behavior. |
| Command substitution | Captures command output for use in another command. |
| Quoting | Prevents spaces, special characters, and empty variables from breaking commands. |
| Wildcards and globbing | Select groups of filenames, but can affect more files than expected. |
| Standard streams | Allow commands to read input and separate normal output from errors. |
| Redirection | Saves output, supplies input, and captures errors. |
| Pipes | Build focused processing stages from small commands. |
| Command chaining | Runs follow-up commands based on success or failure. |
| Exit codes | Give scripts and automation a reliable success or failure signal. |

## Quick command reference

### `pwd`

```bash
pwd
```

**Purpose:** Prints the current working directory.

**Important flags:**

- `-L`: Print the logical path, which can preserve symbolic-link components.
- `-P`: Print the physical path after resolving symbolic links.

**Sample output:**

```text
/home/student/cli-lab
```

**How to read it:** This is the directory where relative pathnames will be interpreted.

**Real-world use case:** Run `pwd` before creating, moving, or removing files to confirm that you are working in the intended location.

**Safety classification:** Read-only.

***

### `cd`

```bash
cd ~/cli-lab
```

**Purpose:** Changes Bash’s current working directory.

**Important forms:**

- `cd /etc`: Change to an absolute path.
- `cd ..`: Move to the parent directory.
- `cd ~`: Move to your home directory.
- `cd -`: Return to the previous directory.

**Sample output:**

```text
```

`cd` normally produces no output when successful.

**How to read it:** Verify a successful directory change with `pwd`.

**Real-world use case:** Move to an application directory before checking service-owned files.

**Safety classification:** Changes shell state only.

**Recovery approach:** Use `cd -` to return to the previous directory.

***

### `ls`

```bash
ls -lah
```

**Purpose:** Lists directory entries and their metadata.

**Important flags:**

- `-l`: Long format with permissions, owner, group, size, and time.
- `-a`: Include hidden names beginning with `.`.
- `-h`: Use human-readable sizes with `-l`.
- `-t`: Sort by modification time, newest first.
- `-S`: Sort by size, largest first.

**Sample output:**

```text
total 12K
drwxr-xr-x 2 student student 4.0K Oct 15 09:25 .
drwxr-xr-x 8 student student 4.0K Oct 15 09:25 ..
-rw-r--r-- 1 student student   18 Oct 15 09:25 notes.txt
```

**How to read it:**

- `-rw-r--r--` is the file type and permission string.
- `student student` is the owner and group.
- `18` is the size in bytes.
- `notes.txt` is the filename.

**Real-world use case:** Inspect a directory before troubleshooting missing files, permissions, or unexpected growth.

**Safety classification:** Read-only.

***

### `cp`

```bash
cp notes.txt notes-backup.txt
```

**Purpose:** Copies a file or directory.

**Important flags:**

- `-i`: Ask before overwriting an existing destination.
- `-n`: Do not overwrite an existing destination.
- `-v`: Print each copied path.
- `-r` or `-R`: Copy directories recursively.
- `-a`: Archive mode. Preserves metadata where possible and copies directories recursively.

**Sample output:**

```text
```

`cp` normally produces no output unless `-v` is used.

**How to read it:** Confirm the copy with `ls -lah notes.txt notes-backup.txt` or compare contents with `cmp`.

**Real-world use case:** Create a dated backup of a configuration file before editing it.

**Safety classification:** Changes data in the practice area.

**Undo or recovery:**

```bash
pwd
ls -lah
rm -i notes-backup.txt
```

Run `pwd` and `ls -lah` before using `rm`.

***

### `mv`

```bash
mv notes-backup.txt archived-notes.txt
```

**Purpose:** Renames a file or moves it to another directory.

**Important flags:**

- `-i`: Ask before overwriting.
- `-n`: Do not overwrite existing destinations.
- `-v`: Print each move.
- `-T`: Treat destination as a file, not as a directory, where supported.

**Sample output:**

```text
```

`mv` normally produces no output unless `-v` is used.

**How to read it:** Verify with `ls -lah`. The old name should disappear and the new name should exist.

**Real-world use case:** Rotate an application-generated report before creating a new one.

**Safety classification:** Changes data in the practice area.

**Undo or recovery:**

```bash
mv archived-notes.txt notes-backup.txt
```

***

### `rm`

```bash
rm -i archived-notes.txt
```

**Purpose:** Removes directory entries. Removing the final hard link to a file makes its data eligible for filesystem reuse.

**Important flags:**

- `-i`: Ask before each removal.
- `-I`: Ask once before large or recursive removals.
- `-r` or `-R`: Remove directories recursively.
- `-f`: Do not prompt and ignore missing files.

**Sample output:**

```text
rm: remove regular file 'archived-notes.txt'?
```

**How to read it:** Type `y` only after confirming the exact filename.

**Real-world use case:** Remove a verified temporary report from the lab directory after copying needed evidence elsewhere.

**Safety classification:** Changes data in the practice area.

**Undo or recovery:** `rm` does not provide a built-in undo. Restore from a backup, snapshot, version-control system, or filesystem recovery process if available.

> **Safety requirement before every `rm` command**
>
> ```bash
> pwd
> ls -lah
> ```
>
> Never treat `rm -rf` as routine. `-r` descends into directories. `-f` suppresses prompts and many errors. Together, they can remove a large amount of data quickly.

***

### `mkdir`

```bash
mkdir -p ~/cli-lab/projects/web
```

**Purpose:** Creates directories.

**Important flags:**

- `-p`: Create missing parents and do not fail if target exists.
- `-v`: Print each created directory.
- `-m MODE`: Set permissions at creation.

**Sample output:**

```text
mkdir: created directory '/home/student/cli-lab/projects'
mkdir: created directory '/home/student/cli-lab/projects/web'
```

This sample output appears only with `-v`.

**How to read it:** Each path is a directory created by the command.

**Real-world use case:** Create a structured workspace for maintenance scripts, reports, and collected logs.

**Safety classification:** Changes data in the practice area.

**Undo or recovery:**

```bash
pwd
ls -lah ~/cli-lab/projects
rmdir ~/cli-lab/projects/web
```

Use `rmdir` only when the target directory is empty.

***

### `touch`

```bash
touch notes.txt
```

**Purpose:** Creates an empty file if it does not exist, or updates timestamps if it already exists.

**Important flags:**

- `-c`: Do not create a missing file.
- `-a`: Update access time only.
- `-m`: Update modification time only.
- `-t [[CC]YY]MMDDhhmm[.ss]`: Set a specific timestamp.

**Sample output:**

```text
```

`touch` normally produces no output.

**How to read it:** Verify creation with `ls -lah notes.txt` or `stat notes.txt`.

**Real-world use case:** Create an empty marker file that tells an operational script that a prior stage completed.

**Safety classification:** Changes data in the practice area.

**Undo or recovery:**

```bash
pwd
ls -lah notes.txt
rm -i notes.txt
```

***

### `cat`

```bash
cat notes.txt
```

**Purpose:** Prints file contents to standard output.

**Important flags:**

- `-n`: Number all output lines.
- `-b`: Number nonblank lines.
- `-A`: Show nonprinting characters visibly.

**Sample output:**

```text
daily server notes
```

**How to read it:** Each printed line comes from the file in the argument.

**Real-world use case:** Read a short status file, small configuration file, or generated report.

**Safety classification:** Read-only.

***

### `less`

```bash
less /etc/os-release
```

**Purpose:** Opens a text file in a pager for safe interactive reading.

**Important keys:**

- `Space`: Next page.
- `b`: Previous page.
- `/text`: Search forward.
- `n`: Repeat search.
- `q`: Quit.

**Sample output:**

```text
PRETTY_NAME="Ubuntu 24.04 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
```

**How to read it:** The display is the file content. `less` does not change the file.

**Real-world use case:** Read a large log or configuration file without flooding the terminal.

**Safety classification:** Read-only.

***

### `head`

```bash
head -n 5 /etc/passwd
```

**Purpose:** Shows the first lines of a file.

**Important flags:**

- `-n NUMBER`: Print a specific number of lines.
- `-c NUMBER`: Print a specific number of bytes.
- `-q`: Suppress file headers when reading multiple files.

**Sample output:**

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
```

**How to read it:** `/etc/passwd` is colon-separated. The first field is the account name. Do not assume every entry represents a human user.

**Real-world use case:** Quickly inspect the beginning of a generated report or configuration file.

**Safety classification:** Read-only.

***

### `tail`

```bash
tail -n 5 /etc/passwd
```

**Purpose:** Shows the last lines of a file.

**Important flags:**

- `-n NUMBER`: Print a specific number of final lines.
- `-f`: Follow appended content until interrupted with `Ctrl+C`.
- `-F`: Follow by filename and retry if the log rotates or is recreated.

**Sample output:**

```text
messagebus:x:100:102::/nonexistent:/usr/sbin/nologin
syslog:x:101:103::/home/syslog:/usr/sbin/nologin
uuidd:x:102:105::/run/uuidd:/usr/sbin/nologin
tcpdump:x:103:106::/nonexistent:/usr/sbin/nologin
student:x:1000:1000:Student User:/home/student:/bin/bash
```

**How to read it:** These are the final five account records in this sample file.

**Real-world use case:** Monitor a service log during approved troubleshooting:

```bash
sudo tail -f /var/log/syslog
```

Use `Ctrl+C` to stop following the file. Log paths vary by distribution and logging configuration. Verify on your system.

**Safety classification:** Read-only.

***

### `grep`

```bash
grep -i "error" sample.log
```

**Purpose:** Searches text lines for a pattern.

**Important flags:**

| Option | Meaning |
|---|---|
| `-i` | Ignore case. |
| `-n` | Include matching line numbers. |
| `-v` | Select nonmatching lines. |
| `-c` | Print count of matching lines. |
| `-E` | Use extended regular expressions. |
| `-F` | Treat pattern as fixed text, not a regular expression. |
| `-r` | Search recursively through directories. |

**Sample output:**

```text
2026-10-15T09:18:44Z ERROR database connection timeout
```

**How to read it:** The output contains only lines that match the pattern. By default, `grep` is case-sensitive.

**Real-world use case:** Find error messages in an application log or identify repeated authentication failures.

**Safety classification:** Read-only.

***

### `find`

```bash
find ~/cli-lab -type f -name "*.txt"
```

**Purpose:** Walks a directory tree and selects paths matching conditions.

**Important expressions:**

| Expression | Meaning |
|---|---|
| `-type f` | Regular files. |
| `-type d` | Directories. |
| `-name "*.txt"` | Filename matches a shell-like pattern. Quote the pattern. |
| `-mtime -1` | Modified less than 24 hours ago. |
| `-size +100M` | Larger than 100 MiB. |
| `-user USER` | Owned by a user. |
| `-maxdepth 1` | Limit descent depth. |
| `-print` | Print matches. Usually implied. |

**Sample output:**

```text
/home/student/cli-lab/notes.txt
/home/student/cli-lab/projects/web/report.txt
```

**How to read it:** Every line is a pathname meeting all stated tests.

**Real-world use case:** Find every text report created in a lab workspace or every file modified recently under an application directory.

**Safety classification:** Read-only when used only for searching.

**Important safety note:** `find` can run actions such as `-delete` or `-exec`. Do not use those in early practice. Inspect matches first.

***

### `sort`

```bash
sort names.txt
```

**Purpose:** Sorts input lines.

**Important flags:**

- `-n`: Numeric sort.
- `-r`: Reverse sort.
- `-u`: Sort and remove duplicate lines.
- `-k`: Sort by a field.
- `-t`: Set field separator.

**Sample output:**

```text
alice
bob
carol
```

**How to read it:** Lines are sorted lexicographically unless another mode is chosen.

**Real-world use case:** Sort IP addresses, usernames, or report values before using `uniq -c`.

**Safety classification:** Read-only.

***

### `uniq`

```bash
sort names.txt | uniq -c
```

**Purpose:** Removes adjacent duplicate lines or counts them.

**Important flags:**

- `-c`: Prefix each line with its count.
- `-d`: Print only repeated lines.
- `-u`: Print only lines that appear once.
- `-i`: Ignore case.

**Sample output:**

```text
      3 alice
      1 bob
      2 carol
```

**How to read it:** The number at the beginning is the count. `uniq` only compares adjacent lines, so sort first when counting all duplicates.

**Real-world use case:** Count frequent source addresses or usernames from log-derived data.

**Safety classification:** Read-only.

***

### `cut`

```bash
cut -d: -f1 /etc/passwd
```

**Purpose:** Extracts selected fields or character positions from each input line.

**Important flags:**

- `-d DELIMITER`: Set the field separator.
- `-f LIST`: Select fields.
- `-c LIST`: Select character positions.
- `--complement`: Select everything except chosen fields.

**Sample output:**

```text
root
daemon
bin
sys
```

**How to read it:** `-d:` means colon is the delimiter. `-f1` selects the first field.

**Real-world use case:** Extract account names from `/etc/passwd` or status-code fields from a consistently formatted lab log.

**Safety classification:** Read-only.

***

### `tr`

```bash
tr 'a-z' 'A-Z' < /etc/os-release
```

**Purpose:** Translates or deletes characters from standard input.

**Important flags:**

- `-d`: Delete listed characters.
- `-s`: Squeeze repeated characters into one.
- `-c`: Complement the character set.

**Sample output:**

```text
PRETTY_NAME="UBUNTU 24.04 LTS"
NAME="UBUNTU"
VERSION_ID="24.04"
```

**How to read it:** Lowercase letters from the input become uppercase in the output. The source file is not modified because input is redirected into `tr`.

**Real-world use case:** Normalize input before counting values.

**Safety classification:** Read-only.

***

### `wc`

```bash
wc -l /etc/passwd
```

**Purpose:** Counts lines, words, bytes, or characters.

**Important flags:**

- `-l`: Lines.
- `-w`: Words.
- `-c`: Bytes.
- `-m`: Characters.
- `-L`: Length of longest line.

**Sample output:**

```text
38 /etc/passwd
```

**How to read it:** The number is the line count. Your account count will differ.

**Real-world use case:** Count matching log records after filtering with `grep`.

**Safety classification:** Read-only.

***

### `xargs`

```bash
find ~/cli-lab -type f -name "*.txt" -print0 | xargs -0 wc -l
```

**Purpose:** Converts standard input into arguments for another command.

**Important flags:**

- `-0`: Read NUL-separated items safely.
- `-n NUMBER`: Pass at most a chosen number of arguments per command.
- `-r`: Do not run the command if input is empty, on GNU systems.
- `-t`: Print commands before running them.

**Sample output:**

```text
  2 /home/student/cli-lab/notes.txt
  5 /home/student/cli-lab/projects/web/report.txt
  7 total
```

**How to read it:** `xargs` passes each found pathname to `wc -l`.

**Real-world use case:** Count lines across a controlled set of reports or analyze many log copies.

**Safety classification:** Depends on the command after `xargs`. Here it is read-only.

**Safety note:** Use `find -print0` with `xargs -0` for arbitrary filenames. This prevents spaces, tabs, quotes, and newlines in filenames from being split incorrectly.

***

### `file`

```bash
file /etc/passwd
```

**Purpose:** Identifies a file’s content type using data inspection rules.

**Important flags:**

- `-b`: Brief output without the filename.
- `-i`: Show MIME-style type information.
- `-L`: Follow symbolic links.
- `-z`: Try to inspect compressed data.

**Sample output:**

```text
/etc/passwd: ASCII text
```

**How to read it:** The output is a best-effort identification based on file content and metadata. It is not a security guarantee.

**Real-world use case:** Check an unknown file before opening or executing it.

**Safety classification:** Read-only.

***

### `stat`

```bash
stat /etc/passwd
```

**Purpose:** Displays detailed metadata for a filesystem object.

**Important flags:**

- `-c FORMAT`: Print selected fields in a custom format.
- `-L`: Follow symbolic links.
- `-f`: Show filesystem status instead of file status.

**Sample output:**

```text
  File: /etc/passwd
  Size: 1847       Blocks: 8          IO Block: 4096   regular file
Device: 8,2   Inode: 524321      Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2026-10-15 08:40:00.000000000 +0530
Modify: 2026-10-10 14:20:00.000000000 +0530
Change: 2026-10-10 14:20:00.000000000 +0530
 Birth: 2026-09-01 11:00:00.000000000 +0530
```

**How to read it:**

- `Size` is the logical file size.
- `Inode` identifies the filesystem object on that filesystem.
- `Access` permissions show who can read, write, or execute.
- `Modify` records content changes.
- `Change` records metadata changes.
- Birth time may not be available on every filesystem.

**Real-world use case:** Check ownership, permissions, timestamps, and inode information during troubleshooting or an authorized forensic review.

**Safety classification:** Read-only.

***

### `history`

```bash
history | tail
```

**Purpose:** Shows commands remembered by Bash.

**Important forms:**

- `history`: Show all available history entries.
- `history N`: Show the last `N` entries.
- `history -c`: Clear in-memory history. Avoid this during normal learning or investigations.
- `!!`: Expand to the prior command.
- `!$`: Expand to the final argument of the prior command.

**Sample output:**

```text
  112  pwd
  113  ls -lah
  114  touch notes.txt
  115  cat notes.txt
  116  history | tail
```

**How to read it:** The left number is the Bash history event number. History may include commands restored from prior sessions.

**Real-world use case:** Repeat or review a command during troubleshooting. Treat history as sensitive because it may contain paths, account names, tokens, or accidental secrets.

**Safety classification:** Read-only.

***

### `alias`

```bash
alias ll='ls -lah'
```

**Purpose:** Creates a shortcut that Bash expands before command execution.

**Important forms:**

- `alias`: List aliases.
- `alias NAME='COMMAND'`: Create an alias in the current shell.
- `unalias NAME`: Remove an alias.
- `type NAME`: Reveal whether a command name is an alias, builtin, function, or executable.

**Sample output:**

```text
alias ll='ls -lah'
```

**How to read it:** `ll` will expand to `ls -lah` in the current interactive shell.

**Real-world use case:** Define a personal read-only listing shortcut.

**Safety classification:** Changes shell state only.

**Undo or recovery:**

```bash
unalias ll
```

***

### `env`

```bash
env | less
```

**Purpose:** Prints the environment passed to the current process.

**Important forms:**

- `env`: Show environment variables.
- `env NAME=value command`: Run one command with a temporary environment variable value.
- `env -i command`: Run with an empty environment. Use carefully.

**Sample output:**

```text
HOME=/home/student
USER=student
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

**How to read it:** Each line has the form `NAME=value`. Values are inherited by child processes unless changed.

**Real-world use case:** Compare an interactive shell environment with a service or script environment.

**Safety classification:** Read-only.

***

### `echo`

```bash
echo "$HOME"
```

**Purpose:** Prints text and expanded variable values.

**Important flags:**

- `-n`: Do not print the ending newline.
- `-e`: Interpret selected backslash escapes. Avoid relying on it in portable scripts.

**Sample output:**

```text
/home/student
```

**How to read it:** Bash expands `$HOME` before `echo` runs because it is inside double quotes.

**Real-world use case:** Print values during a script’s diagnostic output.

**Safety classification:** Read-only.

***

### `which`

```bash
which bash
```

**Purpose:** Shows a command path found through `PATH`.

**Important note:** `which` may not correctly describe aliases, functions, shell builtins, or shell-specific command resolution. Prefer Bash’s `type` for interactive Bash investigation.

**Sample output:**

```text
/usr/bin/bash
```

**How to read it:** This is the executable path found by `which` in the current environment.

**Real-world use case:** Check which executable path a non-builtin command resolves to.

**Safety classification:** Read-only.

***

### `man`

```bash
man bash
```

**Purpose:** Opens the manual page for a command or system interface.

**Important forms:**

- `man bash`: Bash reference page.
- `man 1 grep`: User-command manual for `grep`.
- `man 5 proc`: File-format and kernel interface documentation for `/proc`.
- `man -k keyword`: Search manual-page descriptions by keyword.

**Sample output:**

```text
BASH(1)                  General Commands Manual                 BASH(1)

NAME
       bash - GNU Bourne-Again SHell

SYNOPSIS
       bash [options] [command_string | file]
```

**How to read it:** `NAME` gives a short description. `SYNOPSIS` shows command structure. Square brackets usually indicate optional items. Replace italic placeholders with real values. [man.archlinux](https://man.archlinux.org/man/man-pages.7.en)

**Real-world use case:** Verify flags, exit codes, accepted input, and operational caveats before using a command in production.

**Safety classification:** Read-only.

> `man bash` is very long. Use it as a reference after learning the basics rather than trying to read it from beginning to end. The GNU Bash manual documents command substitution, aliases, history, variables, command search, and execution behavior. [gnu](https://www.gnu.org/software/bash/manual/html_node/index.html)

## Common mistakes

1. Running file-changing commands without first confirming the current directory with `pwd`.
2. Treating aliases or `which` output as a complete explanation of command resolution.
3. Using an unfamiliar option without reading `man command` or `command --help`.

***

# 4.1 Terminal, shell, and Bash

**Level: Beginner**

> ## The 20% that matters
>
> - A terminal is the interface window or device. A shell is the command interpreter inside it.
> - Bash is a commonly used Linux shell.
> - `cd` is a Bash builtin because it must change the current shell’s directory.
> - Bash expands special syntax, finds a command, runs it, and stores an exit status.
> - Use `type` to identify whether a name is an alias, builtin, function, or executable.

## Concept

A **terminal** is a text interface where you type and view output. A terminal emulator is the program that provides that interface in a desktop environment or remote SSH session.

A **shell** is a command interpreter. It reads your input and starts programs or performs shell-built-in actions.

**Bash**, short for Bourne Again Shell, is one shell commonly used on Linux systems. Ubuntu often uses Bash for interactive administrative work, though a user may choose another shell.

A **command prompt** is the text that indicates Bash is ready for input. It may look like:

```text
student@labvm:~/cli-lab$
```

A **command** is an instruction such as `ls`, `grep`, or `cd`.

A **process** is a running instance of a program. When Bash starts an external command such as `ls`, the operating system creates or runs a process for it. Bash itself is also a process.

### Bash command flow

```text
Bash reads command
  -> expands variables and globs
  -> resolves command using PATH
  -> executes process
  -> returns exit code
```

At a practical level, Bash does this:

1. Reads the command line.
2. Interprets shell syntax such as quotes, variables, wildcards, redirection, and pipes.
3. Checks whether the command is an alias, function, shell builtin, or executable.
4. Searches directories in `PATH` when needed.
5. Runs the command.
6. Receives an exit status.
7. Stores that status in `$?`.

Bash documents command search and execution as core shell behavior. [gnu](https://www.gnu.org/software/bash/manual/html_node/index.html)

## Commands

```bash
echo $SHELL
bash --version
type ls
type cd
which bash
man bash
```

## Command-by-command explanation

### Check the configured shell

```bash
echo $SHELL
```

- `echo` prints text.
- `$SHELL` is an environment variable that often identifies the user’s login shell.
- This value may not always identify the shell currently running inside a nested terminal session. Verify with `ps -p $$ -o comm=` if needed later.

**Sample output:**

```text
/bin/bash
```

Read this as the configured shell path for the account.

### Check Bash version

```bash
bash --version
```

- `bash` starts Bash or queries Bash options.
- `--version` prints version and license information.

**Sample output:**

```text
GNU bash, version 5.2.21(1)-release (x86_64-pc-linux-gnu)
Copyright (C) 2022 Free Software Foundation, Inc.
```

Your Bash version can differ. Verify it on your VM.

### Identify an external command

```bash
type ls
```

- `type` is a Bash builtin.
- It explains how Bash resolves a command name.
- `ls` is usually an external executable.

**Sample output:**

```text
ls is /usr/bin/ls
```

Read this as: Bash will run `/usr/bin/ls` when you type `ls`, unless an alias, function, or different command lookup rule takes precedence.

### Identify a shell builtin

```bash
type cd
```

**Sample output:**

```text
cd is a shell builtin
```

`cd` must run in the current shell process. If it ran in a separate child process, the parent Bash shell would stay in the original directory.

### Find the Bash executable

```bash
which bash
```

**Sample output:**

```text
/usr/bin/bash
```

This is the executable path found by `which`. Use `type -a bash` to see multiple possible matches.

### Read Bash documentation

```bash
man bash
```

Use navigation keys inside the manual page:

```text
Space    Next page
b        Previous page
/text    Search for text
n        Next match
q        Quit
```

## Real-world use case

A script behaves differently on two servers. Start by identifying the interpreter and command type:

```bash
echo "$SHELL"
type command-name
type -a command-name
bash --version
```

This can reveal that one system runs Bash while another uses a different shell, or that an alias or shell function changes expected behavior.

## Common mistakes

1. Assuming the terminal and the shell are the same component.
2. Assuming every command is an external executable instead of checking with `type`.
3. Reading `$SHELL` as proof of the currently executing shell in every situation.

***

# 4.2 Commands, Arguments, and Options

**Level: Beginner**

> ## The 20% that matters
>
> - Read command syntax as `command [options] [arguments]`.
> - Options change how a command behaves.
> - Arguments usually name input, output, paths, patterns, or values.
> - Short options use one hyphen, such as `-l`.
> - Long options use two hyphens, such as `--all`.

## Concept

Most Linux commands follow this general structure:

```bash
command [options] [arguments]
```

- `command` is the program or shell builtin to run.
- `[options]` are optional switches that alter behavior.
- `[arguments]` are values passed to the command.

Brackets in documentation usually mean “optional.” Do not type the brackets unless documentation explicitly says to do so.

## Commands

```bash
ls
ls -l
ls -lah
ls --all
grep -i error sample.log
cp source.txt destination.txt
```

## Examples

### `ls`

```bash
ls
```

- Command: `ls`
- Options: none
- Arguments: none, so it lists the current directory.

**Sample output:**

```text
notes.txt
projects
```

### Long listing

```bash
ls -l
```

- Command: `ls`
- Option: `-l`
- Argument: none

**Sample output:**

```text
-rw-r--r-- 1 student student 18 Oct 15 09:25 notes.txt
drwxr-xr-x 3 student student 4.0K Oct 15 09:26 projects
```

### Combined short options

```bash
ls -lah
```

This combines:

```text
-l
-a
-h
```

Bash passes `-lah` to `ls`. The command interprets it as multiple short options because `ls` supports that form.

**Sample output:**

```text
total 12K
drwxr-xr-x 3 student student 4.0K Oct 15 09:26 .
drwxr-xr-x 8 student student 4.0K Oct 15 09:25 ..
-rw-r--r-- 1 student student   18 Oct 15 09:25 notes.txt
drwxr-xr-x 3 student student 4.0K Oct 15 09:26 projects
```

### Long option

```bash
ls --all
```

- `--all` includes hidden entries that begin with `.`.

**Sample output:**

```text
.  ..  .hidden-note  notes.txt  projects
```

### `grep` with option and arguments

Create a safe practice log first:

```bash
printf '%s\n' \
  'INFO service started' \
  'ERROR database timeout' \
  'INFO request complete' > sample.log
```

Now search it:

```bash
grep -i error sample.log
```

- Command: `grep`
- Option: `-i`, ignore case
- First argument: `error`, the search pattern
- Second argument: `sample.log`, the input file

**Sample output:**

```text
ERROR database timeout
```

### `cp` with source and destination arguments

```bash
cp source.txt destination.txt
```

- Command: `cp`
- First argument: source file
- Second argument: destination file

Create a safe example:

```bash
printf 'source data\n' > source.txt
cp source.txt destination.txt
ls -lah source.txt destination.txt
```

**Sample output:**

```text
-rw-r--r-- 1 student student 12 Oct 15 09:30 destination.txt
-rw-r--r-- 1 student student 12 Oct 15 09:30 source.txt
```

## How to read syntax documentation

Consider:

```bash
grep [OPTION]... PATTERNS [FILE]...
```

This means:

- `grep` is required.
- Options are optional.
- One or more patterns may be accepted.
- File arguments may be accepted.
- `...` often means the item may repeat.

Check `man grep` for exact syntax and behavior before using unfamiliar options.

## Real-world use case

An administrator wants to find authentication failures in a log:

```bash
sudo grep -i "failed" /var/log/auth.log
```

- `sudo` runs `grep` with elevated privileges if the log requires it.
- `grep` searches.
- `-i` ignores case.
- `"failed"` is the pattern.
- `/var/log/auth.log` is the log path.

The filename and log availability vary by distribution and logging configuration. Verify on your system.

## Common mistakes

1. Putting options after an argument when a command requires options before arguments.
2. Forgetting that arguments with spaces need quotes.
3. Assuming every command supports the same options or combined short-option format.

***

# 4.3 Environment Variables and `PATH`

**Level: Intermediate**

> ## The 20% that matters
>
> - A shell variable exists in the current shell unless exported.
> - An environment variable is inherited by child processes.
> - `PATH` is a colon-separated search list for executable commands.
> - Put user-controlled writable directories carefully in `PATH`.
> - Scripts should use absolute paths for critical commands or validate dependencies first.

## Concept

A **variable** stores a value.

A **shell variable** exists in the current Bash process:

```bash
PROJECT_NAME="cli-lab"
```

An **environment variable** is exported so child processes inherit it:

```bash
export PROJECT_NAME="cli-lab"
```

Environment variables influence command behavior. Common examples include:

| Variable | Typical purpose |
|---|---|
| `HOME` | User home directory. |
| `USER` | Current username. |
| `SHELL` | Configured shell path. |
| `PATH` | Command search directories. |
| `LANG` | Locale and language behavior. |
| `EDITOR` | Preferred text editor. |
| `HISTFILE` | Bash history file location. |

## `PATH`

When you type a command without a slash, Bash looks for an executable in directories listed in `PATH`.

```text
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Bash searches from left to right. If two directories contain an executable with the same name, Bash normally finds the first matching one.

Ubuntu documentation describes `PATH` as the environment variable containing directories searched when you run a command without specifying an executable path. [wiki.ubuntu](https://wiki.ubuntu.com/PATH)

## Commands

```bash
echo $HOME
echo $USER
echo $SHELL
echo $PATH
env | less
printenv PATH
export LAB_DIR="$HOME/cli-lab"
echo "$LAB_DIR"
which ls
type -a ls
```

## Demonstration

### Inspect common variables

```bash
echo "$HOME"
echo "$USER"
echo "$SHELL"
echo "$PATH"
```

**Sample output:**

```text
/home/student
student
/bin/bash
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

**How to read it:**

- `$HOME` is the user’s home directory.
- `$USER` is the login name in this sample.
- `$SHELL` is usually the user’s configured login shell.
- `$PATH` contains directories separated by `:`.

Use double quotes around variables. They preserve the variable as one argument if its value contains spaces or is empty.

### Inspect the full environment

```bash
env | less
```

**Sample output:**

```text
HOME=/home/student
LANG=en_US.UTF-8
LOGNAME=student
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/home/student/cli-lab
SHELL=/bin/bash
USER=student
```

Each line is a `NAME=value` pair.

### Print one environment variable

```bash
printenv PATH
```

**Sample output:**

```text
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

`printenv` prints environment variables, not unexported shell-only variables.

### Create a session-only environment variable

```bash
export LAB_DIR="$HOME/cli-lab"
echo "$LAB_DIR"
```

**Sample output:**

```text
/home/student/cli-lab
```

`export` makes `LAB_DIR` available to child processes started from this Bash session.

Confirm this safely:

```bash
env | grep '^LAB_DIR='
```

**Sample output:**

```text
LAB_DIR=/home/student/cli-lab
```

### Find command resolution

```bash
which ls
type -a ls
```

**Sample output:**

```text
/usr/bin/ls
ls is /usr/bin/ls
ls is /bin/ls
```

On systems with merged `/usr`, `/bin` may be a symbolic link to `/usr/bin`. The exact output varies by distribution and system layout.

## Add a directory to `PATH` for one session

Create a personal command directory:

```bash
mkdir -p "$HOME/bin"
export PATH="$HOME/bin:$PATH"
echo "$PATH"
```

The new directory appears first. That means a command in `$HOME/bin` can take precedence over a system command with the same name.

**Sample output:**

```text
/home/student/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

## Make a variable persistent

For an interactive Bash configuration, add an `export` line to `~/.bashrc`.

First back up the file:

```bash
cp -i "$HOME/.bashrc" "$HOME/.bashrc.backup"
```

Then edit with an editor you understand. For example:

```bash
nano "$HOME/.bashrc"
```

Add this line:

```bash
export LAB_DIR="$HOME/cli-lab"
```

To add a user command directory to the front of `PATH`:

```bash
export PATH="$HOME/bin:$PATH"
```

Apply the change to the current shell:

```bash
source "$HOME/.bashrc"
```

or:

```bash
. "$HOME/.bashrc"
```

Do not add untrusted or broadly writable directories to `PATH`. A malicious or accidental executable with the same name as a trusted command may be run first.

## Script design guidance

For critical scripts:

- Use absolute paths for important tools when appropriate, such as `/usr/bin/find`.
- Or verify dependencies before use:

```bash
command -v grep >/dev/null 2>&1 || {
    echo "grep is required but was not found in PATH" >&2
    exit 1
}
```

- Use a controlled `PATH` if a script runs from cron, a service, or automation.
- Do not assume an interactive shell profile has run.

## Real-world use case

A script works when an administrator runs it manually but fails when scheduled. The scheduler often has a smaller environment and a different `PATH`.

Inspect and set values explicitly:

```bash
env > "$HOME/cli-lab/interactive-environment.txt"
printenv PATH
command -v journalctl
command -v grep
```

For scripts, define known dependencies and paths instead of relying on interactive aliases or shell startup files.

## Common mistakes

1. Using `VAR = value` with spaces, which is not valid Bash assignment syntax.
2. Adding `.` or an untrusted writable directory early in `PATH`.
3. Assuming shell variables are automatically inherited by commands without `export`.

***

# 4.4 Command History and Aliases

**Level: Intermediate**

> ## The 20% that matters
>
> - `history` shows prior commands available to the current Bash session.
> - `Ctrl+R` searches backward through history interactively.
> - `!!` repeats the previous command and `!$` expands to its final argument.
> - Aliases are text substitutions, not full shell functions.
> - Use `type name` before trusting a familiar command name.

## Concept

Bash history stores commands so you can recall and repeat them. In interactive shells, Bash history and history expansion are normally enabled. Bash commonly uses `~/.bash_history` as the default history file, though configuration can change this. [gnu](https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html)

History may contain sensitive information. Avoid placing passwords, private keys, access tokens, or confidential values directly on a command line.

An **alias** is a textual command shortcut. Bash checks unquoted command names for aliases in command position and substitutes the alias value before execution. [ftp.gnu](https://ftp.gnu.org/old-gnu/Manuals/bash/html_chapter/bashref_6.html)

## Commands

```bash
history
history | tail
alias
alias ll='ls -lah'
ll
unalias ll
```

## Demonstration

### Show command history

```bash
history
```

**Sample output:**

```text
  201  pwd
  202  ls -lah
  203  touch notes.txt
  204  cat notes.txt
  205  history
```

The left column is the history event number. The remainder is the command text.

Show only the most recent entries:

```bash
history | tail
```

**Sample output:**

```text
  196  cd ~/cli-lab
  197  pwd
  198  ls -lah
  199  echo "$PATH"
  200  history | tail
```

### Repeat the prior command

```bash
!!
```

If your previous command was:

```bash
ls -lah
```

Bash expands `!!` to:

```bash
ls -lah
```

This is convenient but risky after commands that modify data. Prefer typing the full command when the prior command was destructive or privileged.

### Reuse the previous final argument

```bash
mkdir -p ~/cli-lab/history-demo
cd !$
```

After the first command, `!$` expands to the final argument:

```bash
cd ~/cli-lab/history-demo
```

Use it only when you have verified the prior command and its final argument.

### Reverse history search

Press:

```text
Ctrl+R
```

Then type a fragment, such as:

```text
grep
```

Bash displays the most recent matching command. Press `Ctrl+R` again for earlier matches. Press `Enter` to run the displayed command or use arrow keys to edit it first.

### Inspect history configuration

```bash
echo "$HISTSIZE"
echo "$HISTFILESIZE"
echo "$HISTFILE"
```

**Sample output:**

```text
1000
2000
/home/student/.bash_history
```

Interpretation:

- `HISTSIZE` controls how many commands Bash retains in memory.
- `HISTFILESIZE` controls the approximate number of lines retained in the history file.
- `HISTFILE` identifies the history file path.

Values are user and distribution dependent. Verify on your system.

## Aliases

### List aliases

```bash
alias
```

**Sample output:**

```text
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
```

Default aliases vary by distribution and user configuration.

### Create a temporary alias

```bash
alias ll='ls -lah'
```

Test it:

```bash
ll
```

**Sample output:**

```text
total 16K
drwxr-xr-x 3 student student 4.0K Oct 15 10:05 .
drwxr-xr-x 8 student student 4.0K Oct 15 09:25 ..
-rw-r--r-- 1 student student   18 Oct 15 09:25 notes.txt
drwxr-xr-x 2 student student 4.0K Oct 15 10:04 projects
```

### Inspect an alias safely

```bash
type ll
```

**Sample output:**

```text
ll is aliased to `ls -lah'
```

This is safer than assuming what `ll` means on another server.

### Remove an alias

```bash
unalias ll
```

Verify:

```bash
type ll
```

**Sample output:**

```text
bash: type: ll: not found
```

### Make an alias persistent

To make a personal alias available in future interactive Bash sessions, add it to `~/.bashrc`.

First create a backup:

```bash
cp -i "$HOME/.bashrc" "$HOME/.bashrc.backup"
```

Edit:

```bash
nano "$HOME/.bashrc"
```

Add:

```bash
alias ll='ls -lah'
```

Reload Bash configuration:

```bash
source "$HOME/.bashrc"
```

Aliases are useful for interactive work, but avoid relying on them in scripts. Non-interactive shells may not load aliases, and aliases can obscure the real command behavior.

## Real-world use case

A junior administrator says `rm` behaves differently on two servers. Investigate before running it:

```bash
type rm
alias rm
```

One account may define:

```bash
alias rm='rm -i'
```

Another account may not. A script should not rely on this protection. Use explicit options where needed.

## Common mistakes

1. Repeating a dangerous command with `!!` without checking what the prior command was.
2. Assuming aliases exist in scripts, cron jobs, containers, or other users’ sessions.
3. Saving secrets in shell history.

***

# 4.5 Command Substitution

**Level: Intermediate**

> ## The 20% that matters
>
> - `$(command)` runs a command and inserts its standard output.
> - Use it to capture short values, such as a timestamp or current directory.
> - Double quotes allow command substitution. Single quotes keep it literal.
> - Prefer `$(...)` over legacy backticks.
> - Quote command substitutions when their output may contain spaces.

## Concept

**Command substitution** runs a command and replaces the substitution syntax with that command’s standard output.

The standard form is:

```bash
$(command)
```

Bash runs the command in a subshell environment, captures its standard output, removes trailing newlines, and inserts the result into the surrounding command. GNU Bash documents `$(command)` as the standard form and identifies backticks as deprecated syntax. [gnu](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html)

## Commands

```bash
echo "Today is $(date)"
echo "Home directory: $HOME"
echo "Current directory: $(pwd)"
echo "File count: $(ls | wc -l)"
```

## Demonstration

### Insert the current date

```bash
echo "Today is $(date)"
```

**Sample output:**

```text
Today is Thu Oct 15 10:15:30 IST 2026
```

Your date, timezone, and format will differ.

Read this as: Bash ran `date`, captured its output, then passed the completed text to `echo`.

### Compare variable expansion and command substitution

```bash
echo "Home directory: $HOME"
```

**Sample output:**

```text
Home directory: /home/student
```

`$HOME` is variable expansion. Bash replaces it with a stored value.

```bash
echo "Current directory: $(pwd)"
```

**Sample output:**

```text
Current directory: /home/student/cli-lab
```

`$(pwd)` is command substitution. Bash runs `pwd` and inserts its output.

### Count visible directory entries

```bash
echo "File count: $(ls | wc -l)"
```

**Sample output:**

```text
File count: 6
```

Interpret carefully:

- `ls` lists visible entries.
- `wc -l` counts output lines.
- The count can be misleading for unusual filenames containing newline characters.
- For controlled lab directories, it is acceptable as a teaching example.
- Later, use `find ... -print0` with NUL-aware tools for robust scripting.

## Single quotes versus double quotes

```bash
echo "Today is $(date)"
echo 'Today is $(date)'
```

**Sample output:**

```text
Today is Thu Oct 15 10:15:30 IST 2026
Today is $(date)
```

Double quotes allow variable expansion and command substitution. Single quotes preserve nearly all characters literally.

## Why use `$(...)` instead of backticks

Legacy syntax:

```bash
echo "Today is `date`"
```

Preferred syntax:

```bash
echo "Today is $(date)"
```

`$(...)` is easier to read and easier to nest:

```bash
echo "Current directory name: $(basename "$(pwd)")"
```

This command runs `pwd`, passes its output to `basename`, then inserts the result into `echo`.

## Safe operational examples

Create a timestamped report filename:

```bash
REPORT_FILE="$HOME/cli-lab/report-$(date +%F).txt"
echo "$REPORT_FILE"
```

**Sample output:**

```text
/home/student/cli-lab/report-2026-10-15.txt
```

Here:

- `REPORT_FILE=` assigns a shell variable.
- `$(date +%F)` creates a date in `YYYY-MM-DD` format.
- Double quotes preserve the result as one value.

## Real-world use case

An administrator creates a timestamped, read-only status report:

```bash
REPORT="$HOME/cli-lab/system-status-$(date +%F-%H%M%S).txt"
df -hT > "$REPORT"
echo "Saved report to: $REPORT"
```

This stores disk-usage output in a uniquely named lab report. The `df` command is read-only. The output redirection creates a file in `~/cli-lab`.

## Common mistakes

1. Using single quotes when variable or command expansion is required.
2. Leaving a command substitution unquoted when its output may contain spaces.
3. Using legacy backticks in complex commands where nesting becomes hard to read.

***

## Part 1 Practice Exercises

### Core Exercise 1: Confirm your lab location

**Goal:** Verify that you are working only inside `~/cli-lab`.

**Commands:**

```bash
cd ~/cli-lab
pwd
ls -lah
```

**Expected result:** `pwd` prints a path ending in `/cli-lab`.

**Undo step:** None. These commands are read-only except for changing the current shell directory.

***

### Core Exercise 2: Inspect command types

**Goal:** Determine how Bash resolves command names.

**Commands:**

```bash
type cd
type ls
type echo
type -a bash
```

**Expected result:** `cd` and `echo` should be identified as Bash builtins. `ls` and `bash` should typically resolve to executable paths.

**Undo step:** None. These commands are read-only.

***

### Core Exercise 3: Create and inspect variables

**Goal:** Create one shell variable and one exported environment variable.

**Commands:**

```bash
COURSE="Linux Command Line"
export LAB_OWNER="$USER"

echo "$COURSE"
echo "$LAB_OWNER"
env | grep '^LAB_OWNER='
```

**Expected result:** `COURSE` prints in the current shell. `LAB_OWNER` also appears in `env` because it was exported.

**Undo step:**

```bash
unset COURSE
unset LAB_OWNER
```

***

### Core Exercise 4: Inspect and test `PATH`

**Goal:** Understand command lookup.

**Commands:**

```bash
echo "$PATH"
which ls
type -a ls
command -v grep
```

**Expected result:** The commands show one or more executable paths. `type -a` may display both `/usr/bin/ls` and `/bin/ls` on merged-`/usr` systems.

**Undo step:** None. These commands are read-only.

***

### Core Exercise 5: Use a temporary alias

**Goal:** Create, inspect, use, and remove an alias.

**Commands:**

```bash
alias labls='ls -lah ~/cli-lab'
type labls
labls
unalias labls
type labls
```

**Expected result:** The first `type labls` shows the alias definition. The final `type labls` reports that it is not found.

**Undo step:** `unalias labls` removes the temporary alias.

***

### Extension Exercise 1: Create a timestamped status file

**Goal:** Use command substitution to build a file name.

**Commands:**

```bash
STATUS_FILE="$HOME/cli-lab/status-$(date +%F-%H%M%S).txt"
printf 'Current directory: %s\n' "$(pwd)" > "$STATUS_FILE"
printf 'Current user: %s\n' "$USER" >> "$STATUS_FILE"
cat "$STATUS_FILE"
```

**What additional skill it develops:** Variable assignment, command substitution, quoting, and basic output redirection.

***

### Extension Exercise 2: Inspect your Bash history settings

**Goal:** Determine where Bash stores history and how many entries it retains.

**Commands:**

```bash
printf 'HISTFILE=%s\n' "$HISTFILE"
printf 'HISTSIZE=%s\n' "$HISTSIZE"
printf 'HISTFILESIZE=%s\n' "$HISTFILESIZE"
history | tail -n 10
```

**What additional skill it develops:** History configuration awareness and secure handling of shell history.

***

### Extension Exercise 3: Compare shell and environment variables

**Goal:** Observe the difference between an unexported and exported variable.

**Commands:**

```bash
LOCAL_ONLY="not exported"
export EXPORTED_VALUE="available to child processes"

bash -c 'printf "LOCAL_ONLY=%s\n" "$LOCAL_ONLY"'
bash -c 'printf "EXPORTED_VALUE=%s\n" "$EXPORTED_VALUE"'
```

**What additional skill it develops:** Understanding inheritance between Bash and child processes.
