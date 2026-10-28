# Linux Processes & Job Management: From Beginner to Server Administration

## Introduction

A Linux server is a collection of running processes: shells, services, applications, schedulers, and kernel worker threads. Process management means finding the right process, understanding its state and resource use, changing its priority safely, sending the correct signal, and knowing when not to act.

Run all labs only on a disposable Ubuntu 22.04/24.04, Debian, Fedora, Rocky, AlmaLinux, or RHEL VM/container. Take a VM snapshot before starting. Do not run process-termination, priority, or service-management exercises on production systems.

Commands that modify process state require appropriate permissions. Your PIDs, usernames, timestamps, load values, and output will differ from the examples.

## Lab safety

Create a lab directory:

```bash
mkdir -p ~/process-lab
cd ~/process-lab
```

Check your identity and available tools:

```bash
whoami
id
command -v ps top pgrep kill renice pstree htop
```

Safety rules:

- Always identify the exact PID and command before sending a signal.
- Prefer `SIGTERM` before `SIGKILL`.
- Never broadly use `pkill python`, `pkill java`, or similar patterns.
- Do not send `SIGKILL` to databases or critical services unless the incident procedure authorizes it.
- Do not kill unknown processes before collecting evidence.
- A process in `D` state may be waiting on hardware or kernel I/O and may not terminate normally.
- Renice and negative nice values can affect service performance.

***

# Part 1: Process Fundamentals

## Program versus process

A **program** is a stored executable file, such as `/usr/bin/python3`, `/usr/sbin/sshd`, or a shell script. A **process** is a running instance of that program, with a PID, memory, file descriptors, environment, security identity, and scheduling state.

One program can create multiple processes.

Examples:

```bash
ps -C bash -o pid,ppid,user,stat,cmd
ps -C python3 -o pid,ppid,user,stat,cmd
ps -C sshd -o pid,ppid,user,stat,cmd
ps -C systemd -o pid,ppid,user,stat,cmd
```

Example output; your values will differ:

```text
    PID    PPID USER     STAT COMMAND
   1234    1230 student  Ss   -bash
   4567    4560 student  S    python3 process-lab/signal_demo.py
```

## PID and PPID

- **PID**: Process ID, a unique identifier for a running process.
- **PPID**: Parent process ID, the PID of the process that created it.
- **PID 1**: Usually `systemd` on modern Linux systems; it is the initial userspace process and commonly reparents orphaned processes.

Inspect a process through `/proc`:

```bash
pid=$(pgrep -f 'sleep 600' | head -n 1)
echo "$pid"
cat "/proc/$pid/status"
cat "/proc/$pid/cmdline"; printf '\n'
ls "/proc/$pid"
```

Example output:

```text
Name:   sleep
State:  S (sleeping)
Pid:    4821
PPid:   1234
Uid:    1000    1000    1000    1000
```

`/proc/PID/cmdline` uses NUL separators between arguments, so it may appear to run together without a final newline.

## Process tree

A process tree shows ancestry. A shell can start a script; the script can start child processes; those children may start further children.

Example:

```text
systemd
└── sshd
    └── bash
        └── python3 cpu_load.py
            ├── python3 cpu_load.py
            └── python3 cpu_load.py
```

Visualize it:

```bash
pstree -p
pstree -p "$$"
pstree -ap | head -n 50
```

The shell variable `$$` contains the current shell PID.

## Process states

| State | Name | Meaning |
|---|---|---|
| `R` | Running/runnable | Executing or waiting for CPU |
| `S` | Interruptible sleep | Waiting for an event, I/O, timer, or signal |
| `D` | Uninterruptible sleep | Waiting on kernel/hardware I/O; cannot be killed normally |
| `T` | Stopped | Paused by `SIGSTOP`, `Ctrl+Z`, or a debugger |
| `Z` | Zombie | Terminated but not yet reaped by parent |
| `I` | Idle kernel thread | Idle kernel worker |

### Zombie process

A zombie is a process that has exited but whose parent has not read its exit status. It consumes a PID-table entry but not CPU time or normal process memory.

A zombie cannot be “killed” because it is already dead. Fix the parent process or restart the service that failed to reap it.

### Orphan process

An orphan is a process whose parent exited first. Linux normally reparents it to PID 1, typically `systemd`.

This is normal for daemons, detached jobs, and processes started with `nohup`.

### Daemon process

A daemon is a long-running background service process, often started by systemd. Examples include `sshd`, `nginx`, `cron`, and `systemd-journald`.

## CPU scheduling

The kernel scheduler decides which runnable process receives CPU time. A process may be runnable but waiting if more work exists than CPU capacity.

### Load average

Load average approximates runnable or uninterruptible work over 1, 5, and 15 minutes.

```bash
uptime
```

Example output:

```text
 09:30:00 up 2:15, 1 user, load average: 2.10, 1.80, 1.20
```

Interpret load relative to CPU count:

```bash
nproc
```

A load of `2.10` on a 4-core machine is generally less concerning than the same load on a 1-core machine. Load is not a direct CPU-percentage measure because it can include uninterruptible I/O wait.

### CPU time versus wall-clock time

- **CPU time**: Time spent executing on CPU.
- **Wall-clock time**: Elapsed real-world time.
- **User CPU**: Time spent in application code.
- **System CPU**: Time spent in kernel code.
- **Context switching**: The kernel switching CPU execution between processes or threads.

Inspect CPU and memory:

```bash
top -b -n 1 | head -n 20
```

## Foreground and background processes

A foreground process is connected to the terminal and receives keyboard input. A background process runs without occupying the terminal prompt.

Start a background process:

```bash
sleep 300 &
jobs -l
```

Example output:

```text
 [docs.kernel](https://docs.kernel.org/process/threat-model.html)+  4821 Running                 sleep 300 &
```

Stop a foreground job:

```bash
sleep 300
```

Press:

```text
Ctrl+Z
```

Then inspect and resume it:

```bash
jobs -l
bg %1
fg %1
```

When a terminal closes, background jobs may receive `SIGHUP`. Use `nohup` or `disown` when a job must survive logout.

```bash
nohup ./long_task.sh > long-task.log 2>&1 &
disown -h %1
```

***

# Part 2: Core Commands

## `ps`

### Purpose

Displays a snapshot of running processes.

### Syntax

