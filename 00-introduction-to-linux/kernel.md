# Topic 05 — Linux Kernel

## 1. Introduction

The Linux kernel is the core software component of a Linux operating-system environment.

It manages communication between applications and hardware while controlling shared resources such as:

- CPU time.
- Memory.
- Storage.
- Network interfaces.
- Hardware devices.
- Processes.
- Security boundaries.

Applications, services, shells, and libraries normally run in user space. The kernel runs in a privileged environment and provides controlled operating-system services to those programs.

```text
Applications and Services
          |
          v
    System Libraries
          |
          v
     System Calls
          |
          v
     Linux Kernel
          |
          v
        Hardware
```

The kernel is essential, but it is not the complete Linux operating system. A usable Linux distribution also includes user-space software, libraries, services, tools, and applications.

This topic explains the Linux kernel conceptually. It does not cover kernel programming, source-code development, kernel compilation, or detailed kernel debugging.

## 2. Learning Objectives

After completing this topic, you should be able to:

- Define the Linux kernel.
- Explain why the kernel is the core of Linux.
- Describe the difference between kernel space and user space.
- Explain the major responsibilities of the kernel.
- Describe process management.
- Explain memory management.
- Describe filesystem and storage interaction.
- Explain Linux networking at a high level.
- Describe the role of device drivers.
- Explain how the kernel provides hardware abstraction.
- Describe kernel security and isolation mechanisms conceptually.
- Explain the purpose of system calls.
- Describe kernel modules at an introductory level.
- Explain what the kernel does when a process needs CPU time.
- Explain what happens when an application requests memory.
- Describe how the kernel handles file access.
- Explain how network data is transmitted.
- Understand why administrators need a working model of the kernel.

## 3. What Is the Linux Kernel?

A **kernel** is the privileged core of an operating system.

It manages the relationship between:

- Applications.
- System services.
- Hardware.
- Shared system resources.

The Linux kernel provides a controlled environment in which multiple programs can run without directly controlling the entire computer.

### Basic relationship

```text
User
  |
  v
Applications
  |
  v
User-Space Software
  |
  v
Linux Kernel
  |
  v
Hardware
```

### Why the kernel is necessary

Without a kernel, each application would need to understand:

- Every CPU architecture.
- Every storage controller.
- Every network adapter.
- Every memory-management detail.
- Every hardware-specific communication method.
- Every security boundary.

The kernel provides common interfaces and coordinates hardware access.

### The kernel is not the complete Linux system

The kernel provides core functionality, but a complete Linux environment also needs:

- System libraries.
- User-space utilities.
- Shells.
- System services.
- Device-management software.
- Package and update systems.
- Configuration files.
- Applications.
- Documentation.

```text
Complete Linux Environment
        |
        +-- Applications
        +-- Services and daemons
        +-- Shells and utilities
        +-- System libraries
        +-- Linux kernel
        +-- Hardware
```

## 4. Kernel Space and User Space

### User space

**User space** is where ordinary programs run.

Examples include:

- Web servers.
- Database systems.
- Shells.
- Monitoring agents.
- Graphical applications.
- System libraries.
- Background services.
- Development tools.

User-space programs normally have restricted access to hardware and kernel memory.

### Kernel space

**Kernel space** is the privileged environment where the Linux kernel operates.

The kernel can:

- Access hardware through drivers.
- Manage physical memory.
- Schedule CPU time.
- Handle network traffic.
- Manage filesystems.
- Enforce security boundaries.
- Respond to system calls.

### Why the separation exists

The separation protects the system from unrestricted application behavior.

If every application could directly access hardware or kernel memory:

- Applications could interfere with one another.
- A programming error could corrupt the system.
- Security controls could be bypassed.
- Hardware resources could be used simultaneously without coordination.
- One process could read another process’s private data.

### Conceptual comparison

| User space | Kernel space |
|---|---|
| Runs applications and services. | Runs the Linux kernel and privileged components. |
| Has restricted hardware access. | Controls hardware through kernel subsystems and drivers. |
| Uses libraries and system calls. | Implements system-call handling and resource management. |
| A failure often affects one process or service. | A serious failure can affect the entire system. |
| Contains web servers, databases, shells, and utilities. | Contains schedulers, memory management, networking, filesystems, and drivers. |

This separation improves isolation, but it is not an absolute guarantee of security. Kernel vulnerabilities, unsafe drivers, and configuration problems can still affect the whole system.

## 5. Main Kernel Responsibilities

