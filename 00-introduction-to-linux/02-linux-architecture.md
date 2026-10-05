# Topic 02 — Linux Architecture

## 1. Introduction

Linux architecture describes how the major parts of a Linux system are organized and how they communicate.

A Linux system is not just the kernel. It is a layered environment containing:

- Hardware.
- The Linux kernel.
- Kernel subsystems.
- Device drivers.
- System calls.
- System libraries.
- System services and daemons.
- Shells and user interfaces.
- Applications.
- Users.

Understanding these relationships is more useful than memorizing isolated terms. When a Linux administrator investigates a slow application, unavailable storage, or a network problem, they reason through these layers to identify where the problem exists.

This topic focuses on the high-level architecture of Linux. It does not teach Linux commands, kernel programming, or advanced system internals.

## 2. Learning Objectives

After completing this topic, you should be able to:

- Describe the major layers of Linux architecture.
- Explain the roles of hardware, the kernel, user space, services, and applications.
- Distinguish user space from kernel space.
- Explain why Linux separates ordinary applications from the kernel.
- Describe what system calls are.
- Explain how applications communicate with the kernel.
- Describe the role of system libraries.
- Explain the purpose of device drivers.
- Identify major Linux kernel subsystems.
- Describe the relationship between users, applications, services, libraries, system calls, the kernel, and hardware.
- Explain how the architecture applies to servers, cloud systems, networks, and development environments.

## 3. The Linux Architecture Model

The following diagram shows a simplified Linux architecture:

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
|       System Calls         |
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

This model is conceptual. Real Linux systems contain additional components, interfaces, background services, virtual devices, compatibility layers, and hardware-specific details.

The layers are connected, but they are not interchangeable:

- Applications are not the kernel.
- System libraries are not system calls.
- The kernel is not the complete operating system.
- Hardware is not directly controlled by ordinary applications.
- A shell is an interface, not the Linux kernel itself.

## 4. Hardware

Hardware is the physical or virtual equipment that provides computing resources.

### Main hardware components

#### CPU

The **central processing unit**, or CPU, executes instructions.

Linux uses the CPU to run:

- Applications.
- System libraries.
- Services.
- Kernel code.
- Device-driver code.

The kernel decides how CPU time is shared among processes.

#### RAM

**Random access memory**, or RAM, temporarily stores instructions and data currently being used.

Linux manages:

- Which processes receive memory.
- How much memory they can use.
- Memory protection.
- Virtual memory.
- Reclaiming memory when it is no longer needed.

#### Storage

Storage devices retain data after the system is powered off.

Examples include:

- SSDs.
- Hard disk drives.
- NVMe devices.
- Virtual disks.
- Network storage.

The kernel provides the interfaces that allow applications and filesystems to use storage.

#### Network devices

Network devices connect the system to other systems.

Examples include:

- Ethernet adapters.
- Wireless adapters.
- Virtual network interfaces.
- Virtual switches.
- Specialized network hardware.

Linux networking components communicate with these devices through drivers and kernel subsystems.

#### GPU

A graphics processing unit, or GPU, performs graphics and parallel-processing work. Linux may communicate with GPUs through kernel drivers and user-space graphics libraries.

#### USB and peripheral devices

Linux can communicate with keyboards, cameras, storage devices, sensors, and other peripherals through device drivers and kernel subsystems.

### Physical and virtual hardware

Linux may run directly on physical hardware or inside a virtual machine.

In a virtual machine, the guest Linux system sees virtual hardware such as:

- Virtual CPUs.
- Virtual RAM.
- Virtual disks.
- Virtual network adapters.

The virtualization platform manages the underlying physical hardware, while the Linux kernel manages the hardware presented to the guest system.

## 5. The Linux Kernel

The **Linux kernel** is the privileged core of the operating system. It controls access to hardware and provides fundamental services to user-space software.

The kernel acts as:

- A resource manager.
- A hardware abstraction layer.
- A security boundary.
- A process manager.
- A networking platform.
- A filesystem interface.
- A communication point between applications and hardware.

The kernel abstracts hardware differences so that applications do not need to understand the exact implementation of every CPU, storage device, or network adapter.

### Major kernel responsibilities

#### Process management

The kernel:

- Creates and tracks processes.
- Assigns CPU time.
- Manages process states.
- Isolates processes.
- Supports communication between processes.

#### Memory management

The kernel:

- Allocates memory.
- Protects memory regions.
- Provides virtual memory.
- Tracks memory usage.
- Reclaims memory when possible.

#### Filesystem management

The kernel provides the interfaces that allow software to work with filesystems and storage.

It handles concepts such as:

- Files.
- Directories.
- File metadata.
- Storage access.
- Filesystem drivers.
- Block devices.

The detailed Linux filesystem hierarchy and administration are covered in later topics.

#### Networking

The kernel manages networking functions such as:

- Network interfaces.
- Network protocols.
- Routing.
- Network sockets.
- Packet transmission.
- Packet reception.
- Virtual network devices.

#### Device drivers

Device drivers allow the kernel to communicate with hardware.

A driver translates general operating-system requests into operations understood by a specific device.

#### Security

The kernel helps enforce:

- Process isolation.
- User and group identities.
- Access boundaries.
- Resource restrictions.
- Security policies.

The kernel provides an important security foundation, but a secure Linux system also depends on configuration, updates, applications, monitoring, and administration.

## 6. Kernel Subsystems

The Linux kernel is organized into logical areas commonly called **subsystems**.

A subsystem handles a particular category of operating-system functionality.

```text
Linux Kernel
    |
    +-- Process Management
    +-- Memory Management
    +-- Filesystem Layer
    +-- Networking
    +-- Device Management
    +-- Security
    +-- Inter-Process Communication
    +-- Time Management
```

### Process management subsystem

This subsystem coordinates running processes and CPU scheduling.

### Memory management subsystem

This subsystem manages physical memory, virtual memory, memory allocation, and memory protection.

### Filesystem layer

The filesystem layer provides a common interface for different filesystems and storage devices.

An application can work with files through a consistent interface without needing to understand every hardware-specific storage detail.

### Networking subsystem

The networking subsystem manages protocol processing, network sockets, interfaces, routing, and communication with network drivers.

### Device-management components

These components support device discovery, device access, drivers, and interactions with hardware.

### Security mechanisms

Kernel security mechanisms help enforce access restrictions, isolation, and resource boundaries.

### Inter-process communication

Inter-process communication, or IPC, allows processes to exchange data and coordinate work.

Examples include:

- Signals.
- Pipes.
- Shared memory.
- Sockets.
- Other kernel-supported communication mechanisms.

The details of these mechanisms belong to later topics.

## 7. User Space and Kernel Space

Linux separates ordinary software from the privileged kernel.

```text
+----------------------------------+
|          USER SPACE              |
|                                  |
| Applications                     |
| System Services                  |
| Daemons                          |
| Shells                          |
| System Libraries                 |
+----------------------------------+
                 |
                 | System Calls
                 |
+----------------------------------+
|         KERNEL SPACE             |
|                                  |
| Process Management               |
| Memory Management                |
| Filesystems                      |
| Networking                       |
| Device Drivers                   |
| Security                        |
+----------------------------------+
                 |
                 |
+----------------------------------+
|            HARDWARE              |
+----------------------------------+
```

### User space

**User space** is where ordinary software runs.

It includes:

- Applications.
- System libraries.
- Shells.
- Services.
- Daemons.
- Monitoring software.
- Development tools.
- User interfaces.

User-space programs normally operate with restricted privileges.

### Kernel space

**Kernel space** is the privileged environment where the Linux kernel operates.

It includes:

- Kernel core functions.
- Process-management code.
- Memory-management code.
- Filesystem code.
- Networking code.
- Device drivers.
- Security mechanisms.

Kernel code has access to resources that ordinary user-space programs cannot directly access.

### Why the separation exists

The separation provides:

- Protection from faulty applications.
- Isolation between processes.
- Controlled access to hardware.
- More predictable resource management.
- A security boundary.
- A consistent interface between software and hardware.

Without this separation, one faulty application could potentially overwrite kernel memory, interfere with unrelated processes, or directly manipulate hardware in unsafe ways.

### User mode and kernel mode

The CPU generally runs code in different privilege levels.

At a simplified level:

- **User mode** is used for ordinary applications.
- **Kernel mode** is used for privileged operating-system operations.

User space and kernel space describe software and memory boundaries. User mode and kernel mode describe processor execution privileges. These concepts are closely related but are not exactly the same term.

## 8. System Calls

A **system call** is a controlled entry point through which a user-space program requests a service from the Linux kernel.

