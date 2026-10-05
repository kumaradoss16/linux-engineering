# Topic 02 — Linux Architecture

## 1. Introduction

Topic 01 defined Linux as a kernel and a distribution as the complete system built around it. This topic opens that system up and shows how its parts are arranged and how they relate to each other.

Architecture matters because every operational problem happens at a particular layer. A slow website, a full disk, a missing network interface and a crashed service are different problems because they live in different parts of the system. If you know the layers, you know where to look.

This topic is conceptual. It does not use commands. Topic 03 follows a request through the architecture, so here the focus is on what each layer is and how the layers connect.

## 2. Learning Objectives

After this topic you should be able to:

- Describe the layers of a Linux system from hardware to users
- Explain the role of CPU, RAM, storage and network devices
- Explain what device drivers do and where they sit
- Describe the main kernel subsystems
- Explain user space and kernel space and why they are separated
- Explain what a system call is
- Explain the role of system libraries
- Distinguish system services, daemons, the shell and applications
- Explain why applications normally do not control hardware directly

## 3. Core Concepts

### 3.1 Hardware

Hardware is the physical (or virtualized) machine. It does the actual work but makes no decisions about who may use it.

| Component | Role |
|---|---|
| CPU | Executes instructions. Only one program's instructions run on a given core at a time. |
| RAM | Holds running programs and their data. Fast, but contents are lost at power-off. |
| Storage | Holds data persistently: hard disks, SSDs, network-attached volumes. |
| Network devices | Network adapters that send and receive data on a network. |
| Other devices | GPU, USB devices, serial ports and similar peripherals. |

In a virtual machine, the hypervisor presents virtual versions of these components. The Linux kernel treats them much like physical ones.

### 3.2 Device drivers

Every device type and every vendor's implementation speaks its own low-level language. A network adapter from one vendor is controlled differently from another vendor's, and the same is true of disk controllers.

A device driver is the code that knows how to control one kind of device. It sits between the kernel's general-purpose subsystems and the specific hardware.

```text
Kernel networking subsystem
          |
          v
   Network device driver
          |
          v
   Specific network adapter
```

Drivers allow the kernel to offer one consistent interface to applications regardless of which hardware is installed. Drivers run in kernel space and are part of the kernel's reach into hardware. On Linux many drivers can be loaded as kernel modules; Topic 05 covers modules.

### 3.3 The Linux kernel and its subsystems

The kernel is the privileged core. It is divided into subsystems, each responsible for one class of resource.

| Subsystem | Responsibility |
|---|---|
| Process management | Creates processes, schedules CPU time, ends processes |
| Memory management | Allocates RAM, isolates processes from each other, manages virtual memory |
| Filesystems | Turns raw storage into files and directories, and provides a common file interface |
| Networking | Implements network protocols and moves data between applications and network devices |
| Device drivers | Control hardware on behalf of the other subsystems |
| Security | Enforces ownership, permissions and other access-control rules |

The subsystems cooperate. Reading a file, for example, involves the filesystem subsystem, memory management for buffering, the storage driver for the actual disk access, and the security subsystem for the permission check.

Topic 05 covers the kernel's internal responsibilities in more depth. Here the point is that the kernel is one component with several internal parts, not a single undivided block.

### 3.4 System calls

A system call is the defined interface through which a program in user space asks the kernel to do something that requires privilege. Examples of requests: open a file, read data, allocate memory, create a new process, send data over the network.

System calls are the only controlled entry point into the kernel. A program cannot jump into arbitrary kernel code. It must use one of the defined requests, and the kernel decides whether to carry it out.

```text
Application
    |
    v
System call
    |
    v
Linux kernel
    |
    v
Hardware
```

### 3.5 System libraries

Programs rarely make system calls by hand. They call functions in system libraries, and the libraries make the system calls.

The most important example is the C library. It provides ready-made functions for common tasks such as opening files, allocating memory and working with network connections. Behind these functions are the corresponding system calls.

Libraries serve two purposes:

1. They hide low-level detail, so programmers do not repeat the same code in every program.
2. They are shared, so many programs use one copy of the same code.

This sharing has an operational consequence: updating a widely used library affects every program that uses it.

### 3.6 Shell and user interfaces

The shell is a program that lets a person interact with the system by typing instructions. It reads what the user enters, interprets it, and asks the system to run the requested programs.

The important architectural fact is that the shell is not part of the kernel. It is an ordinary user-space program, which makes it replaceable. Graphical desktops are another kind of user interface and are also user-space software. Server systems often have no graphical interface at all and are managed through a shell, usually from a remote connection. Topic 12 covers the shell in detail.