The Linux kernel has several major responsibilities.

```text
Linux Kernel
    |
    +-- Process Management
    +-- CPU Scheduling
    +-- Memory Management
    +-- Filesystem and Storage Interaction
    +-- Networking
    +-- Device Drivers
    +-- Security and Isolation
    +-- Inter-Process Communication
    +-- Hardware Abstraction
    +-- System-Call Interface
```

## 6. Process Management

A **process** is a running instance of a program.

The kernel manages the life cycle of processes.

It:

- Creates processes.
- Assigns process identifiers.
- Schedules processes.
- Tracks process state.
- Provides process isolation.
- Manages parent-child relationships.
- Ends processes.
- Reclaims process resources.

### Process relationship

```text
Parent Process
      |
      +---- Child Process
      |
      +---- Child Process
```

Processes may create other processes. The kernel records these relationships and controls how processes interact.

Detailed process administration belongs to a later topic. At this stage, the main principle is:

> The kernel turns program code into controlled, schedulable execution.

## 7. CPU Scheduling

A CPU can execute only a limited number of instruction streams at one time. The kernel’s scheduler determines which runnable process receives CPU time.

```text
Runnable Processes
   |       |       |
   v       v       v
Process A Process B Process C
       \     |     /
        \    |    /
          Scheduler
              |
              v
          CPU Cores
```

### What happens when a process needs CPU time?

1. The process becomes runnable.
2. The kernel records it as eligible for execution.
3. The scheduler evaluates runnable processes.
4. A CPU core begins executing the process.
5. The process runs for a period or until it waits for another resource.
6. The scheduler gives CPU time to another runnable process.
7. The original process may run again later.

### Processes do not always use the CPU

A process may be waiting for:

- Storage.
- Network data.
- User input.
- Another process.
- A timer.
- A hardware device.

When a process is waiting, the kernel can run another process.

### Why CPU scheduling matters

CPU scheduling allows Linux to support:

- Multiple applications.
- Background services.
- Interactive workloads.
- Network servers.
- Database systems.
- Monitoring agents.
- Batch tasks.

The scheduler aims to balance responsiveness, fairness, priority, and throughput according to the system and workload.

## 8. Memory Management

The kernel manages both physical memory and the virtual memory environments used by processes.

### Physical memory

Physical memory refers to actual RAM installed in the system.

The kernel uses RAM for:

- Running processes.
- Kernel data structures.
- Filesystem caches.
- Network buffers.
- Shared libraries.
- Temporary data.

### Virtual memory

Virtual memory gives each process a controlled address space.

This provides:

- Process isolation.
- Memory protection.
- Flexible memory allocation.
- Shared libraries.
- Memory mapping.
- Efficient use of physical memory.

A process does not normally need to know which physical RAM location stores its data.

### What happens when an application requests memory?

1. The application requests memory through a library or operating-system interface.
2. The request reaches the kernel.
3. The kernel checks resource limits and availability.
4. The memory manager creates or expands a virtual memory region.
5. Physical memory may be assigned immediately or when the memory is first used.
6. The application receives a usable memory area.
7. The kernel tracks the allocation and protects it from unrelated processes.

### Memory protection

The kernel prevents ordinary processes from freely accessing:

- Another process’s memory.
- Kernel memory.
- Protected hardware mappings.
- Restricted system resources.

This protection helps prevent one application from corrupting another application.

### Memory pressure

If available memory becomes limited, the kernel may:

- Reclaim filesystem cache.
- Reuse memory that is no longer active.
- Move some memory to storage-backed areas.
- Apply resource policies.
- Terminate processes in severe conditions.

The exact behavior depends on system configuration and workload conditions.

## 9. Filesystems and Storage

The kernel provides the interfaces required for applications to access files and storage.

### Storage layers

A simplified storage path looks like this:

```text
Application
     |
     v
System Call
     |
     v
Virtual Filesystem Layer
     |
     v
Filesystem Implementation
     |
     v
Block Storage Layer
     |
     v
Storage Driver
     |
     v
Storage Device
```

### Filesystem abstraction

Linux supports multiple filesystem types. The kernel provides common interfaces so applications can work with files without needing to understand the internal design of every filesystem.

This abstraction allows an application to work with data stored on:

- Local disks.
- Solid-state storage.
- Virtual disks.
- Network filesystems.
- Removable devices.
- Specialized storage systems.

### What happens when a file is accessed?

