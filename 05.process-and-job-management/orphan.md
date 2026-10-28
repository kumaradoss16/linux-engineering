An orphan process in Linux is a process whose parent process has terminated while the child process is still running. Linux typically reassigns the orphan to a system process such as `systemd` (PID 1) or another configured child subreaper.

## 1. Zombie vs orphan process

| Feature      | Orphan process                          | Zombie process                                      |
| ------------ | --------------------------------------- | --------------------------------------------------- |
| Parent       | Original parent has terminated          | Parent has not collected the child's exit status    |
| Child status | Still running or otherwise alive        | Already terminated                                  |
| `STAT`       | Often `S` or `R`, depending on activity | `Z`                                                 |
| Resource use | Can use CPU and memory                  | No longer executes or uses CPU                      |
| Cleanup      | Normally continues until it exits       | Must be reaped by its parent or an adopting process |

## 2. Create an orphan process in Ubuntu

### Step 1 - Create a test script

```
nano orphan.py
```

Paste:

```
import osimport timepid = os.fork()if pid == 0:    # Child process    print(f"Child PID: {os.getpid()}")    print(f"Original parent PID: {os.getppid()}")    time.sleep(120)    print(f"Parent PID after waiting: {os.getppid()}")else:    # Parent exits immediately    print(f"Parent PID: {os.getpid()}")    print(f"Child PID: {pid}")    os._exit(0)
```

Save and exit Nano: `Ctrl+O`, `Enter`, then `Ctrl+X`.

### Step 2 - Run the script

```
python3 orphan.py
```

The parent exits immediately, while the child continues sleeping for 120 seconds.

Note: The parent may exit before the child's first print statement appears. This is normal because the processes execute independently.

## 3. Verify the orphan process

Use the child PID printed by the script:

```
ps -o pid,ppid,stat,cmd -p CHILD_PID
```

Replace `CHILD_PID` with the actual PID.

Example:

```
    PID    PPID STAT CMD
   5120       1 S    python3 orphan.py
```

Here, the child is still alive, and its PPID may be `1` after adoption. On systems with a different configured subreaper, the PPID may be another process.

You can also inspect it continuously:

```
watch -n 1 'ps -eo pid,ppid,stat,cmd | grep "[o]rphan.py"'
```

Press `Ctrl+C` to stop `watch`.

## 4. Verify cleanup

The child exits naturally after 120 seconds. You can check whether it is still running:

```
ps -p CHILD_PID -o pid,ppid,stat,cmd
```

Replace `CHILD_PID` with the actual PID. If the process has exited and been removed, no process row will appear.

## 5. Important distinction

An orphan is not inherently an error. Linux is designed to handle orphan processes automatically.

- Orphan: The child is still alive after its original parent exits.
- Zombie: The child has exited, but its exit status has not yet been collected.

For process-management practice, remember:

```
# Find likely zombies
ps -eo pid,ppid,stat,comm | awk 'NR == 1 || $3 ~ /^Z/'

# Inspect a specific process and its parent
ps -o pid,ppid,stat,cmd -p PID
```

The second command helps you inspect the process relationship, but a PPID of `1` alone does not prove that a process is an orphan; it may have been launched directly by PID 1 or adopted earlier.
