# Basic `pstree` Commands in Linux

The `pstree` command displays running processes in a tree structure, showing the relationships between parent processes and their child processes.

It is useful for Linux system administration, troubleshooting, and understanding how applications are launched.

## 1. Basic `pstree` commands

| Command         | Purpose                                                                           |
| --------------- | --------------------------------------------------------------------------------- |
| `pstree`        | Displays the process tree.                                                        |
| `pstree -p`     | Displays the process tree with PIDs.                                              |
| `pstree -u`     | Shows changes in user ownership between processes.                                |
| `pstree -a`     | Displays command-line arguments.                                                  |
| `pstree -n`     | Sorts sibling processes numerically by PID.                                       |
| `pstree -s PID` | Displays the ancestors of a specified process.                                    |
| `pstree -p PID` | Displays the process tree rooted at a specified PID.                              |
| `pstree -h`     | Highlights the current process and its ancestors, when supported by the terminal. |

## 2. Practical examples

### Example 1: Display all processes

```
pstree
```

Example output:

```
systemd─┬─NetworkManager
        ├─sshd───sshd───bash
        ├─cron
        └─python3
```

Meaning: `systemd` is the parent of several processes. The `sshd` process launches another `sshd`, which launches `bash`.

The output is illustrative; actual process names and relationships vary by system.

### Example 2: Display process IDs

```
pstree -p
```

Example:

```
systemd(1)───sshd(850)───bash(1234)
```

The numbers in parentheses are process IDs (PIDs).

### Example 3: Display command-line arguments

```
pstree -a
```

This displays the command-line arguments used to start processes, where available.

For example, a Python process may appear with its script path:

```
python3 process-lab/signal_demo.py
```

### Example 4: Display the ancestors of a process

```
pstree -s 1234
```

Shows the chain of ancestor processes leading to PID `1234`, subject to the command's output conventions.

Useful when you want to find which process launched a shell, script, or application.

### Example 5: Display the tree rooted at a specific PID

```
pstree -p 1234
```

Shows PID `1234` and its descendants, rather than the entire process tree.

### Example 6: Show user ownership changes

```
pstree -u
```

Displays user transitions in the tree, helping you identify where process ownership changes.

## 3. Useful combinations

```
# Entire process tree with PIDs and arguments
pstree -pa

# Process tree with user transitions and PIDs
pstree -pu

# Ancestors of a specific process
pstree -s 1234

# Descendants of a specific process
pstree -p 1234
```

## 4. If `pstree` is not installed

On Ubuntu or Debian:

```
sudo apt update
sudo apt install psmisc
```

The `pstree` utility is commonly provided by the `psmisc` package.

Recommended starting point: Learn `pstree`, `pstree -p`, `pstree -pa`, and `pstree -s PID`. These cover the most common process-tree inspection tasks.s
