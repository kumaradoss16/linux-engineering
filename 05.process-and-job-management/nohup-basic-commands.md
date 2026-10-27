# Basic `nohup` Commands in Linux

The `nohup` command runs a program so that it ignores the hangup signal (`SIGHUP`), allowing it to continue running when you close the terminal or disconnect from a session in typical circumstances.

It is commonly used to run background scripts, long-running Python programs, and administrative tasks.

## 1. Basic `nohup` commands

| Command                             | Purpose                                                                       |
| ----------------------------------- | ----------------------------------------------------------------------------- |
| `nohup command &`                   | Runs a command while ignoring hangup signals and places it in the background. |
| `nohup python3 app.py &`            | Runs a Python script in the background.                                       |
| `nohup command > output.log 2>&1 &` | Saves standard output and errors to a log file.                               |
| `nohup command > /dev/null 2>&1 &`  | Discards standard output and errors.                                          |
| `jobs`                              | Lists background jobs in the current shell.                                   |
| `pgrep -af app.py`                  | Searches for matching processes and displays their command lines.             |
| `kill PID`                          | Requests termination of a process.                                            |

## 2. Practical examples

### Example 1: Run a command with `nohup`

```
nohup sleep 300 &
```

- `nohup`: Ignores `SIGHUP`.
- `sleep 300`: Runs for 300 seconds.
- `&`: Starts the command asynchronously in the background.

By default, output is redirected to `nohup.out` when appropriate.

### Example 2: Run a Python script in the background

```
nohup python3 process-lab/signal_demo.py &
```

Use this when you want a script to continue running after you disconnect from the terminal.

### Example 3: Save output to a log file

```
nohup python3 app.py > app.log 2>&1 &
```

Explanation:

| Syntax    | Meaning                                                              |
| --------- | -------------------------------------------------------------------- |
| `>`       | Redirects standard output (stdout).                                  |
| `app.log` | File that stores the output.                                         |
| `2>&1`    | Redirects standard error (stderr) to the same destination as stdout. |
| `&`       | Runs the command in the background.                                  |

This is generally the most useful pattern for long-running scripts because you can inspect their output later.

### Example 4: View the log while the program runs

```
tail -f app.log
```

Press `Ctrl+C` to stop following the log. This normally stops `tail`, not the background application.

### Example 5: Find the running process

```
pgrep -af 'python3 app.py'
```

This searches for matching command lines. Replace the pattern with the command you actually launched.

### Example 6: Stop the process

First identify the correct PID:

```
pgrep -af 'python3 app.py'
```

Then request graceful termination:

```
kill PID
```

Replace `PID` with the actual process ID. By default, `kill` sends `SIGTERM`.

## 3. Complete practical workflow

Try this safe example in your Linux virtual machine.

Step 1 — Start a background task

```
nohup sleep 600 > sleep.log 2>&1 &
```

Step 2 — Find the process

```
pgrep -af 'sleep 600'
```

Step 3 — Inspect the log

```
cat sleep.log
```

The log may be empty because `sleep` produces no output.

Step 4 — Stop the task

Find the PID using `pgrep`, then run:

```
kill PID
```

## 4. `nohup` vs `&` vs `tmux`

| Feature                     | `nohup`                        | `&`               | `tmux`                                               |
| --------------------------- | ------------------------------ | ----------------- | ---------------------------------------------------- |
| Runs in background          | Usually when combined with `&` | Yes               | Can run processes in a separate session              |
| Ignores `SIGHUP`            | Yes                            | No, not by itself | Normally preserves processes in its detached session |
| Reattach to terminal        | No                             | No                | Yes                                                  |
| Interactive terminal access | No, not after disconnecting    | Not by itself     | Yes                                                  |

Important: `nohup` does not guarantee that a program survives reboots, system shutdowns, forced termination, or every session-management policy. For persistent production services, use `systemd`; for interactive development sessions, use `tmux`.