```bash
ps [options]
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `a` | Show processes for all users | `ps a` |
| `u` | User-oriented output | `ps au` |
| `x` | Include processes without a terminal | `ps ax` |
| `-e` | Show every process | `ps -e` |
| `-f` | Full-format listing | `ps -ef` |
| `-F` | Extra full format | `ps -eF` |
| `-p PID` | Select a specific PID | `ps -p 1234` |
| `-u USER` | Select by user | `ps -u alice` |
| `-C NAME` | Select by command name | `ps -C nginx` |
| `--forest` | Show process tree | `ps -ef --forest` |
| `--sort` | Sort output | `ps aux --sort=-%cpu` |
| `-o` | Custom output columns | `ps -eo pid,cmd` |

### Beginner example

```bash
ps
ps aux | head -n 10
ps -ef --forest | head -n 20
```

### Practical server-administration examples

Highest CPU consumers:

```bash
ps aux --sort=-%cpu | head -n 15
```

Highest memory consumers:

```bash
ps aux --sort=-%mem | head -n 15
```

Inspect one process:

```bash
ps -p 1234 -o pid,ppid,user,stat,%cpu,%mem,etime,cmd
```

Inspect a user’s processes:

```bash
ps -u alice -o pid,ppid,stat,%cpu,%mem,cmd
```

Inspect a service:

```bash
ps -C nginx -o pid,ppid,user,stat,cmd
```

### Reading output

```text
USER  PID  %CPU %MEM STAT COMMAND
alice 4821 92.3 1.2 R    python3 cpu_load.py
```

| Column | Meaning |
|---|---|
| `USER` | Effective user |
| `PID` | Process ID |
| `%CPU` | CPU utilization over the process lifetime or recent interval, depending on `ps` implementation |
| `%MEM` | Resident memory as a percentage of physical RAM |
| `STAT` | Process state |
| `COMMAND` | Command name or full command line |

### Common mistakes

- Treating `%CPU` as an instantaneous value.
- Confusing PID with job ID.
- Using `ps aux | grep process` when `pgrep` is safer.
- Killing a parent process without checking its children.

### Verification

```bash
ps -p "$pid" -o pid,ppid,user,stat,%cpu,%mem,etime,cmd
```

***

## `top`

### Purpose

Monitors processes and system resources in real time.

### Syntax

```bash
top [options]
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-d SECONDS` | Refresh interval | `top -d 2` |
| `-p PID` | Monitor a process | `top -p 1234` |
| `-u USER` | Filter by user | `top -u alice` |
| `-H` | Show threads | `top -H -p 1234` |
| `-b` | Batch mode | `top -b -n 1` |
| `-n ITERATIONS` | Number of refreshes | `top -b -n 1` |

### Interactive keys

| Key | Action |
|---|---|
| `h` | Help |
| `P` | Sort by CPU |
| `M` | Sort by memory |
| `T` | Sort by cumulative time |
| `k` | Kill a process |
| `r` | Renice a process |
| `u` | Filter by user |
| `1` | Show individual CPU cores |
| `q` | Quit |

### Beginner example

```bash
top
```

Press `q` to exit.

### Practical server-administration examples

Monitor one process:

```bash
top -p 1234
```

Take one noninteractive snapshot:

```bash
top -b -n 1 | head -n 25
```

Monitor a user:

```bash
top -u alice
```

### Reading top output

| Field | Meaning |
|---|---|
| Load average | Runnable/uninterruptible work over 1, 5, and 15 minutes |
| `%CPU` | Process CPU use |
| `%MEM` | Resident memory as a percentage of physical RAM |
| `VIRT` | Total virtual memory |
| `RES` | Resident physical memory |
| `SHR` | Shared memory |
| `TIME+` | Cumulative CPU time |
| `S` | Process state |
| `NI` | Nice value |

### Common mistakes

- Confusing `VIRT` with actual RAM use.
- Assuming high load always means high CPU.
- Killing from `top` without first confirming the PID and command.
- Forgetting that `top` can show threads when `-H` is used.

### Verification

```bash
top -b -n 1 -p "$pid"
```

***

## `htop`

### Purpose

Provides an enhanced interactive process monitor with easier navigation, filtering, sorting, and tree views.

### Install

Ubuntu/Debian:

```bash
sudo apt update
sudo apt install htop
```

RHEL/Rocky/Alma/Fedora:

```bash
sudo dnf install htop
```

### Important keys

| Key | Action |
|---|---|
| `F1` | Help |
| `F3` | Search |
| `F4` | Filter |
| `F5` | Tree view |
| `F6` | Sort |
| `F9` | Kill |
| `F10` | Quit |

### Beginner example

```bash
htop
```

### Practical server-administration examples

```bash
htop -u alice
htop -p 1234
```

### When htop is better

- Easier navigation and filtering.
- Color-coded resource use.
- Convenient tree view.
- Easier search and sorting.

`htop` is a monitoring aid, not a substitute for `ps`, logs, metrics, and incident evidence.

### Common mistakes

- Killing from `htop` without confirming the process.
- Assuming colors mean the same thing across versions and themes.
- Using it as the only source of evidence.

### Verification

```bash
ps -p "$pid" -o pid,user,stat,%cpu,%mem,cmd
```

***

## `pstree`

### Purpose

Displays the process hierarchy.

### Syntax

```bash
pstree [options] [PID or user]
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-p` | Show PIDs | `pstree -p` |
| `-a` | Show command-line arguments | `pstree -ap` |
| `-u` | Show user transitions | `pstree -u` |
| `-h` | Highlight current process | `pstree -h` |
| `-A` | Use ASCII tree characters | `pstree -A` |

### Beginner example

```bash
pstree -p "$$"
```

### Practical server-administration example

Inspect the ancestry of a workload:

```bash
pid=$(pgrep -f 'cpu_load.py' | head -n 1)
pstree -ap "$pid"
```

### Common mistakes

- Confusing sibling processes with child processes.
- Assuming a process tree remains stable.
- Searching the output with `grep` when `pgrep` is more reliable.

### Verification

```bash
ps -ef --forest
```

***

## `pgrep`

### Purpose

Finds process IDs by name, user, parent, or full command line.

### Syntax

```bash
pgrep [options] pattern
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-l` | Show process name | `pgrep -l nginx` |
| `-a` | Show full command line | `pgrep -a python3` |
| `-u USER` | Filter by user | `pgrep -u alice bash` |
| `-x` | Exact match | `pgrep -x bash` |
| `-f` | Match full command line | `pgrep -f "python3 cpu_load.py"` |
| `-P PPID` | Match children of a parent | `pgrep -P 1234` |
| `-n` | Newest match | `pgrep -n bash` |
| `-o` | Oldest match | `pgrep -o bash` |

### Beginner example

```bash
pgrep -a sleep
```

### Practical server-administration example

Find a specific workload before acting:

```bash
pgrep -af "python3 cpu_load.py"
```

### Why pgrep is safer

`pgrep` matches process attributes directly and avoids accidental matching of the `grep` command itself.

Compare:

```bash
ps aux | grep python
```

with:

```bash
pgrep -af python3
```

### Common mistakes

- Using broad patterns.
- Forgetting `-f` when searching a full command line.
- Assuming one match exists when multiple processes match.
- Using the result in `kill` without inspecting it.

### Verification

```bash
pgrep -af "cpu_load.py"
ps -fp "$(pgrep -f 'cpu_load.py' | tr '\n' ',' | sed 's/,$//')"
```

***

## `kill`

### Purpose

Sends a signal to a process. It does not always terminate it.

### Syntax

```bash
kill [options] PID
```

### Important signals

| Signal | Number | Typical purpose |
|---|---:|---|
| `SIGHUP` | 1 | Hangup; often used to reload configuration |
| `SIGINT` | 2 | Interrupt; usually `Ctrl+C` |
| `SIGQUIT` | 3 | Quit and optionally dump core |
| `SIGKILL` | 9 | Force termination; cannot be caught |
| `SIGTERM` | 15 | Graceful termination request |
| `SIGCONT` | 18 | Resume a stopped process |
| `SIGCHLD` | 17 | Child status changed |
| `SIGSTOP` | 19 | Pause a process |

### Examples

```bash
kill 1234
kill -15 1234
kill -TERM 1234
kill -9 1234
kill -KILL 1234
kill -STOP 1234
kill -CONT 1234
kill -l
```

### Safety rules

- Confirm the exact PID first.
- Use `SIGTERM` first.
- Wait for cleanup.
- Verify afterward.
- Use `SIGKILL` only when necessary.
- Never use `SIGKILL` casually against databases or critical services.

### Common mistakes

- Killing a PID without confirming its command.
- Sending `SIGKILL` immediately.
- Assuming `kill` can terminate a `D`-state process.
- Confusing shell job IDs with PIDs.

### Verification

```bash
kill -0 1234 2>/dev/null && echo "Process exists" || echo "Process absent"
pgrep -af "cpu_load.py"
```

***

## `pkill`

### Purpose

Sends signals to processes selected by name, user, parent, or command-line pattern.

### Syntax

```bash
pkill [options] pattern
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-l` | List matching names | `pkill -l nginx` |
| `-f` | Match full command line | `pkill -f "python3 cpu_load.py"` |
| `-u USER` | Match user | `pkill -u alice bash` |
| `-x` | Exact name match | `pkill -x nginx` |
| `-n` | Newest match | `pkill -n nginx` |
| `-o` | Oldest match | `pkill -o nginx` |
| `-SIGNAL` | Specify signal | `pkill -TERM nginx` |

### Beginner example

```bash
pgrep -l nginx
pkill -TERM nginx
```

### Practical server-administration example

Terminate a specific controlled workload:

```bash
pgrep -af "python3 cpu_load.py"
pkill -TERM -f "python3 cpu_load.py"
```

### Safety warning

`pkill` can match multiple processes.

Avoid:

```bash
pkill python
pkill java
pkill bash
```

Always inspect first:

```bash
pgrep -af pattern
```

### Common mistakes

- Using a broad pattern.
- Forgetting `-f` when the process name differs from the command line.
- Sending `SIGKILL` without trying `SIGTERM`.
- Matching your own shell or terminal process.

### Verification

```bash
pgrep -af "cpu_load.py"
```

***

## `killall`

### Purpose

Sends signals to processes with an exact command name.

### Syntax

```bash
killall [options] NAME
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-i` | Interactive confirmation | `killall -i firefox` |
| `-u USER` | Match user | `killall -u alice bash` |
| `-SIGNAL` | Specify signal | `killall -TERM nginx` |
| `-v` | Verbose | `killall -v nginx` |

### Beginner example

```bash
killall -i firefox
```

### Practical server-administration example

```bash
killall -TERM nginx
```

Prefer `systemctl restart nginx` when the service is managed by systemd.

### Difference

| Command | Selection method |
|---|---|
| `kill` | Exact PID |
| `pkill` | Name, user, parent, or command-line pattern |
| `killall` | Exact command name |

### Common mistakes

- Assuming `killall` only affects one process.
- Using it on a common command name.
- Confusing command name with full command line.

### Verification

```bash
pgrep -x nginx
```

***

## `jobs`

### Purpose

Displays jobs managed by the current shell.

### Syntax

```bash
jobs [options]
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-l` | Show PIDs | `jobs -l` |
| `-p` | Show only PIDs | `jobs -p` |
| `-r` | Show running jobs | `jobs -r` |
| `-s` | Show stopped jobs | `jobs -s` |

### Beginner example

```bash
sleep 300 &
jobs
jobs -l
jobs -r
jobs -s
```

### Practical server-administration example

Check whether a backup job is still running before closing a maintenance terminal:

```bash
jobs -l
```

### Common mistakes

- Confusing job ID `%1` with PID.
- Assuming `jobs` shows processes from another terminal.
- Closing the terminal before checking background jobs.

### Verification

```bash
jobs -l
```

***

## `fg` and `bg`

### Purpose

Move shell jobs between foreground and background.

### Syntax

```bash
fg [%JOB]
bg [%JOB]
```

### Beginner example

```bash
sleep 300
```

Press:

```text
Ctrl+Z
```

Then:

```bash
jobs
bg %1
fg %1
```

### Job ID versus PID

- `%1` is a shell job ID.
- The PID is the operating-system process identifier.

```bash
jobs -l
```

### Terminal closure

A normal background job may receive `SIGHUP` when its controlling terminal closes. To keep it running:

```bash
nohup ./long_task.sh > long-task.log 2>&1 &
disown -h %1
```

`nohup` ignores hangup signals. `disown -h` prevents the shell from sending `SIGHUP` to the job when the shell exits.

### Common mistakes

- Using `%1` with `kill`; use the PID instead.
- Forgetting output redirection with `nohup`.
- Assuming a background process automatically survives terminal closure.

### Verification

```bash
jobs -l
ps -fp "$(jobs -p | tr '\n' ',' | sed 's/,$//')"
```

***

## `nice`

### Purpose

Starts a process with a scheduling priority.

### Syntax

```bash
nice [options] [command]
```

### Nice values

| Nice value | Effect |
|---:|---|
| `-20` | Highest priority |
| `0` | Default |
| `19` | Lowest priority |

Higher nice value means lower scheduling priority. Lower nice value means higher scheduling priority.

Only root can normally set a negative nice value.

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-n VALUE` | Set nice value | `nice -n 10 backup.sh` |

