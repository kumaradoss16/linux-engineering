You can create a zombie process in Linux for learning and process-management practice using a small Python or C program.

A zombie process is a child process that has finished execution but whose parent has not yet collected its exit status using a call such as `wait()`. Its process table entry remains until the parent reaps it.

## 1. Create a zombie process using Python

On your Ubuntu Server, create a test script:

```
nano zombie.py
```

Paste this code:

```
import osimport timepid = os.fork()if pid == 0:    # Child process exits immediately    os._exit(0)else:    # Parent does not collect the child's exit status yet    print(f"Parent PID: {os.getpid()}")    print(f"Child PID: {pid}")    print("Child exited. Waiting 60 seconds without calling wait().")    time.sleep(60)
```

Save and exit Nano: `Ctrl+O`, `Enter`, then `Ctrl+X`.

Run the script:

```
python3 zombie.py
```

The parent process sleeps for 60 seconds while the child has already exited. During this interval, the child should appear as a zombie.

## 2. Find the zombie process

Open a second terminal and run:

```
ps -eo pid,ppid,stat,comm | awk 'NR == 1 || $3 ~ /^Z/'
```

Example output:

```
    PID    PPID STAT COMMAND
   4210    4209 Z    python3
```

The `Z` state identifies the zombie.

You can also inspect all processes with:

```
ps aux
```

Look for `Z` in the `STAT` column or `<defunct>` in the command display.

## 3. Understand the process relationship

Parent process

Still running; sleeping for 60 seconds

Child process — `Z`

Exited, but not yet reaped by its parent

## 4. Verify the zombie's details

Use the child PID printed by your script:

```
ps -o pid,ppid,stat,cmd -p CHILD_PID
```

Replace `CHILD_PID` with the actual number.

- `PID`: the child's process ID.
- `PPID`: the parent's process ID.
- `STAT`: `Z` indicates a zombie.
- `CMD`: may display `python3` or a defunct-process marker, depending on the `ps` output.

## 5. Clean up the test

The script's parent sleeps for 60 seconds. When it exits, the orphaned zombie is normally adopted by a system reaper such as `systemd`, which collects its exit status.

You can wait for the parent to finish naturally. Check whether the zombie remains:

```
ps -eo pid,ppid,stat,comm | awk 'NR == 1 || $3 ~ /^Z/'
```

Important: Do not try to remove a zombie by sending it `SIGKILL`. The child has already exited. The parent must reap it, or the parent must exit so the system can arrange reaping.

## 6. How to prevent zombie processes in real programs

In Python, wait for the child process:

```
import ospid = os.fork()if pid == 0:    os._exit(0)else:    os.waitpid(pid, 0)
```

Here, `os.waitpid(pid, 0)` collects the child's exit status, preventing the child from remaining a zombie.

Key takeaway: `Z` means a process has terminated but its parent has not yet collected its exit status. A brief zombie during normal process cleanup is common; a large or persistent number may indicate a parent process that is not handling child exits correctly.