1. The application requests access to a file.
2. The request enters the kernel through a system call.
3. The kernel identifies the process and its credentials.
4. The kernel checks access permissions.
5. The filesystem layer locates the file.
6. The kernel checks whether the data is already cached.
7. If cached, the data may be returned from memory.
8. If not cached, the storage layer requests data from the device.
9. A storage driver communicates with the hardware.
10. The kernel returns the data or an error to the application.

### Caching

Linux may retain frequently accessed data in memory.

```text
Application
     |
     v
Linux Kernel
     |
     v
Memory Cache
     |
     v
Application
```

If the data is not cached, the kernel may need to access persistent storage.

This is why storage performance cannot be understood separately from memory, caching, workloads, and application behavior.

## 10. Networking

The Linux kernel provides the networking functions used by applications and services.

### Main networking responsibilities

The kernel handles:

- Network interfaces.
- Protocol processing.
- Routing.
- Network sockets.
- Packet buffers.
- Traffic delivery.
- Virtual network devices.
- Network-related security controls.

### Network communication path

```text
Application
     |
     v
Network Library
     |
     v
System Call
     |
     v
Linux Network Stack
     |
     v
Routing and Security Decisions
     |
     v
Network Driver
     |
     v
Network Interface
     |
     v
Network
```

### What happens when network data is transmitted?

1. An application prepares data.
2. A library prepares a network request.
3. The request reaches the kernel.
4. The kernel identifies the destination and network endpoint.
5. Routing decisions determine where the data should go.
6. Network protocols add required information.
7. Security and resource policies are applied.
8. The data is placed into buffers.
9. The network driver prepares the hardware.
10. The network interface transmits the data.

### Receiving data

The reverse process occurs for incoming traffic:

```text
Network
   |
   v
Network Interface
   |
   v
Network Driver
   |
   v
Linux Network Stack
   |
   v
Application Endpoint
   |
   v
Application
```

The kernel receives data from the device, processes it, identifies the intended application, and makes it available to that application.

## 11. Device Drivers

A **device driver** is software that allows the kernel to communicate with a particular hardware device.

Drivers may support:

- Storage devices.
- Network adapters.
- USB devices.
- Graphics hardware.
- Audio devices.
- Sensors.
- Serial devices.
- Input devices.

### Driver relationship

```text
Application
     |
     v
System Call
     |
     v
Kernel Subsystem
     |
     v
Device Driver
     |
     v
Hardware
```

### Why drivers are needed

Hardware devices differ in:

- Communication methods.
- Registers.
- Commands.
- Data formats.
- Timing.
- Error behavior.
- Power-management features.

The driver hides these hardware-specific details behind a kernel-managed interface.

### Driver example

A storage application does not normally need to know:

- Which electrical interface the disk uses.
- Which physical controller manages it.
- Which device registers contain status information.
- How the device signals completion.

The storage driver and kernel subsystem handle these details.

### Driver limitations

If a suitable driver is unavailable or incomplete:

- The hardware may not work.
- Some device features may be unavailable.
- Performance may be reduced.
- The system may require additional firmware.
- The device may be supported only by a vendor-specific component.

## 12. Hardware Abstraction

**Hardware abstraction** means presenting hardware through consistent software interfaces.

```text
Application
     |
     v
Common Operating-System Interface
     |
     v
Kernel Subsystem
     |
     v
Hardware-Specific Driver
     |
     v
Hardware
```

An application can request file access, network communication, or memory without knowing the specific hardware implementation.

### Why abstraction matters

Hardware abstraction provides:

- Portability.
- Consistent application interfaces.
- Centralized resource management.
- Hardware-specific driver isolation.
- Easier software development.
- Better resource sharing.

For example, an application can write data to a file whether the underlying storage is:

- A hard disk.
- An SSD.
- A virtual disk.
- A network filesystem.

The application uses the operating-system interface while the kernel and drivers handle the hardware differences.

## 13. System Calls

A **system call** is a controlled interface through which user-space software requests services from the kernel.

### System-call path

```text
User-Space Application
          |
          v
System Library
          |
          v
System Call Interface
          |
          v
Linux Kernel
          |
          v
Kernel Subsystem
```

### Common system-call categories

System calls may request:

- Process creation.
- Process termination.
- Memory operations.
- File access.
- Network communication.
- Device access.
- Inter-process communication.
- Time and timer operations.
- Identity and permission operations.

### Kernel checks

Before completing a request, the kernel may check:

- Whether the request is valid.
- Whether the memory address is allowed.
- Whether the process has permission.
- Whether a resource limit has been reached.
- Whether the requested device exists.
- Whether the system has enough resources.