### Beginner example

```bash
nice -n 10 sleep 300 &
ps -o pid,ni,cmd -p "$!"
```

### Practical server-administration examples

Run a backup at low priority:

```bash
nice -n 19 ./backup.sh
```

Run a batch report without competing aggressively for CPU:

```bash
nice -n 10 python3 report.py
```

Start an important service with higher priority:

```bash
sudo nice -n -5 /usr/local/bin/important_service
```

### Common mistakes

- Assuming nice alone prevents resource exhaustion.
- Using negative nice values without authorization.
- Forgetting that nice affects CPU scheduling, not disk or network limits.

### Verification

```bash
ps -o pid,ni,cmd -p "$!"
```

***

## `renice`

### Purpose

Changes the nice value of an already-running process.

### Syntax

```bash
renice [options] priority [-p PID] [-u USER]
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-n VALUE` | New nice value | `renice -n 10 -p 1234` |
| `-p PID` | Target process | `renice -n 10 -p 1234` |
| `-u USER` | Target all processes of a user | `sudo renice -n 10 -u alice` |
| `-g PGID` | Target process group | `renice -n 10 -g 1234` |

### Beginner example

```bash
sleep 300 &
pid=$!
renice -n 10 -p "$pid"
ps -o pid,ni,cmd -p "$pid"
```

### Practical server-administration example

Reduce the scheduling priority of a runaway report process:

```bash
pid=$(pgrep -f 'python3 report.py' | head -n 1)
renice -n 19 -p "$pid"
```

Increase priority only when authorized:

```bash
sudo renice -n -5 -p "$pid"
```

### Common mistakes

- Assuming renice fixes a memory leak or infinite loop.
- Lowering nice values without approval.
- Renicing every process for a user when only one process needs adjustment.
- Forgetting that normal users can usually increase niceness but not decrease it.

### Verification

```bash
ps -o pid,ni,cmd -p "$pid"
```

***

# Part 3: Resource Consumption

## CPU usage

CPU usage measures how much processor time a process consumes. High CPU is not automatically bad; a backup, compiler, or report may legitimately use CPU.

Find high-CPU processes:

```bash
ps aux --sort=-%cpu | head -n 15
top -b -n 1 | head -n 20
```

## Memory usage

Memory concepts:

| Term | Meaning |
|---|---|
| Virtual memory | Total address space reserved by a process |
| Resident memory | Physical RAM currently used |
| Shared memory | Memory shared with other processes |
| Swap | Disk-backed memory used when RAM pressure occurs |

Check system memory:

```bash
free -h
cat /proc/meminfo | head -n 20
```

Find high-memory processes:

```bash
ps aux --sort=-%mem | head -n 15
```

Inspect one process:

```bash
pid=$(pgrep -f 'cpu_load.py' | head -n 1)
cat "/proc/$pid/status" | grep -E '^(Name|State|Pid|PPid|VmRSS|VmSize|Threads)'
```

## Disk I/O and load

Disk I/O can cause high load without high CPU.

```bash
vmstat 1 5
uptime
du -sh /var/log
```

Example `vmstat` output:

```text
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 8123456 123456 2345678    0    0    12    45  120  240  8  2 88  2  0
```

High `wa` means significant I/O wait. High `b` can indicate blocked processes.

## OOM killer and limits

When memory is exhausted, the kernel may invoke the OOM killer and terminate a process. This can appear as an unexpected service restart.

Check logs:

Ubuntu/Debian:

```bash
sudo journalctl -k | grep -i -E 'oom|out of memory' | tail -n 50
sudo grep -i -E 'oom|out of memory' /var/log/kern.log 2>/dev/null | tail -n 50
```

RHEL-family:

```bash
sudo journalctl -k | grep -i -E 'oom|out of memory' | tail -n 50
sudo grep -i -E 'oom|out of memory' /var/log/messages 2>/dev/null | tail -n 50
```

Inspect process limits:

```bash
pid=$(pgrep -f 'cpu_load.py' | head -n 1)
cat "/proc/$pid/limits"
ulimit -a
```

## Identify problem processes

| Problem | Command |
|---|---|
| High CPU | `ps aux --sort=-%cpu \| head` |
| High memory | `ps aux --sort=-%mem \| head` |
| Long-running process | `ps -eo pid,user,etime,cmd --sort=-etime \| head` |
| Stopped process | `ps -eo pid,user,stat,cmd \| awk '$3 ~ /^T/'` |
| Zombie process | `ps -eo pid,ppid,stat,cmd \| awk '$3 ~ /^Z/'` |
| Unexpected process | `ps aux`, `pgrep -af pattern`, and inspect executable path |

***

# Part 4: Signals in Depth

Signals are notifications sent to processes.

| Signal | Number | Purpose | Can be caught? |
|---|---:|---|---|
| `SIGHUP` | 1 | Hangup/config reload | Usually yes |
| `SIGINT` | 2 | Interrupt, usually `Ctrl+C` | Usually yes |
| `SIGQUIT` | 3 | Quit and dump core | Usually yes |
| `SIGKILL` | 9 | Force termination | No |
| `SIGTERM` | 15 | Graceful termination request | Usually yes |
| `SIGCONT` | 18 | Resume stopped process | No |
| `SIGCHLD` | 17 | Child process status changed | Usually yes |
| `SIGSTOP` | 19 | Pause process | No |

## SIGINT versus SIGTERM versus SIGKILL

| Signal | Typical source | Cleanup opportunity | Use |
|---|---|---|---|
| `SIGINT` | `Ctrl+C` | Usually yes | Stop an interactive foreground job |
| `SIGTERM` | `kill`, `pkill` | Usually yes | Preferred termination request |
| `SIGKILL` | Administrator | No | Last resort |

## SIGSTOP and SIGCONT

Pause a process for investigation:

```bash
kill -STOP 1234
ps -p 1234 -o pid,stat,cmd
```

Resume it:

```bash
kill -CONT 1234
```

`SIGSTOP` is useful for temporarily freezing a noncritical process, but do not leave a production service stopped.

## Sending signals

```bash
kill -TERM PID
pkill -TERM -f "python3 cpu_load.py"
killall -TERM process-name
```