### 3.7 System services and daemons

Many programs on a Linux system are not started by a person at all. They run in the background, usually from boot onward, to provide a function: accept remote logins, answer web requests, resolve names, record logs.

- A daemon is a background process that runs without an interactive terminal, usually for a long time.
- A system service is the managed function a daemon provides, started, stopped and supervised by the system's service manager.

Topic 11 covers the full lifecycle. For this topic, treat them as user-space programs that provide ongoing functions to the system and its users.

### 3.8 Applications

Applications are the programs that deliver the actual value of the machine: a web server, a database, a Python application, a browser, monitoring software. They run in user space. They use libraries and system calls to obtain kernel services.

Some programs sit in between. A web server is an application, but it usually runs as a daemon and is managed as a service. The categories describe roles, not separate kinds of software.

### 3.9 Users

At the top of the architecture are the users. A user may be a person, but may also be a service account, an account that exists so that a service can run with limited privileges. The kernel identifies every process by the user it runs as and uses that identity to decide what the process may access. Topics 08 and 09 cover users, groups and permissions.

## 4. Architecture / Internal Flow

The layered architecture of a Linux system:

```text
+-----------------------------+
|           Users             |
+-----------------------------+
              |
+-----------------------------+
|       Applications          |
+-----------------------------+
              |
+-----------------------------+
| Shell / User Interfaces     |
+-----------------------------+
              |
+-----------------------------+
| System Libraries / APIs     |
+-----------------------------+
              |
+-----------------------------+
|       System Calls          |
+-----------------------------+
              |
+-----------------------------+
|        Linux Kernel         |
|                             |
| Process Management          |
| Memory Management           |
| File Systems                |
| Networking                  |
| Device Drivers              |
| Security                    |
+-----------------------------+
              |
+-----------------------------+
|          Hardware           |
+-----------------------------+
```

System services and daemons run in the same region as applications, in user space. The diagram shows them under the application layer because they use the same libraries and system calls to reach the kernel.

### User space and kernel space

The architecture divides into two regions:

```text
+-------------------------------------------------+
|                  USER SPACE                     |
|  Applications, shell, services, daemons,        |
|  system libraries                               |
+-------------------------------------------------+
|              System call interface              |
+-------------------------------------------------+
|                 KERNEL SPACE                    |
|  Kernel subsystems and device drivers           |
+-------------------------------------------------+
|                   HARDWARE                      |
+-------------------------------------------------+
```

| | User space | Kernel space |
|---|---|---|
| Runs | Applications, shells, services, libraries | The kernel and its drivers |
| Privilege | Restricted | Full access to hardware and all memory |
| If it fails | Usually one process is affected | Can affect the entire system |
| Hardware access | Only through the kernel | Direct |

### Why the separation exists

Without a boundary, any program could read or overwrite another program's memory, control any device, or ignore access rules. A single faulty program could crash the whole machine.

The separation provides:

- Stability: a failing application is a failed process, not a failed system.
- Security: a compromised application is still limited by what the kernel allows.
- Isolation: processes cannot read or alter each other's memory.
- Fairness: the kernel controls how CPU and memory are shared.

The boundary is enforced by the CPU's privilege levels. Kernel code runs in a privileged mode and user programs run in a restricted mode. An attempt by a user program to perform a privileged operation is blocked.

## 5. How It Works

When an application needs something only the kernel can do, the architecture works in this order:

1. The application calls a function in a system library.
2. The library prepares the request and makes the corresponding system call.
3. The CPU switches from restricted mode to privileged mode, and control passes to the kernel.
4. The kernel checks the request, including whether the calling process is permitted to do it.
5. The relevant subsystem handles the request, using a device driver if hardware is involved.
6. The kernel returns the result and the CPU returns to restricted mode.
7. The library passes the result back to the application.

This is the reason applications normally do not control hardware directly. The kernel is the one component trusted to arbitrate access, and the system call interface is the controlled door.

Role of each piece in this sequence:

| Piece | Role in the sequence |
|---|---|
| System library | Converts a convenient function call into a system call |
| System call | Controlled crossing from user space to kernel space |
| Kernel subsystem | Decides and performs the operation |
| Device driver | Translates the kernel's generic request into device-specific control |

Topic 03 walks through specific examples, such as opening a file or sending network data, in detail.

## 6. Real-World Example