### Results and errors

The kernel may return:

- Requested data.
- A success result.
- A process identifier.
- A memory mapping.
- A completion status.
- An error condition.

The application then decides how to handle the result.

## 14. Security and Isolation

Security is one of the kernel’s core responsibilities.

### Identity

The kernel associates processes with identities such as:

- User identity.
- Group identity.
- Effective credentials.
- Security labels or policies.

These identities help determine what a process can access.

### Permission checks

The kernel can check whether a process is allowed to:

- Read a file.
- Modify a file.
- Execute an operation.
- Access a device.
- Open a network endpoint.
- Perform a privileged action.

### Process isolation

The kernel separates processes so that one process normally cannot directly modify another process’s memory.

Process isolation helps protect:

- Application data.
- Credentials.
- Program state.
- Kernel resources.
- System stability.

### Resource isolation

The kernel can restrict resources such as:

- CPU.
- Memory.
- Number of processes.
- Storage operations.
- Network operations.

This is especially important for multi-user systems, containers, and shared infrastructure.

### Security modules and policies

Linux supports additional security mechanisms that can enforce policies beyond basic user and group permissions.

These mechanisms may restrict:

- Which files a process can access.
- Which capabilities it can use.
- Which system operations it can perform.
- How services interact with one another.

The Linux kernel user-space API documentation includes security-related interfaces such as seccomp, Landlock, and Linux Security Modules. [kernel](https://www.kernel.org/doc/html/v6.11/userspace-api/index.html)

Security is not provided by the kernel alone. Secure applications, correct configuration, updates, monitoring, and operational controls are also necessary.

## 15. Kernel Modules

A **kernel module** is an optional component that can extend kernel functionality.

Modules may provide:

- Device drivers.
- Filesystem support.
- Network features.
- Hardware-specific functionality.
- Other kernel capabilities.

### Why modules exist

Kernel modules allow functionality to be added without placing every possible feature directly into the core kernel image.

They can help:

- Support different hardware.
- Reduce the permanently loaded kernel footprint.
- Add filesystem support.
- Adapt to changing hardware configurations.
- Extend kernel capabilities during system operation.

### Conceptual relationship

```text
Linux Kernel Core
       |
       +---- Built-in functionality
       |
       +---- Loadable kernel modules
                    |
                    +---- Device drivers
                    +---- Filesystem support
                    +---- Network features
```

Kernel modules run with kernel-level privileges. They are not ordinary user-space applications.

### Modules and security

Because modules operate with high privileges:

- They must come from trusted sources.
- Compatibility matters.
- Incorrect modules can destabilize a system.
- Security controls may restrict which modules can load.
- Some systems require module signing.

Modern Linux systems may load modules automatically when required, depending on the hardware and distribution configuration. Enterprise documentation describes modules as a way to extend kernel functionality without necessarily rebooting the system. [docs.redhat](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/kernel_administration_guide/chap-documentation-kernel_administration_guide-working_with_kernel_modules)

This lesson does not cover module-management commands or module development.

## 16. What the Kernel Does in Common Situations

### A process needs CPU time

```text
Process Becomes Runnable
          |
          v
Kernel Scheduler Evaluates It
          |
          v
CPU Time Is Assigned
          |
          v
Process Executes
```

The scheduler may later pause the process so another runnable process can execute.

### An application requests memory

```text
Application Requests Memory
          |
          v
Kernel Validates Request
          |
          v
Virtual Memory Region Assigned
          |
          v
Physical Memory Used as Needed
          |
          v
Application Uses the Memory
```

The kernel tracks ownership and protects the allocation from unrelated processes.

### A file is accessed

```text
Application
     |
     v
System Call
     |
     v
Permission Check
     |
     v
Filesystem Layer
     |
     v
Cache or Storage Driver
     |
     v
Data Returned
```

The data may come from memory or persistent storage.

### Network data is transmitted

```text
Application
     |
     v
Network System Call
     |
     v
Kernel Network Stack
     |
     v
Routing and Policy Checks
     |
     v
Network Driver
     |
     v
Network Interface
```

The kernel coordinates protocols, buffers, routing, and device communication.

### Hardware is accessed

```text
Application
     |
     v
User-Space Interface
     |
     v
System Call
     |
     v
Kernel Subsystem
     |
     v
Device Driver
     |
     v
Hardware
```

The application uses a controlled operating-system interface instead of directly controlling device hardware.

## 17. Linux Kernel Architecture at a High Level

Linux is commonly described as a **monolithic kernel with modular capabilities**.

### Monolithic kernel concept

A monolithic kernel contains many core operating-system services in kernel space, including:

- Process management.
- Memory management.
- Networking.
- Filesystem support.
- Device management.
- Security mechanisms.

This does not mean the kernel is one inseparable file with no modular structure. Linux has well-defined subsystems and can use loadable modules.

### Modular capabilities

Modules allow selected capabilities to be added or removed according to system needs.

```text
Kernel Space
    |
    +-- Core kernel
    +-- Process management
    +-- Memory management
    +-- Networking
    +-- Filesystems
    +-- Built-in drivers
    +-- Loadable modules
```

The distinction is useful:

- **Monolithic** describes the broad kernel design.
- **Modular** describes the ability to extend or configure kernel functionality.
- **User space** remains separate from kernel space even when modules are loaded.

This topic does not discuss kernel source code or implementation details.

## 18. Real-World Example: Linux Application Server

Consider a Linux application server running a backend service.

```text
Client Request
      |
      v
Network Interface
      |
      v
Linux Network Stack
      |
      v
Application Process
      |
      +---- Memory Management
      |
      +---- Filesystem Access
      |
      +---- Database Network Connection
      |
      v
Response to Client
```

### Kernel responsibilities

While the application runs, the kernel:

- Schedules the application process.
- Allocates memory.
- Protects the application’s memory.
- Receives network data.
- Sends network data.
- Provides filesystem access.
- Enforces process identity and permissions.
- Communicates with network and storage hardware.

### Application responsibilities

The application:

- Parses the request.
- Applies business logic.
- Validates application data.
- Communicates with a database.
- Creates a response.

The kernel does not understand the application’s business rules. It provides the operating-system resources the application needs.

## 19. Enterprise Perspective

A production environment may contain several Linux systems:

```text
Internet
   |
   v
Firewall
   |
   v
Load Balancer
   |
   v
Linux Web Servers
   |
   v
Linux Application Servers
   |
   v
Linux Database Servers
   |
   v
Storage and Backup
```

The kernel is present within each Linux server, but the workload differs.

### Web server

The kernel manages:

- Network connections.
- Web-server processes.
- Memory.
- Static-file access.
- Network-interface communication.

### Application server

The kernel manages:

- Application processes.
- Runtime memory.
- Database connections.
- Logging output.
- Network communication.
- Resource limits.

### Database server

The kernel manages:

- Database processes.
- Memory and caches.
- Storage access.
- Filesystem operations.
- Client connections.
- CPU scheduling.

### Monitoring server

The kernel provides access to:

- Process information.
- Memory statistics.
- Network statistics.
- Storage activity.
- System events.

Monitoring software runs in user space but depends on kernel interfaces to observe the system.

## 20. Practical Mental Model

Use this model to remember the kernel’s role:

```text
Applications Request Services
            |
            v
System Calls Cross the Boundary
            |
            v
Kernel Checks and Coordinates
            |
            v
Subsystem Handles the Operation
            |
            v
Driver Communicates with Hardware
            |
            v
Result Returns to the Application
```

A shorter model is:

```text
Request
  |
Check
  |
Schedule
  |
Access
  |
Return
```

For nearly every kernel-managed operation, ask:

1. What is the application requesting?
2. Which system interface receives the request?
3. Which kernel subsystem handles it?
4. Is a permission or resource check required?
5. Does the operation need hardware?
6. Could the result come from memory or a cache?
7. What result or error returns to the application?

## 21. Common Misconceptions

### “The Linux kernel is the complete operating system”

The kernel is the core component. A complete Linux environment also includes user-space libraries, utilities, services, interfaces, and applications.

### “The kernel runs all programs”

Applications and services normally run in user space. The kernel manages their execution and provides services.

### “User space cannot communicate with the kernel”

User-space programs communicate with the kernel through system calls and other defined interfaces.

### “Kernel space and user space are separate computers”

They are separate privilege areas within the same system.

### “The kernel directly understands application business logic”

The kernel manages generic resources. It does not understand whether a web request represents a product search, payment, or user profile operation.

### “Every file access reaches the disk”

Linux may satisfy file access from memory caches or other software-managed resources.

### “A device driver is an application”

A device driver operates as part of the kernel or as a kernel module and has much greater privileges than an ordinary application.

### “Kernel modules are ordinary libraries”

Kernel modules extend kernel functionality and run with kernel-level privileges. They are not ordinary user-space libraries.

### “More CPU usage always means a process is healthy”

High CPU usage may be expected, inefficient, blocked by application design, or caused by a runaway process. The kernel schedules CPU time but does not determine whether the application is doing useful work.

### “Linux security is provided only by the kernel”

The kernel provides important mechanisms, but secure systems also require application security, correct configuration, updates, identity management, monitoring, and operational controls.

## 22. Professional Relevance

### System administration

Kernel knowledge helps administrators reason about:

- CPU saturation.
- Memory pressure.
- Storage latency.
- Network failures.
- Device problems.
- Process isolation.
- Resource limits.
- Hardware compatibility.

### System engineering

System engineers use kernel concepts to design:

- Server architectures.
- Resource boundaries.
- Storage systems.
- Network platforms.
- Virtualization hosts.
- Container environments.
- Hardware and software standards.

### Network engineering

Network engineers need to understand:

- The Linux network stack.
- Network interfaces and drivers.
- Routing and packet processing.
- Virtual networking.
- Network resource usage.
- Application-to-network communication.

### Security engineering

Security engineers work with:

- User-space and kernel-space boundaries.
- System calls.
- Process isolation.
- Device access.
- Security policies.
- Resource restrictions.
- Kernel-related events.

### Cloud engineering

Cloud engineers interact with:

- Linux kernels inside virtual machines.
- Virtual CPUs and memory.
- Virtual disks.
- Virtual network devices.
- Kernel resource limits.
- Container hosts.

### DevOps and SRE

DevOps engineers and SREs use kernel concepts to understand:

- Container resource limits.
- Service behavior.
- Application performance.
- Monitoring metrics.
- Storage and network bottlenecks.
- System reliability.

### Backend development

Backend developers benefit from understanding:

- Process execution.
- Memory allocation.
- File access.
- Network communication.
- Resource limits.
- Application behavior under load.

## 23. Summary

The Linux kernel is the privileged core of a Linux operating system.

Its main responsibilities include:

- Process management.
- CPU scheduling.
- Memory management.
- Filesystem and storage interaction.
- Networking.
- Device management.
- Security and isolation.
- Hardware abstraction.
- System-call handling.

Applications run in user space and request kernel services through system calls and system libraries.

The kernel checks requests, selects the appropriate subsystem, manages resources, communicates with hardware when necessary, and returns a result or error.

Kernel modules extend kernel functionality by providing optional support such as device drivers and filesystem features.

The main operational principle is:

> The application requests, the kernel controls, the subsystem operates, the driver communicates, and the result returns.

## 24. Knowledge Check

1. What is the Linux kernel?
2. Why is the kernel considered the core of Linux?
3. What is the difference between user space and kernel space?
4. Why does the separation between user space and kernel space exist?
5. What responsibilities belong to process management?
6. What happens when a process needs CPU time?
7. What is virtual memory?
8. What happens when an application requests memory?
9. How does the kernel handle file access?
10. Why might file data be returned from memory instead of storage?
11. What does the Linux networking subsystem do?
12. What is the role of a device driver?
13. What is hardware abstraction?
14. What is a system call?
15. How does the kernel enforce security and isolation?
16. What is a kernel module?
17. Why are kernel modules more privileged than ordinary applications?
18. How does a Linux application server depend on the kernel?
19. Why is the Linux kernel not the complete operating system?
20. Which kernel subsystem would you investigate for a storage, memory, CPU, or network problem?

## 25. Completion Checklist

You should now understand:

- [ ] What the Linux kernel is.
- [ ] Why the kernel is central to Linux.
- [ ] The difference between kernel space and user space.
- [ ] Why privilege separation exists.
- [ ] Process management.
- [ ] CPU scheduling.
- [ ] Memory management.
- [ ] Virtual memory and memory protection.
- [ ] Filesystem and storage interaction.
- [ ] Linux networking.
- [ ] Device drivers.
- [ ] Hardware abstraction.
- [ ] System calls.
- [ ] Permission and resource checks.
- [ ] Security and process isolation.
- [ ] Inter-process communication at a conceptual level.
- [ ] Kernel modules.
- [ ] What happens when a process needs CPU time.
- [ ] What happens when an application requests memory.
- [ ] How file access reaches storage.
- [ ] How network data reaches a device.
- [ ] How the kernel returns results and errors.
- [ ] Why administrators and engineers need kernel knowledge.

## 26. Next Topic

**Topic 06 — Linux Distributions**