In `top`, press `k`, enter the PID, then choose a signal.

***

# Part 5: Daemons, Zombies, and Orphans

## Daemons

Daemons are background service processes.

Inspect common services:

```bash
systemctl status ssh 2>/dev/null || systemctl status sshd 2>/dev/null || true
systemctl status nginx 2>/dev/null || true
ps -C sshd -o pid,ppid,user,stat,cmd
pgrep -a sshd
pgrep -a nginx
```

Ubuntu commonly uses `ssh.service`; RHEL-family systems commonly use `sshd.service`.

## Zombie processes

Find zombies:

```bash
ps aux | awk '$8 ~ /^Z/ {print}'
ps -eo pid,ppid,stat,cmd | awk '$3 ~ /^Z/ {print}'
```

Inspect the parent:

```bash
zombie_pid=$(ps -eo pid,stat --no-headers | awk '$2 ~ /^Z/ {print $1; exit}')
if [[ -n "$zombie_pid" ]]; then
  ppid=$(ps -o ppid= -p "$zombie_pid" | tr -d ' ')
  ps -fp "$ppid"
fi
```

Fix the parent or restart the affected application. Killing the zombie itself does not solve the underlying reaping problem.

## Orphan processes

Find long-running processes:

```bash
ps -eo pid,ppid,user,stat,etime,cmd --sort=-etime | head -n 20
```

An orphan may be normal if it is a daemon or detached job. Investigate if it is unexpected, privileged, or consuming resources.

## Long-running processes

Check elapsed time:

```bash
ps -eo pid,user,etime,cmd --sort=-etime | head -n 20
```

Example output:

```text
  PID USER     ELAPSED COMMAND
  987 root     12-03:22:11 /usr/sbin/sshd -D
 1234 student 01:22:10 python3 process-lab/signal_demo.py
```

***

# Part 6: Real-World Server Use Cases

## High CPU usage

1. Identify the process:

```bash
ps aux --sort=-%cpu | head -n 15
```

2. Confirm ownership and command:

```bash
pid=$(ps aux --sort=-%cpu | awk 'NR==2 {print $2}')
ps -fp "$pid"
pstree -ap "$pid"
```

3. Determine whether it is expected.

4. Reduce priority if safe:

```bash
renice -n 19 -p "$pid"
```

5. Stop or terminate only after approval:

```bash
kill -TERM "$pid"
```

6. Restart the affected service if needed:

```bash
sudo systemctl restart service-name
```

## High memory usage

1. Identify the consumer:

```bash
ps aux --sort=-%mem | head -n 15
free -h
```

2. Inspect memory details:

```bash
pid=$(ps aux --sort=-%mem | awk 'NR==2 {print $2}')
grep -E '^(Name|State|VmRSS|VmSize|Threads)' "/proc/$pid/status"
```

3. Check swap and OOM logs:

```bash
free -h
sudo journalctl -k | grep -i -E 'oom|out of memory' | tail -n 50
```

4. Review application logs and metrics.

5. Restart only when approved and safe:

```bash
sudo systemctl restart service-name
```

## Unresponsive application

1. Check state:

```bash
pid=$(pgrep -f 'application-name' | head -n 1)
ps -p "$pid" -o pid,ppid,user,stat,%cpu,%mem,etime,cmd
```

2. Send `SIGTERM`:

```bash
kill -TERM "$pid"
```

3. Wait and verify:

```bash
sleep 10
pgrep -af 'application-name'
```

4. Escalate to `SIGKILL` only if required:

```bash
kill -KILL "$pid"
```

5. Restart through systemd:

```bash
sudo systemctl restart service-name
```

## Suspicious process

Collect evidence before killing:

```bash
pid=$(pgrep -f 'suspicious-pattern' | head -n 1)
ps -fp "$pid"
cat "/proc/$pid/cmdline"; printf '\n'
readlink -f "/proc/$pid/exe"
ls -l "/proc/$pid/exe"
pstree -ap "$pid"
```

Check network activity if authorized tools are installed:

```bash
sudo ss -tunap | grep "pid=$pid"
```

Do not immediately kill the process if forensic preservation matters. Isolate the host or stop the service according to the incident-response plan.

## Background job management

Start a long backup:

```bash
nohup ./backup.sh > backup.log 2>&1 &
disown -h %1
```

Check it:

```bash
jobs -l
pgrep -af backup.sh
tail -f backup.log
```

Monitor CPU:

```bash
pid=$(pgrep -f 'backup.sh' | head -n 1)
top -p "$pid"
```

***

# Hands-On Labs

# Lab 7.1 - Process Investigation

## Objective

Identify PID, PPID, CPU usage, memory usage, command, and process state.

## Task

1. Start `sleep 600`.
2. From another terminal, find and inspect it.
3. View its process tree.
4. Monitor it with `top`.
5. Inspect `/proc`.
6. Record PID, PPID, CPU, memory, command, state, and parent.

## Solution

In terminal 1:

```bash
sleep 600
```

In terminal 2:

```bash
pgrep -a sleep
```

Example output:

```text
4821 sleep 600
```

Inspect it:

```bash
pid=$(pgrep -f '^sleep 600$' | head -n 1)
ps -p "$pid" -o pid,ppid,user,stat,%cpu,%mem,etime,cmd
```

Example output; your values will differ:

```text
    PID    PPID USER     STAT %CPU %MEM     ELAPSED COMMAND
   4821    1234 student  S     0.0  0.0       00:01:12 sleep 600
```

View the process tree:

```bash
pstree -p "$pid"
pstree -p
```

Monitor it:

```bash
top -p "$pid"
```

Inspect `/proc`:

```bash
cat "/proc/$pid/status"
cat "/proc/$pid/cmdline"; printf '\n'
ls "/proc/$pid"
```

Record:

```text
PID: 4821
PPID: 1234
CPU: near 0%
Memory: near 0%
Command: sleep 600
State: S
Parent: interactive Bash shell
```

## Expected-results checklist

- [ ] The sleep process has a PID and PPID.
- [ ] CPU usage is near zero.
- [ ] State is usually `S`.
- [ ] Parent is the shell or terminal process.
- [ ] `/proc/PID/cmdline` shows `sleep 600`.

***

# Lab 7.2 - Signal Management

## Objective

Practice `SIGINT`, `SIGTERM`, `SIGKILL`, `SIGSTOP`, and `SIGCONT` safely.

## Task

Create a Python test process, then pause, resume, terminate, and verify it.

## Solution

Create the script:

```bash
cd ~/process-lab
cat > signal_demo.py <<'EOF'
import time
while True:
    print("Process running...", flush=True)
    time.sleep(2)
EOF
```

Run it in the foreground:

```bash
python3 signal_demo.py
```

Press:

```text
Ctrl+C
```

`Ctrl+C` sends `SIGINT`.

Start it in the background:

```bash
python3 signal_demo.py > signal-demo.log 2>&1 &
```

Find its PID:

```bash
pid=$(pgrep -f 'python3 signal_demo.py' | head -n 1)
echo "$pid"
```

Pause it:

```bash
kill -STOP "$pid"
```

Verify:

```bash
ps -p "$pid" -o pid,stat,cmd
```

Expected state:

```text
T
```

Resume it:

```bash
kill -CONT "$pid"
ps -p "$pid" -o pid,stat,cmd
```

Expected state:

```text
S
```

Send graceful termination:

```bash
kill -TERM "$pid"
sleep 2
pgrep -af 'signal_demo.py' || echo "Process terminated"
```

If it remains, force kill:

```bash
kill -KILL "$pid"
pgrep -af 'signal_demo.py' || echo "Process removed"
```

## Expected-results checklist

- [ ] `Ctrl+C` stops the foreground process.
- [ ] `SIGSTOP` pauses the process.
- [ ] `SIGCONT` resumes it.
- [ ] `SIGTERM` requests graceful shutdown.
- [ ] `SIGKILL` forcibly removes it.
- [ ] `pgrep` confirms the final state.

***

# Lab 7.3 - High CPU Simulation

## Objective

Create a controlled CPU workload, investigate it, reduce its priority, and terminate it safely.

## Task

Run a two-process CPU workload, identify it, renice it, terminate it, and verify CPU recovery.

## Solution

Create the workload:

```bash
cd ~/process-lab
cat > cpu_load.py <<'EOF'
import multiprocessing
import time

def burn_cpu():
    while True:
        pass

if __name__ == "__main__":
    processes = []
    for _ in range(2):
        p = multiprocessing.Process(target=burn_cpu)
        p.start()
        processes.append(p)

    time.sleep(120)

    for p in processes:
        p.terminate()
EOF
```

Run it:

```bash
python3 cpu_load.py > cpu-load.log 2>&1 &
```

Investigate:

```bash
top
htop
ps aux --sort=-%cpu | head -n 15
pgrep -af cpu_load.py
pstree -p | grep -A2 -B2 cpu_load
```

Identify the parent:

```bash
parent_pid=$(pgrep -f '^python3 cpu_load.py$' | head -n 1)
echo "Parent PID: $parent_pid"
pgrep -P "$parent_pid" -a
```

Reduce priority:

```bash
renice -n 19 -p "$parent_pid"
ps -o pid,ni,stat,%cpu,cmd -p "$parent_pid"
```

Terminate gracefully:

```bash
kill -TERM "$parent_pid"
sleep 3
pgrep -af cpu_load.py || echo "Workload stopped"
```

If needed, force kill remaining child processes:

```bash
for pid in $(pgrep -f 'cpu_load.py'); do
  kill -KILL "$pid"
done
```

Verify:

```bash
pgrep -af cpu_load.py || echo "No cpu_load processes remain"
top -b -n 1 | head -n 15
```

## Expected-results checklist

- [ ] CPU usage rises during simulation.
- [ ] The workload is identified by command and PID.
- [ ] Parent and child processes are distinguished.
- [ ] Renice reduces scheduling priority.
- [ ] `SIGTERM` stops the controlled workload.
- [ ] CPU usage returns to normal.

***

# Part 7: Final Project - ProcWatch

## Project goal

Build **ProcWatch**, a Python command-line tool that monitors Linux processes and detects high CPU, high memory, long-running processes, stopped processes, zombies, and suspicious processes.

## Project structure

```text
procwatch/
├── procwatch/
│   ├── __init__.py
│   ├── cli.py
│   ├── config.py
│   ├── collector.py
│   ├── detectors/
│   │   ├── cpu_detector.py
│   │   ├── memory_detector.py
│   │   ├── duration_detector.py
│   │   ├── state_detector.py
│   │   └── suspicious_detector.py
│   ├── alerts/
│   │   ├── console_alert.py
│   │   ├── file_alert.py
│   │   └── webhook_alert.py
│   ├── reports/
│   │   ├── terminal_report.py
│   │   ├── json_report.py
│   │   ├── csv_report.py
│   │   └── markdown_report.py
│   └── utils/
│       ├── time_utils.py
│       ├── process_utils.py
│       └── config_utils.py
├── tests/
│   ├── test_cpu_detector.py
│   ├── test_memory_detector.py
│   ├── test_state_detector.py
│   └── test_suspicious_detector.py
├── docs/
│   ├── 01-overview.md
│   ├── 02-installation.md
│   ├── 03-usage.md
│   ├── 04-detection-rules.md
│   ├── 05-alerting.md
│   └── 06-troubleshooting.md
├── reports/
├── requirements.txt
├── pyproject.toml
└── README.md
```

## Install dependencies

```bash
sudo apt update
sudo apt install python3 python3-venv python3-pip
python3 -m venv ~/process-lab/procwatch-venv
source ~/process-lab/procwatch-venv/bin/activate
python -m pip install --upgrade pip
python -m pip install psutil pytest
```

RHEL/Rocky/Alma/Fedora:

```bash
sudo dnf install python3 python3-pip
python3 -m venv ~/process-lab/procwatch-venv
source ~/process-lab/procwatch-venv/bin/activate
python -m pip install --upgrade pip
python -m pip install psutil pytest
```

## Create the project

```bash
mkdir -p ~/process-lab/procwatch/procwatch/{detectors,alerts,reports,utils}
mkdir -p ~/process-lab/procwatch/{tests,docs,reports}
cd ~/process-lab/procwatch
```

### `requirements.txt`

```text
psutil>=5.9.0
pytest>=7.0.0
```

### `procwatch/__init__.py`

```python
"""ProcWatch: Linux process monitoring and alerting."""

__version__ = "1.0.0"
```

### `procwatch/config.py`

```python
from dataclasses import dataclass, field
from pathlib import Path


@dataclass
class ProcWatchConfig:
    cpu_threshold: float = 80.0
    memory_threshold_mb: float = 500.0
    memory_percent_threshold: float = 70.0
    long_running_hours: float = 24.0
    interval_seconds: int = 30
    suspicious_paths: tuple[str, ...] = (
        "/tmp",
        "/var/tmp",
        "/dev/shm",
    )
    excluded_names: tuple[str, ...] = field(default_factory=tuple)
    root_allowlist: tuple[str, ...] = field(default_factory=tuple)
    custom_suspicious_patterns: tuple[str, ...] = field(default_factory=tuple)
    report_directory: Path = Path("reports")
    alert_log: Path = Path("reports/procwatch-alerts.log")

    def is_excluded(self, name: str) -> bool:
        return name in self.excluded_names
```

### `procwatch/collector.py`

```python
import logging
from dataclasses import dataclass
from datetime import datetime, timezone

import psutil

from .config import ProcWatchConfig


@dataclass
class ProcessRecord:
    pid: int
    ppid: int
    name: str
    cmdline: str
    username: str
    cpu_percent: float
    memory_percent: float
    rss_mb: float
    state: str
    create_time: datetime
    elapsed_seconds: float
    nice: int
    parent_name: str
    child_count: int
    executable: str


class ProcessCollector:
    def __init__(self, config: ProcWatchConfig):
        self.config = config

    def collect(self) -> list[ProcessRecord]:
        records = []
        processes = {p.pid: p for p in psutil.process_iter()}

        for process in processes.values():
            try:
                if self.config.is_excluded(process.name()):
                    continue

                with process.oneshot():
                    create_time = datetime.fromtimestamp(
                        process.create_time(), tz=timezone.utc
                    )
                    memory_info = process.memory_info()
                    parent = process.parent()
                    children = process.children()

                    records.append(
                        ProcessRecord(
                            pid=process.pid,
                            ppid=process.ppid(),
                            name=process.name(),
                            cmdline=" ".join(process.cmdline()),
                            username=process.username(),
                            cpu_percent=process.cpu_percent(interval=None),
                            memory_percent=process.memory_percent(),
                            rss_mb=memory_info.rss / (1024 * 1024),
                            state=process.status(),
                            create_time=create_time,
                            elapsed_seconds=max(
                                0.0,
                                (datetime.now(timezone.utc) - create_time).total_seconds(),
                            ),
                            nice=process.nice(),
                            parent_name=parent.name() if parent else "",
                            child_count=len(children),
                            executable=process.exe() or "",
                        )
                    )
            except (psutil.NoSuchProcess, psutil.AccessDenied, psutil.ZombieProcess):
                continue

        return records
```

### `procwatch/detectors/cpu_detector.py`

```python
from dataclasses import dataclass

from ..collector import ProcessRecord


@dataclass
class Detection:
    rule: str
    severity: str
    pid: int
    user: str
    evidence: str
    process: ProcessRecord


class CpuDetector:
    def __init__(self, threshold: float):
        self.threshold = threshold

    def detect(self, records: list[ProcessRecord]) -> list[Detection]:
        findings = []

        for record in records:
            if record.cpu_percent > self.threshold:
                findings.append(
                    Detection(
                        rule="high_cpu",
                        severity="WARNING",
                        pid=record.pid,
                        user=record.username,
                        evidence=f"CPU {record.cpu_percent:.1f}% exceeds {self.threshold:.1f}%",
                        process=record,
                    )
                )

        return findings
```

### `procwatch/detectors/memory_detector.py`

```python
from dataclasses import dataclass

from ..collector import ProcessRecord


@dataclass
class MemoryDetection:
    rule: str
    severity: str
    pid: int
    user: str
    evidence: str
    process: ProcessRecord


class MemoryDetector:
    def __init__(self, threshold_mb: float, threshold_percent: float):
        self.threshold_mb = threshold_mb
        self.threshold_percent = threshold_percent

    def detect(self, records: list[ProcessRecord]) -> list[MemoryDetection]:
        findings = []

        for record in records:
            if (
                record.rss_mb > self.threshold_mb
                or record.memory_percent > self.threshold_percent
            ):
                findings.append(
                    MemoryDetection(
                        rule="high_memory",
                        severity="WARNING",
                        pid=record.pid,
                        user=record.username,
                        evidence=(
                            f"RSS {record.rss_mb:.1f} MB, "
                            f"memory {record.memory_percent:.1f}%"
                        ),
                        process=record,
                    )
                )

        return findings
```

### `procwatch/detectors/duration_detector.py`

