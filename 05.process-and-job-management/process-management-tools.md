# Purpose of Linux Process Management Tools

Linux provides these commands to view, monitor, search, control, and manage running processes. They are essential for Linux administration, troubleshooting, and system performance monitoring.

## 1. Overview of the tools

| Command  | Purpose                                                   | Typical use                                            |
| -------- | --------------------------------------------------------- | ------------------------------------------------------ |
| `ps`     | Displays information about running processes              | View process IDs (PIDs) and status                     |
| `top`    | Monitors processes and system resource usage in real time | Identify high CPU or memory usage                      |
| `pgrep`  | Searches for processes by name or other attributes        | Find the PID of a process                              |
| `kill`   | Sends a signal to a process                               | Stop or control a process                              |
| `renice` | Changes the scheduling priority of a running process      | Give a process lower or higher CPU scheduling priority |
| `pstree` | Displays processes in a hierarchical tree                 | Understand parent-child process relationships          |
| `htop`   | Interactive, user-friendly process monitor                | Monitor and manage processes visually in the terminal  |

## 2. Explanation with examples

1\. `ps` — Process Status

Displays a snapshot of currently running processes.

```
ps
ps aux
ps -ef
```

- `ps`: Shows processes associated with your terminal.
- `ps aux`: Shows processes across the system, including CPU and memory usage.
- `ps -ef`: Shows processes in a full-format listing.

Example: Find which processes are running before troubleshooting a service.

2\. `top` — Real-Time Process Monitor

Displays running processes and updates system statistics continuously.

```
top
```

It shows CPU usage, memory usage, load averages, and process information.

- Press `P` to sort by CPU usage.
- Press `M` to sort by memory usage.
- Press `q` to quit.

Example: Identify a process consuming excessive CPU.

3\. `pgrep` — Process Search

Finds process IDs by matching process names or other criteria.

```
pgrep nginx
pgrep -a python
pgrep -u root
```

- `pgrep nginx`: Finds PIDs matching `nginx`.
- `pgrep -a python`: Shows matching PIDs and command lines.
- `pgrep -u root`: Finds processes owned by root.

Example: Find the PID of an application without listing every process.

4\. `kill` — Send a Signal to a Process

Sends a signal to a process, often to request that it terminate.

```
kill 1234
kill -TERM 1234
kill -KILL 1234
```

- `kill 1234`: Normally sends `SIGTERM`, requesting graceful termination.
- `kill -TERM 1234`: Explicitly requests graceful termination.
- `kill -KILL 1234`: Forces termination; the process cannot catch or ignore this signal.

Replace `1234` with the actual PID.

Important: Use `SIGKILL` only when necessary because the process cannot clean up before exiting. You may need appropriate permissions.

5\. `renice` — Change Process Priority

Changes a process's nice value, which influences CPU scheduling priority.

```
renice 10 -p 1234
renice -n 5 -p 1234
```

The nice-value range is typically `-20` to `19`.

- `-20`: Highest scheduling priority.
- `0`: Default nice value.
- `19`: Lowest scheduling priority.

Example: Reduce the scheduling priority of a CPU-intensive background task.

Note: Raising priority by assigning a lower nice value generally requires elevated privileges. Nice values influence scheduling; they do not guarantee CPU time.

6\. `pstree` — Process Tree

Displays processes as a hierarchical tree, showing parent-child relationships.

```
pstree
pstree -p
pstree -p 1
```

- `pstree`: Shows the process hierarchy.
- `pstree -p`: Includes process IDs.
- `pstree -p 1`: Shows the process tree rooted at PID 1, where supported.

Example: Trace which process launched a shell, script, or service.

7\. `htop` — Interactive Process Monitor

An enhanced, interactive alternative to `top`, when installed.

```
htop
```

It provides color-coded CPU and memory meters, searchable processes, sorting, and interactive process management.

- `F3`: Search for a process.
- `F6`: Choose a sort column.
- `F9`: Select a signal to send to a process.
- `F10`: Quit.

Example: Find a process using too much memory and terminate it interactively.

Install on Ubuntu or Debian if needed:

```
sudo apt update
sudo apt install htop
```

## 3. How these tools work together

Consider a Linux server where a Python application is consuming excessive CPU.

<img width="642" height="398" alt="image" src="https://github.com/user-attachments/assets/ec15c68e-0b22-4338-937d-9bd491be3fae" />

Use `pstree -p` when you need to understand how the process relates to its parent and child processes.

## 4. Quick memory guide

| If you want to…                  | Use      |
| -------------------------------- | -------- |
| List processes                   | `ps`     |
| Monitor system activity live     | `top`    |
| Find a process by name           | `pgrep`  |
| Stop a process                   | `kill`   |
| Adjust CPU scheduling priority   | `renice` |
| Understand process relationships | `pstree` |
| Manage processes interactively   | `htop`   |

Recommended learning order: `ps` → `top` → `pgrep` → `kill` → `renice` → `pstree` → `htop`.

These tools cover the core of Linux process management. Practise them in a Linux virtual machine using ordinary user-owned processes first, especially before experimenting with `kill` and process priorities.
