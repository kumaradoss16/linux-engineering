# Topic 02 - Linux Architecture

## 1. Introduction

Linux architecture describes how users, applications, system software, the Linux kernel, and hardware work together.

A Linux system is not a single program. It is a collection of layers and components with different responsibilities:

- Hardware provides physical resources.
- The Linux kernel manages those resources.
- System libraries provide programming interfaces.
- System services perform background tasks.
- Shells and graphical interfaces allow interaction.
- Applications perform useful work for users and other systems.

Understanding this structure helps explain what happens when an application reads a file, uses memory, sends network traffic, or communicates with a device.

This topic focuses on the high-level architecture of Linux. It does not cover commands, kernel programming, source-code implementation, or advanced system internals.

## 2. Learning Objectives

After completing this topic, you should be able to:

- Identify the main layers of a Linux system.
- Explain the relationship between hardware and the Linux kernel.
- Describe the role of CPU, RAM, storage, and network devices.
- Explain the purpose of device drivers.
- Describe the major responsibilities of Linux kernel subsystems.
- Distinguish user space from kernel space.
- Explain what system calls are.
- Describe the role of system libraries.
- Explain the difference between system services and applications.
- Describe how users interact with Linux.
- Explain why applications normally do not control hardware directly.
- Trace the general path between an application and a hardware resource.
- Apply the architecture to a realistic server environment.

## 3. Complete Linux Architecture

The following diagram shows the main conceptual layers of a Linux system:

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

This is a conceptual architecture. Real Linux systems contain additional components, and not every application follows exactly the same path for every operation.

The main relationship is:

```text
User
  |
  v
Application
  |
  v
User-space software
  |
  v
System call interface
  |
  v
Linux kernel
  |
  v
Hardware or system resource
```

## 4. Hardware Layer

Hardware is the physical foundation of a Linux system.

### CPU

The **Central Processing Unit**, or CPU, executes instructions.

The CPU performs calculations and runs instructions belonging to:

- The Linux kernel.
- System services.
- Applications.
- Device-management software.
- Other user-space programs.

A system with multiple CPU cores can execute multiple instruction streams at the same time, although the kernel still needs to schedule work across those cores.

The CPU supports different privilege levels. Linux uses these levels to separate ordinary application execution from privileged kernel operations.

### RAM

**Random Access Memory**, or RAM, stores data and instructions that are actively being used.

Linux uses RAM for:

- Running processes.
- Kernel data structures.
- Application data.
- Filesystem caches.
- Network buffers.
- Shared libraries.
- Temporary working data.

RAM is faster than persistent storage but is normally volatile. Its contents are lost when the system loses power.

### Storage

Storage devices retain data after power is removed.

Examples include:

- Hard disk drives.
- Solid-state drives.
- NVMe devices.
- USB storage.
- Virtual disks.
- Network-attached storage.

Linux interacts with storage through layers that include:

- Device drivers.
- Block-device subsystems.
- Filesystem implementations.
- Caching mechanisms.
- User-space applications.

Applications normally work with files and directories rather than directly controlling storage sectors.

### Network devices

Network devices connect a Linux system to other systems.

Examples include:

- Ethernet adapters.
- Wireless adapters.
- Virtual network interfaces.
- Fibre Channel adapters.
- Cloud-provider virtual network devices.

The Linux kernel coordinates communication between network applications and these devices.

### Other hardware

Linux may also interact with:

- GPUs.
- USB devices.
- Audio hardware.
- Cameras.
- Sensors.
- Printers.
- Serial devices.
- Input devices.

Each device requires suitable support through a driver or kernel subsystem.

## 5. Linux Kernel Layer

The Linux kernel is the privileged core of the system. It manages hardware and provides controlled services to software running in user space.

The kernel is not the complete Linux operating system. It is the central layer that connects user-space software with hardware and system resources.

### Main kernel responsibilities

The kernel manages:

- Processes and CPU time.
- Memory.
- Filesystems and storage access.
- Networking.
- Hardware devices.
- Security boundaries.
- Inter-process communication.
- Timers and interrupts.
- Resource allocation.