```python
from dataclasses import dataclass

from ..collector import ProcessRecord


@dataclass
class DurationDetection:
    rule: str
    severity: str
    pid: int
    user: str
    evidence: str
    process: ProcessRecord


class DurationDetector:
    def __init__(self, long_running_hours: float):
        self.long_running_hours = long_running_hours

    def detect(self, records: list[ProcessRecord]) -> list[DurationDetection]:
        threshold_seconds = self.long_running_hours * 3600
        findings = []

        for record in records:
            if record.elapsed_seconds > threshold_seconds:
                hours = record.elapsed_seconds / 3600
                findings.append(
                    DurationDetection(
                        rule="long_running",
                        severity="INFO",
                        pid=record.pid,
                        user=record.username,
                        evidence=f"Running for {hours:.1f} hours",
                        process=record,
                    )
                )

        return findings
```

### `procwatch/detectors/state_detector.py`

```python
from dataclasses import dataclass

from ..collector import ProcessRecord


@dataclass
class StateDetection:
    rule: str
    severity: str
    pid: int
    user: str
    evidence: str
    process: ProcessRecord


class StateDetector:
    def detect(self, records: list[ProcessRecord]) -> list[StateDetection]:
        findings = []

        for record in records:
            if record.state == psutil.STATUS_STOPPED:
                findings.append(
                    StateDetection(
                        rule="stopped_process",
                        severity="WARNING",
                        pid=record.pid,
                        user=record.username,
                        evidence=f"Process state is {record.state}",
                        process=record,
                    )
                )

            if record.state == psutil.STATUS_ZOMBIE:
                findings.append(
                    StateDetection(
                        rule="zombie_process",
                        severity="CRITICAL",
                        pid=record.pid,
                        user=record.username,
                        evidence=(
                            f"Zombie process; parent PID {record.ppid}; "
                            f"parent command {record.parent_name}"
                        ),
                        process=record,
                    )
                )

        return findings
```

### `procwatch/detectors/suspicious_detector.py`

```python
from dataclasses import dataclass
from pathlib import Path

from ..collector import ProcessRecord


@dataclass
class SuspiciousDetection:
    rule: str
    severity: str
    pid: int
    user: str
    evidence: str
    process: ProcessRecord


class SuspiciousDetector:
    def __init__(
        self,
        suspicious_paths: tuple[str, ...],
        root_allowlist: tuple[str, ...],
        custom_patterns: tuple[str, ...],
    ):
        self.suspicious_paths = tuple(Path(path) for path in suspicious_paths)
        self.root_allowlist = root_allowlist
        self.custom_patterns = custom_patterns

    def detect(self, records: list[ProcessRecord]) -> list[SuspiciousDetection]:
        findings = []

        for record in records:
            executable = Path(record.executable) if record.executable else None

            if executable:
                for suspicious_path in self.suspicious_paths:
                    try:
                        executable.relative_to(suspicious_path)
                        findings.append(
                            SuspiciousDetection(
                                rule="suspicious_path",
                                severity="CRITICAL",
                                pid=record.pid,
                                user=record.username,
                                evidence=f"Executable under {suspicious_path}: {executable}",
                                process=record,
                            )
                        )
                        break
                    except ValueError:
                        pass

            if not record.cmdline:
                findings.append(
                    SuspiciousDetection(
                        rule="empty_command_line",
                        severity="WARNING",
                        pid=record.pid,
                        user=record.username,
                        evidence="Process has no visible command line",
                        process=record,
                    )
                )

            if record.username == "root" and record.name not in self.root_allowlist:
                findings.append(
                    SuspiciousDetection(
                        rule="unexpected_root_process",
                        severity="WARNING",
                        pid=record.pid,
                        user=record.username,
                        evidence=f"Root process not in allowlist: {record.name}",
                        process=record,
                    )
                )

            for pattern in self.custom_patterns:
                if pattern.lower() in record.cmdline.lower():
                    findings.append(
                        SuspiciousDetection(
                            rule="custom_pattern",
                            severity="CRITICAL",
                            pid=record.pid,
                            user=record.username,
                            evidence=f"Command line matches pattern: {pattern}",
                            process=record,
                        )
                    )

        return findings
```

### `procwatch/alerts/console_alert.py`

```python
import logging


class ConsoleAlert:
    def send(self, detection) -> None:
        logger = logging.getLogger("procwatch")
        logger.warning(
            "[%s] %s PID=%s USER=%s RULE=%s EVIDENCE=%s",
            detection.severity,
            "ALERT",
            detection.pid,
            detection.user,
            detection.rule,
            detection.evidence,
        )
```

### `procwatch/alerts/file_alert.py`

```python
import logging
from pathlib import Path


class FileAlert:
    def __init__(self, path: Path):
        self.path = path
        self.path.parent.mkdir(parents=True, exist_ok=True)
        self.logger = logging.getLogger("procwatch.file")
        handler = logging.FileHandler(self.path)
        handler.setFormatter(
            logging.Formatter(
                "%(asctime)s %(levelname)s %(message)s"
            )
        )
        self.logger.addHandler(handler)
        self.logger.setLevel(logging.INFO)

    def send(self, detection) -> None:
        self.logger.info(
            "timestamp=%s pid=%s user=%s rule=%s severity=%s evidence=%s",
            detection.process.create_time.isoformat(),
            detection.pid,
            detection.user,
            detection.rule,
            detection.severity,
            detection.evidence,
        )
```

### `procwatch/alerts/webhook_alert.py`

```python
import json
import logging
import urllib.request


class WebhookAlert:
    def __init__(self, url: str):
        self.url = url
        self.logger = logging.getLogger("procwatch.webhook")

    def send(self, detection) -> None:
        payload = {
            "severity": detection.severity,
            "rule": detection.rule,
            "pid": detection.pid,
            "user": detection.user,
            "evidence": detection.evidence,
            "command": detection.process.cmdline,
        }

        request = urllib.request.Request(
            self.url,
            data=json.dumps(payload).encode("utf-8"),
            headers={"Content-Type": "application/json"},
            method="POST",
        )

        try:
            with urllib.request.urlopen(request, timeout=5) as response:
                self.logger.info("Webhook status: %s", response.status)
        except Exception as exc:
            self.logger.warning("Webhook delivery failed: %s", exc)
```

### `procwatch/reports/terminal_report.py`

```python
from ..collector import ProcessRecord


class TerminalReport:
    def render(self, records: list[ProcessRecord]) -> str:
        lines = [
            "ProcWatch Report",
            "================",
            "",
            "PID    CPU%   MEM%   USER          STATE  ELAPSED  COMMAND",
        ]

        for record in records:
            hours, remainder = divmod(int(record.elapsed_seconds), 3600)
            minutes, seconds = divmod(remainder, 60)
            elapsed = f"{hours:02d}:{minutes:02d}:{seconds:02d}"

            lines.append(
                f"{record.pid:<6} {record.cpu_percent:<6.1f} "
                f"{record.memory_percent:<6.1f} {record.username:<13} "
                f"{record.state:<6} {elapsed:<8} {record.cmdline[:60]}"
            )

        return "\n".join(lines)
```

### `procwatch/reports/json_report.py`

```python
import json
from dataclasses import asdict
from pathlib import Path

from ..collector import ProcessRecord


class JsonReport:
    def write(self, records: list[ProcessRecord], path: Path) -> Path:
        path.parent.mkdir(parents=True, exist_ok=True)
        payload = [asdict(record) for record in records]

        with path.open("w", encoding="utf-8") as handle:
            json.dump(payload, handle, indent=2, default=str)

        return path
```

### `procwatch/reports/csv_report.py`

```python
import csv
from dataclasses import asdict
from pathlib import Path

from ..collector import ProcessRecord


class CsvReport:
    def write(self, records: list[ProcessRecord], path: Path) -> Path:
        path.parent.mkdir(parents=True, exist_ok=True)
        fieldnames = list(asdict(records[0]).keys()) if records else []

        with path.open("w", newline="", encoding="utf-8") as handle:
            writer = csv.DictWriter(handle, fieldnames=fieldnames)
            writer.writeheader()

            for record in records:
                writer.writerow(asdict(record))

        return path
```

### `procwatch/reports/markdown_report.py`

```python
from pathlib import Path

from ..collector import ProcessRecord


class MarkdownReport:
    def write(self, records: list[ProcessRecord], path: Path) -> Path:
        path.parent.mkdir(parents=True, exist_ok=True)
        lines = [
            "# ProcWatch Report",
            "",
            "| PID | User | State | CPU % | RSS MB | Elapsed seconds | Command |",
            "|---:|---|---|---:|---:|---:|---|",
        ]

        for record in records:
            lines.append(
                f"| {record.pid} | {record.username} | {record.state} | "
                f"{record.cpu_percent:.1f} | {record.rss_mb:.1f} | "
                f"{record.elapsed_seconds:.0f} | `{record.cmdline}` |"
            )

        path.write_text("\n".join(lines) + "\n", encoding="utf-8")
        return path
```

