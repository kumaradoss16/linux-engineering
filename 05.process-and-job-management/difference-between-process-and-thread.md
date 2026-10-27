# Difference Between Process and Thread in Linux

A process is an executing program with its own virtual address space and operating-system-managed resources. A thread is an execution path within a process; threads in the same process typically share memory and many resources while maintaining separate execution state.

In Linux system administration, understanding this distinction helps with performance troubleshooting, CPU utilization, memory usage, and application debugging.

## 1. Process vs. thread at a glance

| Feature              | Process                                                  | Thread                                                                |
| -------------------- | -------------------------------------------------------- | --------------------------------------------------------------------- |
| Definition           | An instance of an executing program                      | An execution unit within a process                                    |
| Memory               | Has its own virtual address space                        | Shares the process's address space with sibling threads               |
| Code and data        | Has its own process address space                        | Can access shared code, global data, and heap                         |
| Stack                | Has its own stack                                        | Each thread has its own stack                                         |
| Registers            | Has its own execution state                              | Has its own registers and instruction pointer                         |
| Resources            | Has process-level resources, such as file descriptors    | Typically shares file descriptors and other process resources         |
| Communication        | Often uses pipes, sockets, shared memory, or other IPC   | Can communicate through shared memory, with synchronization           |
| Creation overhead    | Generally higher                                         | Generally lower                                                       |
| Failure impact       | A process crash is usually isolated from other processes | An unhandled fatal error in a thread can terminate the entire process |
| Linux identification | Process ID (PID)                                         | Thread ID (TID); the thread-group ID (TGID) identifies its process    |

## 2. Visual representation

<img width="539" height="371" alt="image" src="https://github.com/user-attachments/assets/bfe5db64-daa2-4826-a343-81c6f4ec5f49" />

## 3. Practical Linux examples

### A. View processes

Use `ps` to list processes:

```
ps -eo pid,ppid,comm
```

Example output:

```
    PID    PPID COMMAND
      1       0 systemd
   1200       1 sshd
   1234    1200 bash
   4567    1234 python3
```

- `PID`: Process ID.
- `PPID`: Parent Process ID.
- `COMMAND`: Command name.

Each process has its own PID.

### B. View threads

Use `ps -T` to display threads for a particular process:

```
ps -T -p 4567
```

Or use:

```
ps -eLf
```

The `-L` option displays threads, while `-f` requests full-format output.

Example:

```
UID      PID   PPID   LWP  C NLWP STIME TTY  TIME CMD
student 4567   1234  4567  0    3 10:00 ?    0:00 python3
student 4567   1234  4568  0    3 10:00 ?    0:00 python3
student 4567   1234  4569  0    3 10:00 ?    0:00 python3
```

Here:

- `PID` is the process or thread-group ID.
- `LWP` is the individual Linux thread ID.
- `NLWP` is the number of threads in the process.

All three rows belong to the same process, but each represents a different thread.

### C. View threads interactively

```
top -H -p 4567
```

This displays individual threads for process `4567`, helping you identify which thread is consuming CPU.

With `htop`, you can also enable thread display through its available display options.

## 4. How Linux creates processes and threads

Linux uses the `fork()` system call to create a new process, commonly followed by `exec()` to run another program.

Threads are commonly created using the `pthread_create()` library function, which uses Linux kernel facilities to create a new thread.

| Operation                   | Common mechanism                         |
| --------------------------- | ---------------------------------------- |
| Create a process            | `fork()`                                 |
| Replace a process's program | `exec()`                                 |
| Create a thread             | `pthread_create()`                       |
| Wait for a child process    | `wait()` / `waitpid()`                   |
| Synchronize threads         | Mutexes, condition variables, semaphores |

A technical nuance: Linux represents both processes and threads as kernel tasks. Threads created with shared resources are distinguished by which resources they share, rather than by an entirely separate kernel object type.

## 5. When should you use processes or threads?

- Processes: Prefer them when you need stronger memory isolation, independent application components, or separate failure boundaries.
- Threads: Prefer them when tasks need to share data efficiently within one application, such as handling multiple client connections.

Neither is universally faster. The right choice depends on workload, synchronization costs, isolation requirements, and the programming model.

Remember: A process provides an execution environment and resource boundary; a thread provides an individual execution path within that environment.