The kernel documentation describes separate areas for user-space APIs, memory management, filesystems, device support, and other subsystems. [docs.kernel](https://docs.kernel.org/userspace-api/index.html)

## 6. Kernel Subsystems

A **kernel subsystem** is a major part of the kernel responsible for a particular area of system management.

### Process management

The process-management subsystem handles running programs.

It is responsible for:

- Creating processes.
- Scheduling processes.
- Assigning CPU time.
- Managing process states.
- Ending processes.
- Supporting communication between processes.

A process is a running instance of a program. Process administration is covered in a later topic.

### Memory management

The memory-management subsystem controls how memory is allocated and used.

It handles:

- Physical memory.
- Virtual memory.
- Process address spaces.
- Memory protection.
- Memory allocation.
- File-backed memory.
- Memory reclamation.

Linux memory management includes mechanisms such as virtual memory and demand paging. [docs.kernel](https://docs.kernel.org/admin-guide/mm/index.html)

### Filesystem management

The filesystem subsystem provides a common way for software to access stored data.

It helps Linux work with different filesystem types through a consistent interface.

The filesystem layer coordinates:

- Files.
- Directories.
- Metadata.
- Storage requests.
- Caching.
- Access checks.
- Filesystem-specific behavior.

Different filesystems may store data differently, but applications can use common operating-system interfaces.

Detailed filesystem structure and administration belong to later topics.

### Networking

The networking subsystem manages communication between local applications, network interfaces, and remote systems.

It supports:

- Network protocols.
- Network sockets.
- Routing.
- Network interfaces.
- Packet handling.
- Network buffering.
- Virtual networking.

A web server, database client, monitoring system, and DNS service can all use kernel networking facilities.

### Device drivers

A **device driver** is software that allows the kernel to communicate with a particular hardware device.

A driver may handle:

- Device initialization.
- Data transfer.
- Device status.
- Interrupts.
- Device-specific operations.
- Power-management functions.

The driver hides hardware-specific details from most applications.

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
Hardware Device
```

Applications generally do not write directly to hardware registers or control electrical signals. They request services from the kernel, which uses the appropriate driver.

### Security

The kernel contributes to security by enforcing boundaries between:

- Users.
- Processes.
- Applications.
- Kernel resources.
- Hardware.
- Network operations.

Security features may include:

- User and group identities.
- Permission checks.
- Process isolation.
- Resource restrictions.
- Security modules.
- Capabilities.
- Auditing interfaces.

The kernel provides mechanisms, but secure operation also depends on system configuration, applications, updates, monitoring, and administrator decisions.

### Inter-process communication

**Inter-process communication**, or IPC, allows processes to exchange information or coordinate work.

Examples of communication mechanisms include:

- Signals.
- Pipes.
- Shared memory.
- Sockets.
- Message queues.

The kernel helps create, control, and protect these communication paths.

## 7. User Space and Kernel Space

### User space

**User space** is the area where ordinary software runs.

Examples include:

- Applications.
- System libraries.
- Shells.
- Graphical interfaces.
- System services.
- Daemons.
- Administration tools.
- Monitoring software.

User-space programs normally run with restricted privileges. They cannot freely access all memory, hardware devices, or kernel data structures.

### Kernel space

**Kernel space** is the privileged execution area used by the Linux kernel.

The kernel can:

- Manage CPU scheduling.
- Access hardware through drivers.
- Control memory mappings.
- Handle system calls.
- Manage filesystems.
- Process network traffic.
- Enforce security boundaries.

### Why the separation exists

The separation between user space and kernel space provides:

- Protection against accidental damage.
- Isolation between applications.
- Controlled hardware access.
- Better system security.
- A stable interface between applications and the kernel.

If every application could directly modify hardware or kernel memory, one faulty or malicious program could damage the entire system.

### Conceptual comparison

| User space | Kernel space |
|---|---|
| Runs applications and services. | Runs the Linux kernel and privileged components. |
| Has restricted hardware access. | Has controlled access to hardware and system resources. |
| A failure usually affects one process or service. | A serious failure can affect the entire system. |
| Uses system libraries and system calls. | Implements system-call handling and resource management. |
| Contains web servers, databases, shells, and utilities. | Contains schedulers, memory management, filesystems, networking, and drivers. |

The boundary is not a complete security guarantee. Vulnerabilities, misconfigurations, unsafe drivers, and other failures can still affect system security.

## 8. System Calls

A **system call** is a controlled request from a user-space program to the Linux kernel.

Applications use system calls when they need services that require kernel involvement.

Common categories include:

- Process creation and termination.
- Memory allocation and mapping.
- File access.
- Network communication.
- Device access.
- Time and timer operations.
- Inter-process communication.
- Permission and identity operations.

### Why system calls are needed

Applications do not normally access hardware directly because:

- Hardware differs between systems.
- Direct access could corrupt data.
- Applications should not bypass security checks.
- The kernel must coordinate shared resources.
- The operating system needs a consistent interface.

System calls give applications controlled access to operating-system services.

### General system-call flow

```text
Application
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
     |
     v
Hardware or System Resource
```

The kernel validates the request, checks permissions and resource availability, performs or schedules the operation, and returns a result.

### System-call example: accessing a file

Conceptually, an application reading a file follows a flow such as:

```text
Application requests data
          |
          v
System library prepares the request
          |
          v
System call enters the kernel
          |
          v
Kernel checks the request and access
          |
          v
Filesystem subsystem locates the data
          |
          v
Storage driver communicates with hardware
          |
          v
Kernel receives the data
          |
          v
Data is returned to the application
```

The application does not need to understand the internal layout of the storage device.

## 9. System Libraries and APIs

### What is a system library?

A **system library** is reusable software that provides functions applications can call.

Libraries help applications interact with:

- Files.
- Processes.
- Memory.
- Networking.
- Time.
- Threads.
- Devices.
- Other system services.

A library often provides a more convenient interface than calling the kernel directly.

### What is an API?

An **Application Programming Interface**, or API, is a defined interface that allows one piece of software to request functions from another piece of software.

A system library can provide an API to an application. The library may then use system calls to request services from the kernel.

```text
Application
    |
    v
Library API
    |
    v
System Call
    |
    v
Linux Kernel
```

### Why libraries are useful

System libraries:

- Reduce repeated application code.
- Hide low-level details.
- Improve portability.
- Provide consistent interfaces.
- Help applications use operating-system functions safely.
- Convert application requests into system-call operations.

For example, a programming language may provide a file-reading function. Internally, that function may use a library interface, which eventually requests file access from the kernel.

### Libraries are not the kernel

A library runs in user space. It does not replace the kernel.

The library prepares a request. The kernel performs the privileged operation and controls access to the resource.

## 10. Shells, User Interfaces, and Applications

### Users

Users interact with Linux through:

- Applications.
- Shells.
- Graphical interfaces.
- Remote administration tools.
- APIs.
- Automated systems.

A user may be a human, but a service or automated process can also act as a system user.

### Applications

Applications perform useful work.

Examples include:

- Web servers.
- Database systems.
- Browsers.
- Programming environments.
- Monitoring platforms.
- Backup systems.
- Security applications.

Applications usually run in user space and rely on libraries, system services, and kernel facilities.

### Shells

A **shell** is a command interpreter that provides an interface between a user and the operating system.

A shell can:

- Start programs.
- Pass input to programs.
- Display output.
- Connect programs together.
- Manage environment information.
- Interpret user instructions.

The shell is a user-space program. It is not the Linux kernel.

Shell usage and commands belong to a later topic.

### Graphical user interfaces

A graphical interface provides another way to interact with Linux.

It may include:

- Windows.
- Menus.
- Desktop applications.
- File browsers.
- Configuration panels.
- Terminal emulators.

Both graphical applications and shell-based tools ultimately rely on system libraries, services, and kernel interfaces.

## 11. System Services and Daemons

A **system service** is a background function that provides a capability to the operating system or other applications.

A **daemon** is a background process that commonly provides a service.

Examples include services for:

- Networking.
- Logging.
- Time synchronization.
- Web hosting.
- Databases.
- Monitoring.
- Remote administration.
- Scheduling.

The relationship can be represented as:

```text
Application or User
        |
        v
System Service
        |
        v
Daemon Process
        |
        v
Linux Kernel
        |
        v
Hardware / Network / Storage
```

A daemon is a process in user space, not part of the kernel. It uses kernel services to access system resources.

Service management, service dependencies, and daemon administration are covered later.

## 12. How Applications Communicate with the Kernel

A typical application request follows this general path:

```text
Application
     |
     v
System Library / API
     |
     v
System Call
     |
     v
Linux Kernel
     |
     +---- Process Management
     |
     +---- Memory Management
     |
     +---- Filesystem
     |
     +---- Networking
     |
     +---- Device Drivers
     |
     v
Hardware / System Resource
```

### Step-by-step flow

1. The application decides it needs an operating-system service.
2. It calls a function provided by a system library or API.
3. The library prepares the request.
4. A system call transfers control to the kernel.
5. The kernel identifies the appropriate subsystem.
6. The kernel checks permissions, resource availability, and request validity.
7. The relevant subsystem performs or schedules the operation.
8. A driver may communicate with physical hardware.
9. The kernel produces a result or error.
10. Control returns to the application.

### Why applications do not directly control hardware

Direct hardware access would create several problems:

- Applications could interfere with one another.
- Data could be corrupted.
- Security boundaries could be bypassed.
- Hardware-specific code would be repeated in every application.
- Resource sharing would become difficult.
- A faulty application could affect the entire machine.

The kernel provides abstraction and coordination.

## 13. Architecture Examples

### Example 1: Reading data from storage

Suppose a database application needs to retrieve a record.

```text
Database Application
        |
        v
Database Library
        |
        v
System Call
        |
        v
Linux Kernel
        |
        v
Filesystem Subsystem
        |
        v
Storage Subsystem
        |
        v
Storage Driver
        |
        v
SSD or Hard Disk
```

The kernel may also use cached data if the requested information is already available in memory.

The database does not need to know whether the data is located on a particular storage sector, SSD cell, or virtual disk block.

### Example 2: Sending network data

Suppose a web application sends a response to a client.

```text
Web Application
        |
        v
Network Library
        |
        v
System Call
        |
        v
Linux Kernel
        |
        v
Network Stack
        |
        v
Network Driver
        |
        v
Network Adapter
        |
        v
Network
```

The kernel handles network protocols, buffering, routing decisions, and communication with the network device.

### Example 3: Starting an application

When one program starts another program:

1. The request begins in user space.
2. The kernel creates or prepares a new process.
3. The kernel assigns process resources.
4. The program is loaded into memory.
5. The process receives an identity and execution context.
6. The scheduler gives it CPU time.
7. The program begins execution in user space.

The exact implementation involves many details, but this model is sufficient for understanding the architecture.

### Example 4: Allocating memory

When an application needs more memory:

1. The application requests memory through a library interface.
2. The request reaches the kernel.
3. The kernel checks available memory and process limits.
4. The kernel creates or extends a virtual memory mapping.
5. Physical memory may be assigned immediately or when the memory is first used.
6. The application receives a usable memory region.

The application sees a controlled virtual address space rather than directly managing physical RAM.

## 14. Real-World Enterprise Perspective

Consider a multi-tier application environment:

```text
Users
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
Storage
  |
  v
Backup Infrastructure
```

Linux architecture appears at each server layer.

### Load balancer

The load balancer receives incoming traffic and distributes it among web servers.

Linux may provide:

- Network interfaces.
- Routing.
- Packet processing.
- Firewall functions.
- Monitoring interfaces.
- Load-balancing software.

### Web servers

Web-server applications run in user space. They use:

- System libraries.
- Network system calls.
- Kernel networking.
- Storage interfaces.
- Process and memory management.

### Application servers

Application servers use CPU, memory, storage, and networking through the operating system.

The Linux kernel schedules their processes and controls their resource access.

### Database servers

Database systems rely on:

- Filesystem and storage interfaces.
- Memory management.
- Process management.
- Network communication.
- Security boundaries.

### Storage and backup infrastructure

Backup applications run in user space, while the kernel provides access to storage devices, filesystems, network interfaces, and data-transfer mechanisms.

### Engineering roles

- **System administrators** work with operating-system configuration, resources, services, storage, and access.
- **System engineers** design how Linux systems interact across infrastructure layers.
- **Network engineers** work with interfaces, routing, network services, and network appliances.
- **Security engineers** analyze isolation, permissions, system boundaries, and security events.
- **DevOps engineers** automate deployment and configuration across Linux systems.
- **Cloud engineers** work with Linux images, virtual hardware, cloud networking, and storage.
- **SREs** observe resource behavior and improve reliability.
- **Backend developers** build applications that use Linux interfaces and services.

## 15. Practical Mental Model

Use the following mental model:

```text
Hardware
    |
    v
Linux Kernel
    |
    +-- CPU and process management
    +-- Memory management
    +-- Filesystems and storage
    +-- Networking
    +-- Device drivers
    +-- Security and isolation
    |
    v
System Libraries and Services
    |
    v
Shells and Applications
    |
    v
Users
```

Remember these relationships:

- Hardware provides resources.
- The kernel controls and coordinates those resources.
- Drivers connect kernel subsystems to hardware.
- System calls provide a controlled boundary into the kernel.
- Libraries make system interfaces easier for applications to use.
- Services perform background functions.
- Applications provide useful functionality.
- Users interact with applications and interfaces.

A concise principle is:

> Applications request services. The kernel controls resources. Hardware performs the physical operation.

## 16. Common Misconceptions

### “The shell is the kernel”

The shell is a user-space program that interprets user instructions. The kernel is the privileged core that manages hardware and system resources.

### “Applications run directly on hardware”

Applications normally run in user space and request privileged services through system calls and libraries.

### “System libraries are part of the kernel”

System libraries run in user space. They provide convenient interfaces that may use system calls to communicate with the kernel.

### “A daemon is part of the kernel”

A daemon is normally a user-space background process. It uses kernel facilities but is separate from the kernel.

### “The kernel handles every application feature”

The kernel provides general operating-system services. Application features such as database queries, web routing, user interfaces, and business logic are implemented by user-space software.

### “User space means the user’s home directory”

User space is an execution and privilege concept. It refers to software running outside the kernel, not to a particular filesystem directory.

### “Kernel space and user space are separate computers”

They are separate privilege areas within the same system, not separate physical machines.

### “Every operation requires physical hardware access”

Some operations can be served from memory, caches, virtual devices, or software-managed resources without immediate physical-device activity.

## 17. Summary

Linux architecture is based on cooperation between hardware, the kernel, user-space software, and users.

The main layers are:

1. Hardware.
2. Linux kernel.
3. System calls.
4. System libraries and APIs.
5. System services and daemons.
6. Shells and user interfaces.
7. Applications.
8. Users.

The kernel provides the central control layer. It manages CPU time, memory, filesystems, networking, devices, security, and resource allocation.

User-space programs cannot normally control hardware directly. They use libraries and system calls to request services from the kernel.

Device drivers allow kernel subsystems to communicate with particular hardware devices.

The separation between user space and kernel space improves isolation, security, and resource control.

Understanding this architecture makes later topics easier because commands and administrative actions can be connected to the components they affect.

## 18. Knowledge Check

1. What are the major layers of a Linux system?
2. What is the role of the Linux kernel?
3. What is the difference between user space and kernel space?
4. Why do applications normally use system calls instead of controlling hardware directly?
5. What is a system call?
6. What is the purpose of a system library?
7. What is a device driver?
8. What responsibilities belong to the process-management subsystem?
9. What responsibilities belong to the memory-management subsystem?
10. How does a database application access data from storage?
11. How does a web application send data through a network interface?
12. What is the difference between a system service and a Linux kernel subsystem?
13. Why is isolation between applications useful?
14. How can Linux architecture appear in an enterprise application environment?

## 19. Completion Checklist

You should now understand:

- [ ] The main layers of Linux architecture.
- [ ] The role of CPU, RAM, storage, and network devices.
- [ ] The purpose of the Linux kernel.
- [ ] Process-management responsibilities.
- [ ] Memory-management responsibilities.
- [ ] Filesystem and storage responsibilities.
- [ ] Networking responsibilities.
- [ ] Device-driver responsibilities.
- [ ] Kernel security and isolation responsibilities.
- [ ] The meaning of user space.
- [ ] The meaning of kernel space.
- [ ] Why user space and kernel space are separated.
- [ ] What system calls are.
- [ ] Why applications use system calls.
- [ ] The role of system libraries and APIs.
- [ ] The difference between applications and system services.
- [ ] The role of daemons.
- [ ] How an application accesses storage.
- [ ] How an application sends network data.
- [ ] How Linux architecture appears in enterprise infrastructure.
- [ ] Why understanding architecture is more useful than memorizing isolated commands.

## 20. Next Topic

**Topic 03 - How Linux Works**