### `procwatch/cli.py`

```python
import argparse
import logging
import sys
from pathlib import Path

from .collector import ProcessCollector
from .config import ProcWatchConfig
from .detectors.cpu_detector import CpuDetector
from .detectors.duration_detector import DurationDetector
from .detectors.memory_detector import MemoryDetector
from .detectors.state_detector import StateDetector
from .detectors.suspicious_detector import SuspiciousDetector
from .alerts.console_alert import ConsoleAlert
from .alerts.file_alert import FileAlert
from .reports.csv_report import CsvReport
from .reports.json_report import JsonReport
from .reports.markdown_report import MarkdownReport
from .reports.terminal_report import TerminalReport


def build_parser() -> argparse.ArgumentParser:
    parser = argparse.ArgumentParser(
        prog="procwatch",
        description="Monitor Linux processes and detect resource or security issues.",
    )

    subparsers = parser.add_subparsers(dest="command", required=True)

    scan = subparsers.add_parser("scan", help="Perform a one-time scan")
    scan.add_argument("--json", action="store_true", help="Write JSON report")
    scan.add_argument("--csv", action="store_true", help="Write CSV report")
    scan.add_argument("--markdown", action="store_true", help="Write Markdown report")
    scan.add_argument("--output", default="reports", help="Report directory")

    watch = subparsers.add_parser("watch", help="Monitor continuously")
    watch.add_argument("--interval", type=int, default=30)
    watch.add_argument("--cpu-threshold", type=float, default=80.0)
    watch.add_argument("--memory-threshold", type=float, default=500.0)
    watch.add_argument("--long-running-hours", type=float, default=24.0)
    watch.add_argument(
        "--suspicious-paths",
        default="/tmp,/var/tmp,/dev/shm",
    )
    watch.add_argument("--exclude", default="")

    return parser


def parse_exclusions(value: str) -> tuple[str, ...]:
    return tuple(item.strip() for item in value.split(",") if item.strip())


def run_scan(args) -> int:
    config = ProcWatchConfig(report_directory=Path(args.output))
    collector = ProcessCollector(config)
    records = collector.collect()

    print(TerminalReport().render(records))

    if args.json:
        path = JsonReport().write(
            records, config.report_directory / "procwatch-report.json"
        )
        print(f"JSON report: {path}")

    if args.csv:
        path = CsvReport().write(
            records, config.report_directory / "procwatch-report.csv"
        )
        print(f"CSV report: {path}")

    if args.markdown:
        path = MarkdownReport().write(
            records, config.report_directory / "procwatch-report.md"
        )
        print(f"Markdown report: {path}")

    return 0


def run_watch(args) -> int:
    import time

    config = ProcWatchConfig(
        cpu_threshold=args.cpu_threshold,
        memory_threshold_mb=args.memory_threshold,
        long_running_hours=args.long_running_hours,
        interval_seconds=args.interval,
        suspicious_paths=tuple(args.suspicious_paths.split(",")),
        excluded_names=parse_exclusions(args.exclude),
    )

    collector = ProcessCollector(config)
    console = ConsoleAlert()
    file_alert = FileAlert(config.alert_log)

    logging.basicConfig(
        level=logging.INFO,
        format="%(asctime)s %(levelname)s %(message)s",
    )

    cpu = CpuDetector(config.cpu_threshold)
    memory = MemoryDetector(
        config.memory_threshold_mb, config.memory_percent_threshold
    )
    duration = DurationDetector(config.long_running_hours)
    state = StateDetector()
    suspicious = SuspiciousDetector(
        config.suspicious_paths,
        config.root_allowlist,
        config.custom_suspicious_patterns,
    )

    while True:
        records = collector.collect()
        findings = (
            cpu.detect(records)
            + memory.detect(records)
            + duration.detect(records)
            + state.detect(records)
            + suspicious.detect(records)
        )

        for finding in findings:
            console.send(finding)
            file_alert.send(finding)

        time.sleep(config.interval_seconds)


def main() -> int:
    parser = build_parser()
    args = parser.parse_args()

    if args.command == "scan":
        return run_scan(args)

    if args.command == "watch":
        return run_watch(args)

    parser.print_help()
    return 1


if __name__ == "__main__":
    sys.exit(main())
```

### `pyproject.toml`

```toml
[build-system]
requires = ["setuptools>=68"]
build-backend = "setuptools.build_meta"

[project]
name = "procwatch"
version = "1.0.0"
description = "Linux process monitoring and alerting tool"
requires-python = ">=3.10"
dependencies = [
    "psutil>=5.9.0",
]

[project.scripts]
procwatch = "procwatch.cli:main"

[tool.setuptools.packages.find]
include = ["procwatch*"]
```

### Install and run

```bash
cd ~/process-lab/procwatch
python -m pip install -e .
procwatch scan
procwatch scan --json --csv --markdown --output reports/
procwatch watch --interval 30
procwatch watch --cpu-threshold 75 --memory-threshold 400
procwatch watch --long-running-hours 12
procwatch watch --suspicious-paths /tmp,/var/tmp,/dev/shm
procwatch watch --exclude systemd,sshd,dbus
```

## Example output

```text
ProcWatch Report
================

PID    CPU%   MEM%   USER          STATE  ELAPSED  COMMAND
4821   96.4   1.8    alice         R      00:12:33 python3 cpu_load.py
2210   0.2    21.4   webapp        S      03:12:44 /usr/local/bin/wavecorp-web
5312   0.0    0.1    dev1          T      01:22:10 python3 signal_demo.py
6102   0.0    0.0    worker        Z      00:00:04 python3 worker.py
7231   2.1    0.4    alice         S      00:01:11 /tmp/.hidden/update
```

## Implementation requirements

- Python 3.10 or newer.
- `psutil` for process and system metrics.
- `argparse` or `click` for CLI.
- Dataclasses for process records.
- `logging` for alerts.
- `json`, `csv`, and `pathlib` for reports.
- `pytest` for tests.
- Optional Flask or Streamlit dashboard.

Design rules:

- Do not kill processes automatically by default.
- Provide a separate optional remediation mode with explicit confirmation.
- Log timestamp, PID, user, rule, and evidence for every detection.
- Make thresholds configurable.
- Allow exclusions to reduce false positives.
- Handle processes that exit during scanning.
- Handle permission errors safely.
- Use mocked `psutil` data in unit tests.

***

# Cheat Sheets

## Process-management command cheat sheet

```bash
ps
ps aux
ps -ef
ps -ef --forest
ps aux --sort=-%cpu | head -n 15
ps aux --sort=-%mem | head -n 15
ps -p PID -o pid,ppid,user,stat,%cpu,%mem,etime,cmd
ps -u USER -o pid,ppid,stat,%cpu,%mem,cmd
ps -C NAME -o pid,ppid,user,stat,cmd

top
top -d 2
top -p PID
top -u USER
top -b -n 1

htop
htop -u USER
htop -p PID

pstree
pstree -p
pstree -ap PID

pgrep NAME
pgrep -l NAME
pgrep -af PATTERN
pgrep -u USER NAME
pgrep -P PPID
pgrep -x NAME

kill PID
kill -TERM PID
kill -KILL PID
kill -STOP PID
kill -CONT PID
pkill -TERM -f PATTERN
killall -TERM NAME

jobs
jobs -l
fg %JOB
bg %JOB
nohup COMMAND > output.log 2>&1 &
disown -h %JOB

nice -n 10 COMMAND
renice -n 19 -p PID
sudo renice -n -5 -p PID
```

## Signal cheat sheet

| Signal | Number | Purpose | Can be caught? |
|---|---:|---|---|
| `SIGHUP` | 1 | Hangup/config reload | Usually yes |
| `SIGINT` | 2 | Interrupt, usually `Ctrl+C` | Usually yes |
| `SIGQUIT` | 3 | Quit and dump core | Usually yes |
| `SIGKILL` | 9 | Force termination | No |
| `SIGTERM` | 15 | Graceful termination | Usually yes |
| `SIGCONT` | 18 | Resume stopped process | No |
| `SIGCHLD` | 17 | Child status changed | Usually yes |
| `SIGSTOP` | 19 | Pause process | No |

## Process state cheat sheet

| State | Meaning |
|---|---|
| `R` | Running or runnable |
| `S` | Interruptible sleep |
| `D` | Uninterruptible sleep |
| `T` | Stopped |
| `Z` | Zombie |
| `I` | Idle kernel thread |