A Linux server hosts a company's internal web application. A user in the office opens the application in a browser.

Where each architectural piece appears:

```text
Users (staff, and the service account the web server runs as)
          |
Application (the web application and its web server)
          |
Daemon / service (web server running in the background)
          |
System libraries (network and file functions)
          |
System calls
          |
Kernel: networking, filesystem, memory, process management, security
          |
Device drivers: network adapter driver, storage driver
          |
Hardware: network adapter, CPU, RAM, disk
```

An administrator connected through a remote shell is a separate set of user-space programs on the same machine, going through the same kernel.

Two failures at different layers:

1. The web application has a memory bug and its process is terminated. Users see errors, but the kernel, the shell sessions and other services are unaffected. This is a user-space failure and is contained.
2. A faulty network driver causes network problems or system instability. Because drivers run in kernel space, a fault here can affect the whole machine. Troubleshooting looks at the kernel and hardware layers instead of the application.

The architecture tells you which of these you are dealing with.

## 7. Practical Mental Model

Think of the system as a secure facility:

- Users and applications are visitors and tenants.
- System libraries are the front-desk forms that make requests easy to fill out.
- System calls are the single approved counter where requests are handed in.
- The kernel is the security and operations office behind the counter. It decides what is allowed and does the work.
- Drivers are the specialists who know how to operate each specific machine.
- Hardware is the equipment.

Nobody outside the office operates the equipment directly.

Short form:

```text
Application -> Library -> System Call -> Kernel -> Driver -> Hardware
```

## 8. Common Misconceptions

**"The kernel is the whole operating system."** The kernel is the core. Libraries, the shell, services and utilities are part of the complete system but run in user space.

**"The shell is part of the kernel."** The shell is an ordinary user-space program. It can be replaced without changing the kernel.

**"Applications talk directly to the hardware."** Normally they request services through system calls, and the kernel drives the hardware.

**"System calls and library functions are the same thing."** Library functions are what programmers call. Many of them make system calls internally, and some do not need the kernel at all.

**"Drivers are applications."** Drivers run in kernel space, as part of the kernel's reach into hardware. A faulty driver can destabilize the whole system, unlike a faulty application.

**"A crash in user space brings down the system."** Usually one process fails and the rest of the system continues. Kernel-level faults are the ones that threaten the whole machine.

**"Daemons are part of the kernel."** Daemons are user-space processes that happen to run in the background.

## 9. Professional Relevance

Layered thinking is the basis of troubleshooting on production systems:

- Is the problem in hardware, in the kernel or a driver, in a service, or in an application?
- Does a patch update the kernel, a shared library, or one application? The blast radius differs for each.
- Security controls apply at different layers: kernel access control, service accounts, application configuration.
- Capacity questions map to subsystems: CPU scheduling, memory, storage, network.

System administrators, network engineers, security engineers and SREs all use this layered model, even when they use different tools. It is also how you decide whether a problem is yours to fix or belongs to another team.

## 10. Summary

A Linux system is built in layers. Hardware sits at the bottom. The kernel above it manages processes, memory, filesystems, networking, devices and security, and reaches hardware through device drivers. User-space software, including libraries, the shell, services, daemons and applications, sits above the kernel and requests its services through system calls.

The boundary between user space and kernel space exists for stability, security and isolation. Applications use system libraries, which make system calls, so the kernel remains the single trusted mediator for hardware and shared resources.

## 11. Knowledge Check

1. List the layers of a Linux system from hardware to users.
2. What is the role of a device driver?
3. Name the six subsystems listed for the Linux kernel and state one responsibility of each.
4. What is a system call, and why is it described as a controlled entry point?
5. What role do system libraries play between applications and the kernel?
6. Why is the shell not part of the kernel?
7. What is the difference between user space and kernel space?
8. Why is it safer for applications not to access hardware directly?
9. Why can a faulty driver be more serious than a faulty application?
10. A service crashes but the machine and its other services keep running. Which layer does this suggest and why?

## 12. Completion Checklist

```text
[ ] I can describe the layers from hardware to users
[ ] I can explain the role of CPU, RAM, storage and network devices
[ ] I understand what a device driver does
[ ] I can name the main kernel subsystems and their roles
[ ] I can explain what a system call is
[ ] I understand the role of system libraries
[ ] I can distinguish the shell, services, daemons and applications
[ ] I can explain user space vs kernel space
[ ] I can explain why the separation exists
[ ] I can explain why applications do not control hardware directly
```

## 13. Next Topic

Topic 03 — How Linux Works