System calls are interaction points between user space and the kernel. The Linux kernel documentation describes them as traditional interfaces through which userspace interacts with the kernel. [docs.kernel](https://docs.kernel.org/process/adding-syscalls.html)

### Why system calls exist

Applications need operating-system services such as:

- Creating processes.
- Allocating memory.
- Opening files.
- Reading data.
- Writing data.
- Sending network information.
- Creating network connections.
- Accessing system time.
- Communicating with devices.

Allowing applications to directly perform these operations would create security and stability problems. System calls allow the kernel to:

1. Receive the request.
2. Check the request.
3. Check the caller’s permissions.
4. Perform or reject the operation.
5. Return a result or error.

### System-call flow

```text
Application
     |
     v
System Library
     |
     v
System Call
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

The result returns through the same general path:

```text
Hardware or System Resource
     |
     v
Kernel Subsystem
     |
     v
Linux Kernel
     |
     v
System Call Result
     |
     v
System Library
     |
     v
Application
```

Most applications do not invoke system calls directly. A system library usually provides a more convenient programming interface that prepares the request and communicates with the kernel.

The Linux man-pages project describes system-call interfaces as functions that wrap operations performed by the kernel. [man.archlinux](https://man.archlinux.org/man/man-pages.7.en)

### What happens during a system call

At a conceptual level:

1. An application requests an operating-system service.
2. A system library prepares the request.
3. The CPU transfers controlled execution from user mode to kernel mode.
4. The kernel identifies the requested operation.
5. The kernel checks parameters and permissions.
6. The appropriate kernel subsystem performs the operation.
7. The kernel prepares a result.
8. Execution returns to user space.
9. The application receives the result or an error.

The exact processor instructions and internal implementation vary by architecture and kernel version. Those details are outside the scope of this foundation topic.

## 9. System Libraries and APIs

A **system library** is reusable software that provides functions for applications and other user-space programs.

Libraries can:

- Provide convenient programming interfaces.
- Hide low-level implementation details.
- Prepare system-call arguments.
- Convert results into forms applications can use.
- Provide consistent behavior across programs.

### Library API versus system call

An **API**, or application programming interface, is a defined way for software components to communicate.

A library API may operate entirely in user space. A system-call interface crosses the boundary into the kernel.

```text
Application
     |
     v
Library API
     |
     v
System-Call Wrapper
     |
     v
Linux Kernel
```

An application may call a normal library function, while the library internally uses a system call when kernel services are required.

This means:

- Not every library function is a system call.
- A library may use one or more system calls.
- Some library functions may perform work entirely in user space.
- The kernel provides the privileged operation when hardware or protected resources are involved.

## 10. Shells, Interfaces, and Applications

### Shells

A **shell** is a user-space program that provides an interface for interacting with the operating system.

A shell can:

- Accept user input.
- Start applications.
- Pass information to programs.
- Connect program input and output.
- Use environment information.
- Interpret shell language features.

A shell is not the kernel. It is a user-space interface to the operating system.

Detailed shell usage belongs to a later topic.

### Graphical user interfaces

A graphical user interface provides windows, menus, controls, and visual applications.

It may include:

- Desktop environments.
- Display servers.
- Window managers.
- Graphical system tools.
- Desktop applications.

Linux servers often run without a full graphical desktop because remote administration and server services do not always require one.

### Terminal emulators

A terminal emulator is a user-space application that provides a terminal window. It may run a shell inside that window.

The terminal emulator, shell, and kernel are separate components:

```text
User
  |
  v
Terminal Emulator
  |
  v
Shell
  |
  v
System Libraries / System Calls
  |
  v
Linux Kernel
```

### Applications

Applications are user-space programs that perform useful work.

Examples include:

- Web servers.
- Database systems.
- Browsers.
- Monitoring software.
- Backup systems.
- Development environments.
- Security platforms.
- Network services.

Applications use libraries and operating-system interfaces rather than normally accessing hardware directly.

## 11. System Services and Daemons

A **system service** is a software component that provides a function to other programs, users, or systems.

A **daemon** is a background process that performs a service, often without direct user interaction.

Examples include services that provide:

- Web hosting.
- Databases.
- DNS.
- Logging.
- Monitoring.
- Time synchronization.
- Remote access.
- Backup.
- Network management.

A simplified relationship is:

```text
User or Application
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

Services and daemons run in user space, but they use system calls to request kernel services.

Service management and system initialization are covered in later topics.

## 12. Device Drivers

A **device driver** is software that allows the operating system to communicate with a hardware device.

### Why drivers are needed

Hardware devices have different:

- Communication protocols.
- Register layouts.
- Performance characteristics.
- Error conditions.
- Control interfaces.

Applications should not need separate hardware-specific code for every storage device or network adapter. The driver and kernel provide that abstraction.

### Driver relationship

```text
Application
     |
     v
System Library
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

For a network operation, the path may involve:

```text
Application
     |
     v
Network API
     |
     v
Linux Networking Subsystem
     |
     v
Network Driver
     |
     v
Network Adapter
```

For storage access:

```text
Application
     |
     v
Filesystem Interface
     |
     v
Storage Subsystem
     |
     v
Storage Driver
     |
     v
Storage Device
```

Drivers may run as part of the kernel or be loaded as kernel modules. This lesson introduces the concept only; it does not cover module development or loading procedures.

## 13. How the Layers Work Together

### Complete conceptual flow

```text
User
  |
  v
Application
  |
  v
System Library
  |
  v
System Call
  |
  v
Linux Kernel
  |
  v
Kernel Subsystem
  |
  v
Hardware / Resource
  |
  v
Linux Kernel
  |
  v
Application
  |
  v
User
```

Each layer has a responsibility:

| Layer | Main responsibility |
|---|---|
| User | Requests work or receives results. |
| Application | Performs a useful task. |
| System library | Provides reusable interfaces and prepares requests. |
| System call | Provides a controlled entry point into the kernel. |
| Linux kernel | Manages resources, security, and hardware access. |
| Kernel subsystem | Handles a particular operation such as networking or storage. |
| Device driver | Communicates with a specific device. |
| Hardware | Performs the physical or virtual operation. |

### Example: opening and reading a file

Conceptually:

1. A user asks an application to open a file.
2. The application uses a library interface.
3. The request crosses the system-call boundary.
4. The kernel checks the requested path and access conditions.
5. The filesystem subsystem identifies the file.
6. The storage subsystem determines where the data is located.
7. A storage driver communicates with the device.
8. The kernel receives the data.
9. The result is returned to the application.
10. The application presents the result to the user.

The application does not directly control storage hardware.

### Example: sending network data

Conceptually:

1. An application prepares data for transmission.
2. A networking library prepares the communication request.
3. The request enters the kernel through a system interface.
4. The kernel networking subsystem processes the data.
5. Routing and network policy are evaluated.
6. The network driver prepares the device operation.
7. The network adapter transmits the data.
8. The kernel reports the result to the application.

This same architecture supports applications ranging from web browsers to backend services and monitoring platforms.

## 14. Real-World Infrastructure Example

Consider a Linux application server:

```text
Users
  |
  v
Web Application
  |
  v
Application Libraries
  |
  v
System Calls
  |
  v
Linux Kernel
  |
  +---- Process Management
  +---- Memory Management
  +---- Filesystem Layer
  +---- Networking
  +---- Security
  |
  +---- Storage Device
  +---- Network Adapter
  +---- CPU and RAM
```

When a user submits a request:

1. The network adapter receives packets.
2. The Linux kernel networking subsystem processes them.
3. The application receives the request through user-space interfaces.
4. The application may access a database or file.
5. The kernel coordinates storage and network operations.
6. The application creates a response.
7. The kernel sends the response through the network adapter.

This model helps administrators determine where to investigate problems:

- Slow application logic may be a user-space issue.
- High memory usage may involve the application and memory-management layer.
- A storage delay may involve the filesystem, storage subsystem, driver, or device.
- Lost network packets may involve the application, networking subsystem, driver, adapter, or external network.
- Permission failures may involve user identity, security policy, or application behavior.

## 15. Practical Mental Model

Use the following model:

```text
User
  |
  v
Applications
  |
  v
User Space
  |
  | Controlled requests through system calls
  v
Kernel Space
  |
  v
Hardware and system resources
```

Remember these principles:

1. Users interact with applications and interfaces.
2. Applications run in user space.
3. Libraries simplify access to system functionality.
4. System calls cross the boundary into the kernel.
5. The kernel manages hardware and protected resources.
6. Device drivers translate general kernel operations into device-specific operations.
7. Results return from hardware through the kernel to the application.
8. The kernel is central, but it is not the complete Linux operating system.

## 16. Common Misconceptions

### “Applications talk directly to hardware”

Most applications do not directly control hardware. They use libraries, system interfaces, and system calls. The kernel controls access to protected resources.

### “The shell is the operating system”

A shell is a user-space interface. It allows users to interact with the operating system but is not the kernel or the entire operating system.

### “System libraries and system calls are the same”

They are related but different:

- A system library provides reusable user-space functions.
- A system call is a controlled interface into the kernel.

A library may use one or more system calls internally.

### “User space is unimportant”

User space contains applications, services, libraries, shells, and most software users interact with. The kernel cannot provide a usable system by itself.

### “Kernel space and user space are just folders”

They are conceptual privilege and memory domains, not ordinary filesystem directories.

### “A driver is the same as hardware”

A driver is software that communicates with hardware. The device itself remains separate.

### “Every operation crosses into the kernel”

Some operations can be completed entirely in user space. Operations requiring protected resources, hardware, or kernel-managed objects generally involve system calls.

### “The kernel handles application business logic”

The kernel manages system resources and hardware. Application business logic belongs in user-space applications and services.

## 17. Professional Relevance

### System administration

System administrators use the architecture to identify whether an issue belongs to:

- An application.
- A service.
- A library.
- A process.
- The kernel.
- A driver.
- Hardware.
- Storage.
- Networking.

### System engineering

System engineers design how operating-system components interact with:

- Hardware.
- Virtualization platforms.
- Storage systems.
- Network infrastructure.
- Applications.
- Monitoring and automation systems.

### Network engineering

Network engineers need to understand:

- Network interfaces.
- Drivers.
- Kernel networking.
- Sockets.
- User-space network services.
- Network monitoring systems.
- Network automation platforms.

### Security engineering

Security engineers rely on the separation between user space and kernel space to reason about:

- Privilege boundaries.
- Process isolation.
- Device access.
- Security controls.
- Attack surfaces.
- Monitoring and containment.

### Cloud engineering

Cloud engineers work with Linux architecture inside:

- Virtual machines.
- Container hosts.
- Kubernetes nodes.
- Cloud images.
- Software-defined networks.
- Storage platforms.

### DevOps and SRE

DevOps engineers and SREs use the architecture to connect:

- Application behavior.
- System resource usage.
- Deployment environments.
- Logs.
- Monitoring signals.
- Network performance.
- Storage performance.

### Backend development

Backend developers benefit from understanding how applications use:

- Libraries.
- Processes.
- Memory.
- Filesystems.
- Network sockets.
- System calls.
- Linux services.

## 18. Summary

Linux architecture is a layered relationship between users, applications, user-space software, system interfaces, the kernel, drivers, and hardware.

The main structure is:

```text
Users
  |
Applications
  |
System Libraries
  |
System Calls
  |
Linux Kernel
  |
Device Drivers
  |
Hardware
```

The Linux kernel manages protected resources such as CPU time, memory, storage, networking, and devices.

User-space programs do not normally control hardware directly. They request services through system libraries and system calls. The kernel validates and performs those requests through its subsystems and device drivers.

This separation improves isolation, security, portability, and resource management.

## 19. Knowledge Check

1. What are the major layers of Linux architecture?
2. What is the role of the Linux kernel?
3. What is the difference between user space and kernel space?
4. Why should ordinary applications not directly control hardware?
5. What is a system call?
6. What is the difference between a system library and a system call?
7. What is the purpose of a device driver?
8. Name several major Linux kernel subsystems.
9. What is the relationship between a service, a daemon, and the kernel?
10. Describe the conceptual path followed when an application reads data from storage.
11. Describe the conceptual path followed when an application sends network data.
12. Why is the Linux kernel not the complete operating system?

## 20. Completion Checklist

You should now understand:

- [ ] The major layers of Linux architecture.
- [ ] The role of CPU, RAM, storage, and network devices.
- [ ] The Linux kernel’s position in the architecture.
- [ ] Major kernel responsibilities.
- [ ] The purpose of kernel subsystems.
- [ ] The difference between user space and kernel space.
- [ ] Why privilege separation exists.
- [ ] The meaning of system call.
- [ ] The role of system libraries.
- [ ] The purpose of device drivers.
- [ ] The difference between applications and services.
- [ ] The meaning of a daemon.
- [ ] How applications communicate with the kernel.
- [ ] How the kernel communicates with hardware.
- [ ] How storage and networking fit into the architecture.
- [ ] How this model applies to real Linux servers.
- [ ] Why architecture knowledge helps with troubleshooting and system design.

## 21. Next Topic

**Topic 03 — How Linux Works**