## Process priority cheat sheet

| Nice value | Meaning |
|---:|---|
| `-20` | Highest priority; normally root only |
| `0` | Default |
| `10` | Lower priority |
| `19` | Lowest priority |

## ProcWatch CLI cheat sheet

```bash
procwatch scan
procwatch scan --json
procwatch scan --csv
procwatch scan --markdown
procwatch scan --output reports/

procwatch watch --interval 30
procwatch watch --cpu-threshold 80 --memory-threshold 500
procwatch watch --long-running-hours 24
procwatch watch --suspicious-paths /tmp,/var/tmp,/dev/shm
procwatch watch --exclude systemd,sshd,nginx
```

***

# Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| High CPU usage | Legitimate workload, runaway loop, or compromised process | Identify PID, inspect command/parent, check logs, renice if safe, terminate only when authorized |
| High memory usage | Memory leak, cache growth, oversized workload, or too many workers | Check RSS, swap, OOM logs, application metrics, and restart only when approved |
| Process will not terminate | It ignores `SIGTERM`, is stuck in `D` state, or has child processes | Check state, inspect children, wait, then use `SIGKILL` only if authorized |
| Process is in `D` state | Waiting on kernel or hardware I/O | Check storage, NFS, device, kernel logs, and hardware; `SIGKILL` may not work |
| Zombies keep appearing | Parent is not reaping child exit status | Investigate and restart/fix the parent application |
| Background job stops after terminal closes | Terminal closure sends `SIGHUP` | Use `nohup`, redirect output, and use `disown -h` |
| `renice` permission denied | User tried to lower nice value | Use `sudo` or ask an authorized administrator |
| `pkill` matches too many processes | Broad pattern or missing `-x` | Always run `pgrep -af` first; use exact PID where possible |
| Monitoring tool reports false positives | Threshold too low, expected process not excluded, or CPU sampling interval too short | Tune thresholds, add exclusions, sample over time, and validate detections manually |
| Service disappears unexpectedly | OOM killer, crash, configuration failure, or supervisor restart | Check `systemctl status`, journal logs, kernel OOM logs, and application logs |

***

# Practice Exercises

1. Start `sleep 600`, find its PID, and inspect `/proc/PID/status`.
2. Start three background jobs and use `jobs -l`, `fg`, and `bg`.
3. Use `nohup` to start a long-running script and verify it survives terminal closure.
4. Find the top five CPU-consuming processes.
5. Find the top five memory-consuming processes.
6. Pause and resume a process using `SIGSTOP` and `SIGCONT`.
7. Use `pgrep -af` before terminating a controlled workload.
8. Create a CPU workload, renice it, and terminate it safely.
9. Find all zombie processes and identify their parents.
10. Generate JSON, CSV, and Markdown reports using ProcWatch.

***

# Interview Questions

## 1. What is the difference between a program and a process?

A program is a stored executable file. A process is a running instance of that program with a PID, memory, state, and execution context.

## 2. Why is `SIGTERM` preferred over `SIGKILL`?

`SIGTERM` allows the application to shut down cleanly, flush data, close connections, and release resources. `SIGKILL` cannot be caught and may leave incomplete work or stale state.

## 3. What is a zombie process?

A zombie is a terminated process whose parent has not collected its exit status. It consumes a PID entry but not CPU or normal memory. Investigate the parent.

## 4. What does a high load average mean?

It indicates work waiting for CPU or potentially blocked on uninterruptible I/O. Interpret it relative to CPU count and current process states.

## 5. Why should `pkill` be used carefully?

It can match multiple processes. Always inspect matches with `pgrep -af` before sending a signal, and prefer an exact PID for critical services.

***

# Final Capstone Project

## Objective

Deploy ProcWatch on a Linux VM, simulate resource issues, create controlled stopped and zombie processes, configure thresholds, generate reports, investigate findings, and document an incident-response workflow.

## Steps

### 1. Deploy ProcWatch

```bash
cd ~/process-lab/procwatch
python -m pip install -e .
procwatch scan
```

### 2. Simulate high CPU

```bash
cd ~/process-lab
python3 cpu_load.py > cpu-load.log 2>&1 &
```

Monitor:

```bash
procwatch scan
top -b -n 1 | head -n 20
pgrep -af cpu_load.py
```

### 3. Simulate high memory

Create a controlled memory workload:

```bash
cat > ~/process-lab/memory_load.py <<'EOF'
import time

data = []

for _ in range(50):
    data.append(bytearray(20 * 1024 * 1024))
    time.sleep(1)

time.sleep(60)
EOF

python3 ~/process-lab/memory_load.py > memory-load.log 2>&1 &
```

Monitor:

```bash
free -h
procwatch scan
ps aux --sort=-%mem | head -n 10
```

### 4. Create a stopped process

```bash
python3 ~/process-lab/signal_demo.py > stopped-demo.log 2>&1 &
pid=$(pgrep -f 'signal_demo.py' | head -n 1)
kill -STOP "$pid"
ps -p "$pid" -o pid,user,stat,etime,cmd
```

### 5. Create a zombie safely

Create a parent that intentionally leaves a child unreaped for a short time:

```bash
cat > ~/process-lab/zombie_demo.py <<'EOF'
import os
import time

child = os.fork()

if child == 0:
    os._exit(0)

print(f"Child PID: {child}")
time.sleep(60)
EOF

python3 ~/process-lab/zombie_demo.py > zombie-demo.log 2>&1 &
```

Find it:

```bash
ps -eo pid,ppid,stat,cmd | awk '$3 ~ /^Z/ {print}'
```

After the demonstration, terminate the parent so the zombie is reaped:

```bash
pkill -TERM -f 'python3 zombie_demo.py'
```

### 6. Configure thresholds and exclusions

```bash
procwatch watch \
  --interval 10 \
  --cpu-threshold 70 \
  --memory-threshold 300 \
  --long-running-hours 1 \
  --exclude systemd,sshd,dbus
```

### 7. Generate reports

```bash
procwatch scan --json --csv --markdown --output reports/
ls -lh reports/
```

### 8. Investigate and resolve

```bash
pgrep -af 'cpu_load.py|memory_load.py|signal_demo.py|zombie_demo.py'
ps aux --sort=-%cpu | head -n 10
ps aux --sort=-%mem | head -n 10
free -h
```

Terminate the controlled workloads:

```bash
pkill -TERM -f 'python3 cpu_load.py'
pkill -TERM -f 'python3 memory_load.py'
pkill -TERM -f 'python3 signal_demo.py'
pkill -TERM -f 'python3 zombie_demo.py'
```

Verify:

```bash
pgrep -af 'cpu_load.py|memory_load.py|signal_demo.py|zombie_demo.py' || echo "All lab workloads stopped"
free -h
```

### 9. Incident-response workflow

Document:

1. Detection time and alert source.
2. Affected PID, PPID, user, command, executable, and state.
3. CPU, memory, elapsed time, and parent-child relationship.
4. Whether the process was expected.
5. Evidence collected from `/proc`, logs, and process tree.
6. Actions taken: renice, stop, terminate, restart, or escalate.
7. Verification that CPU, memory, and service health recovered.
8. Follow-up actions: threshold tuning, exclusions, monitoring improvements, and root-cause analysis.

***

# Cleanup

> Review before running. This removes only the process-lab workloads and generated reports from this tutorial.

```bash
pkill -TERM -f 'python3 cpu_load.py' 2>/dev/null || true
pkill -TERM -f 'python3 memory_load.py' 2>/dev/null || true
pkill -TERM -f 'python3 signal_demo.py' 2>/dev/null || true
pkill -TERM -f 'python3 zombie_demo.py' 2>/dev/null || true
pkill -TERM -f '^sleep 600$' 2>/dev/null || true

sleep 2

pgrep -af 'cpu_load.py|memory_load.py|signal_demo.py|zombie_demo.py|^sleep 600$' || echo "No lab processes remain"

rm -rf ~/process-lab/procwatch-venv
rm -f ~/process-lab/{cpu_load.py,memory_load.py,signal_demo.py,zombie_demo.py}
rm -f ~/process-lab/{cpu-load.log,memory-load.log,signal-demo.log,zombie-demo.log,stopped-demo.log}
rm -rf ~/process-lab/procwatch/reports
```

# Next Topics

- Linux systemd services and journal logging.
- Process limits, cgroups, control groups, and resource isolation.
- Containers and namespaces.
- Performance monitoring with `sar`, `iostat`, `pidstat`, and Prometheus.
- Linux security auditing and incident response.
