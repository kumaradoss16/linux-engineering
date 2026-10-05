# Topic 02 — Linux Architecture

## 1. Introduction

Linux architecture describes how users, applications, system software, the Linux kernel, and hardware interact.

A Linux system is not one single program. It is a collection of layers with different responsibilities:

* **Users** interact with applications.
* **Applications** run in user space.
* **Libraries** provide programming interfaces.
* **System calls** cross the boundary into the kernel.
* The **Linux kernel** manages resources and hardware.
* **Device drivers** connect kernel functions to specific devices.

Understanding these relationships helps explain what happens when a program reads a file, uses memory, sends network data, or communicates with hardware.

This topic remains conceptual. It does not teach Linux commands, kernel programming, or detailed administration.

### Visual color model

The DEVSPIRE website uses semantic colors to reinforce technical relationships:

| Concept                            | Semantic Color | Hex       |
| ---------------------------------- | -------------- | --------- |
| Users / Human interaction          | Orange         | `#F97316` |
| Applications / User-space software | Blue           | `#2563EB` |
| APIs / System calls / Interfaces   | Purple         | `#7C3AED` |
| Linux Kernel / Kernel space        | Navy           | `#0F172A` |
| Hardware / Physical resources      | Slate          | `#475569` |
| Networking                         | Teal           | `#0F766E` |
| Storage / Filesystems              | Amber          | `#B45309` |
| Security / Privilege concerns      | Red            | `#B91C1C` |
| Success / Verification             | Green          | `#15803D` |

These colors are semantic, not decorative.

Color must reinforce labels and structure rather than replace them. All diagrams must remain understandable in grayscale and to users with color-vision differences.

---

## 2. Learning Objectives

After completing this topic, you should be able to:

* Describe the major layers of Linux architecture.
* Distinguish hardware, kernel space, user space, and applications.
* Explain the role of the Linux kernel.
* Identify major kernel subsystems.
* Explain what system calls are.
* Explain how system libraries relate to system calls.
* Describe the role of system services and daemons.
* Explain why applications normally do not access hardware directly.
* Describe the purpose of device drivers.
* Explain the difference between user space and kernel space.
* Describe how an application request travels through a Linux system.
* Connect the architecture to real server and infrastructure environments.

---

# 3. Linux Architecture at a Glance

The following diagram shows the main relationships.

When rendered on the DEVSPIRE website, apply the semantic colors described below:

* **Users:** Orange
* **Applications and user-space software:** Blue
* **System libraries and system calls:** Purple
* **Linux kernel:** Navy
* **Hardware:** Slate
* **Networking components:** Teal
* **Storage/filesystem components:** Amber
* **Security boundaries and security warnings:** Red

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
| Memory Management            |
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

### Semantic rendering

```text
Users
[ORANGE]

    |
    v

Applications / User Space
[BLUE]

    |
    v

Libraries / APIs / System Calls
[PURPLE]

    |
    v

Linux Kernel / Kernel Space
[NAVY]

    |
    v

Hardware
[SLATE]
```

The `[COLOR]` labels above are **implementation guidance only**. Do not display them in the final website diagram.

The diagram is a simplified architectural model. Real Linux systems contain additional components, asynchronous operations, caches, virtual devices, background services, interrupts, and interactions between multiple kernel subsystems.

---

# 4. The Main Architecture Layers

## Users

Users are people or automated systems that interact with a Linux environment.

A user may interact with Linux through:

* A graphical interface.
* A shell.
* A remote administration system.
* An application interface.
* An automated deployment process.

Users do not normally communicate directly with hardware.

They interact with applications or system interfaces that request services from the operating system.

### Visual treatment

Use **Orange `#F97316`** for:

* Users
* Administrators
* Human interaction
* User-facing interfaces

Orange represents the **human interaction layer** and also functions as the primary DEVSPIRE brand accent.

---

## Applications

Applications perform useful work.

Examples include:

* Web servers.
* Database systems.
* Python applications.
* Browsers.
* Monitoring platforms.
* Backup software.
* Development tools.
* Security applications.

An application generally runs in **user space**.

It can request services from the kernel, but it is restricted from directly controlling privileged system resources.

### Visual treatment

Use **Blue `#2563EB`** for:

* Applications
* User-space programs
* Services
* Daemons
* User-space development tools

Blue represents **software operating outside the kernel**.

---

## Shells and User Interfaces

A **shell** is a user interface and command interpreter that allows a user or script to interact with the operating system.

A graphical desktop, terminal emulator, remote management interface, or application API may also provide ways to interact with Linux.

The shell belongs to user space.

It does not replace the kernel. Instead, it requests operating-system services through user-space libraries and system calls.

Detailed shell usage belongs to a later topic.

### Visual treatment

Shells and user interfaces use the **Blue user-space color**.

---

## System Libraries and APIs

A **system library** is reusable software that provides functions for applications and other user-space programs.

Libraries can:

* Provide common programming functions.
* Hide low-level implementation details.
* Convert application requests into system-call requests.
* Offer consistent interfaces across applications.
* Reduce the need for every application to implement the same logic.

An **API**, or Application Programming Interface, is a defined way for one software component to request functionality from another.

Libraries and APIs are not the same as the kernel.

They are user-space components that help applications communicate with the operating system.

### Visual treatment

Use **Purple `#7C3AED`** for:

* System libraries
* APIs
* System interfaces
* System calls
* User/kernel interaction boundaries

Purple represents **interfaces and abstraction boundaries**.

---

## System Calls

A **system call** is a controlled entry point through which a user-space application requests a service from the kernel.

Typical requests include:

* Creating or managing a process.
* Allocating memory.
* Accessing a file.
* Communicating over a network.
* Accessing a device.
* Waiting for an event.
* Checking system information.

The Linux kernel documentation describes system calls as interaction points between user space and the kernel.

A system call is a boundary, not just an ordinary library function.

When a program makes a system call, the processor transitions from a restricted user-mode context into a privileged kernel-mode context.

### Visual treatment

Use **Purple `#7C3AED`** for the system-call boundary.

For example:

```text
+---------------------------+
|       USER SPACE          |
|                           |
|      Application          |
+---------------------------+
             |
             |
       SYSTEM CALL
        [PURPLE]
             |
             v
+---------------------------+
|       KERNEL SPACE        |
|                           |
|      Linux Kernel         |
+---------------------------+
```

Purple should visually communicate:

> Controlled interface between user space and kernel space.

---

## Linux Kernel

The Linux kernel is the privileged core of the system.

It manages:

* CPU scheduling.
* Processes and threads.
* Memory.
* Filesystems.
* Storage access.
* Networking.
* Device drivers.
* Security mechanisms.
* Inter-process communication.
* System resources.

The kernel does not normally provide the complete user-facing environment.

It provides the core mechanisms that user-space software uses.

### Visual treatment

Use **Navy `#0F172A`** for:

* Linux kernel
* Kernel space
* Kernel subsystems
* Privileged operating-system functionality

Navy represents the **privileged system-management layer**.

---

## Hardware

Hardware provides the physical or virtual resources managed by Linux.

Examples include:

* CPU.
* RAM.
* Storage devices.
* Network adapters.
* GPUs.
* USB devices.
* Hardware controllers.
* Virtual disks.
* Virtual network adapters.
* Virtual CPUs.

Linux may run on:

* Physical hardware.
* A virtual machine.
* A cloud instance.
* An embedded device.

In each case, the kernel manages the resources exposed to it.

### Visual treatment

Use **Slate `#475569`** for hardware.

Slate represents:

* Physical resources.
* Virtual hardware.
* Devices.
* Controllers.

Hardware should remain visually neutral so that the software architecture remains the primary focus.

---

# 5. User Space and Kernel Space

## User Space

**User space** is the environment where ordinary applications and services run.

User-space components include:

* Applications.
* System libraries.
* Shells.
* Daemons.
* Background services.
* Development tools.
* User interfaces.

User-space programs run with restricted privileges.

They cannot normally read or modify arbitrary kernel memory or directly control hardware.

### Color

Use **Blue `#2563EB`**.

---

## Kernel Space

**Kernel space** is the privileged environment used by the Linux kernel.

Kernel-space functions include:

* Process management.
* Memory management.
* Filesystem support.
* Networking.
* Device drivers.
* Security enforcement.
* Hardware interaction.

Kernel-space code has access to protected system resources.

A severe kernel-level failure can affect the entire operating system.

### Color

Use **Navy `#0F172A`**.

---

## Why the Separation Exists

The separation between user space and kernel space provides:

* Fault isolation.
* Security boundaries.
* Controlled hardware access.
* Process isolation.
* Resource management.
* More predictable system behavior.

Without this separation, an ordinary application could potentially overwrite kernel memory, interfere with other applications, or control hardware without permission.

### Security color rule

Use **Red `#B91C1C`** only when discussing:

* Security violations.
* Privilege escalation.
* Denied access.
* Security risks.
* Boundary failures.

Do not color the entire kernel red.

The kernel itself remains **Navy**.

---

## Conceptual Privilege Model

```text
+----------------------------------+
|          USER SPACE              |
|                                  |
| Applications                     |
| Services                         |
| Daemons                          |
| Shells                           |
| System Libraries                 |
+----------------------------------+
                 |
                 | System Calls
                 |
                 v
+----------------------------------+
|         KERNEL SPACE             |
|                                  |
| Process Management               |
| Memory Management                |
| Filesystems                      |
| Networking                       |
| Device Drivers                   |
| Security                         |
+----------------------------------+
                 |
                 v
+----------------------------------+
|            HARDWARE              |
|                                  |
| CPU, RAM, Storage, Network       |
| Devices and Controllers          |
+----------------------------------+
```

### Rendering rules

* User Space: **Blue**
* System-call boundary: **Purple**
* Kernel Space: **Navy**
* Hardware: **Slate**
* Security violation or denied-access callout: **Red**

The labels and layer boundaries must explain the architecture even when color is unavailable.

---

# 6. System Calls and Interfaces

## What Problem Do System Calls Solve?

Applications need operating-system services, but allowing applications to directly control hardware would create security and stability problems.

System calls provide a controlled interface:

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
Hardware or System Resource
```

### Semantic colors

* Application: Blue
* Library: Purple
* System Call: Purple
* Kernel: Navy
* Hardware: Slate

The application makes a request.

The kernel:

1. Identifies the requested service.
2. Checks whether the request is valid.
3. Checks applicable permissions and restrictions.
4. Uses the appropriate kernel subsystem.
5. Interacts with hardware or an internal resource.
6. Returns a result or error.

---

## System Calls Versus Libraries

A library function runs in user space unless it needs to request a kernel service.

For example:

```text
Application
     |
     v
Library Function
     |
     | May invoke a system call
     v
Linux Kernel
```

The application usually calls a library interface instead of manually constructing a low-level kernel request.

This makes application development easier and provides a consistent interface across many programs.

---

## System Calls Are Controlled Boundaries

A system call is not unrestricted access to the kernel.

The kernel can:

* Validate arguments.
* Check user identity.
* Check permissions.
* Check resource limits.
* Reject invalid requests.
* Return an error.
* Isolate the operation from unrelated processes.

The system-call boundary therefore has both a technical and security purpose.

### Security visual rule

Normal system-call boundaries use **Purple**.

Use **Red** only when explaining:

* A denied request.
* A privilege violation.
* A security vulnerability.
* Unauthorized access.

---

# 7. Major Linux Kernel Subsystems

The Linux kernel is logically divided into subsystems.

These subsystems cooperate rather than operating as completely independent programs.

---

## Process Management

The process-management subsystem:

* Creates and terminates processes.
* Schedules work on CPUs.
* Tracks process state.
* Coordinates threads.
* Supports process communication.
* Manages process relationships.

A web server and a database server may both be running at the same time.

Process management controls how they share CPU resources.

### Color

Use **Navy** because process management is part of the kernel.

---

## Memory Management

The memory-management subsystem:

* Allocates memory.
* Releases memory.
* Provides virtual memory.
* Protects process address spaces.
* Manages memory used by the kernel.
* Coordinates physical RAM and storage-backed memory.

The kernel gives processes controlled memory views rather than allowing them to access arbitrary physical memory.

### Color

Use **Navy** for the kernel subsystem.

Use **Slate** when specifically representing physical RAM.

---

## Filesystem Layer

The filesystem subsystem provides a common interface for applications to access data.

It coordinates:

* Files and directories.
* Filesystem operations.
* Storage caching.
* File metadata.
* Storage-device access.
* Filesystem-specific implementations.

Applications can use a common interface even when the underlying storage technology differs.

### Color

Use **Amber `#B45309`** for:

* Filesystems.
* Storage.
* Disks.
* Block storage.
* Storage paths.

The filesystem remains a kernel subsystem, so its architectural container may remain Navy while storage-specific elements use Amber.

---

## Networking Subsystem

The networking subsystem handles:

* Network protocols.
* Network interfaces.
* Packets.
* Routing.
* Sockets.
* Network namespaces.
* Virtual network devices.

Network-related architecture:

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
     |
     v
Network
```

### Color

Use **Teal `#0F766E`** for network-related components.

The Linux Networking Subsystem remains conceptually part of the **Navy kernel layer**, but its network-specific elements may use Teal as a semantic accent.

---

## Device Drivers

A **device driver** is software that allows the kernel to communicate with a particular device.

Examples include drivers for:

* Network adapters.
* Storage controllers.
* USB devices.
* Graphics hardware.
* Audio devices.
* Sensors.

The driver translates general kernel requests into device-specific operations.

```text
Kernel Subsystem
       |
       v
Device Driver
       |
       v
Hardware Controller
       |
       v
Physical Device
```

### Color

* Kernel subsystem: Navy
* Device driver: Navy
* Hardware controller: Slate
* Network-specific driver path: Teal accent
* Storage-specific driver path: Amber accent

The driver itself should not be confused with the hardware it controls.

---

## Security Mechanisms

The kernel helps enforce:

* User and group identities.
* Access boundaries.
* Process isolation.
* Resource restrictions.
* Security policies.
* Network controls.

Security is distributed across the system.

The kernel is important, but secure operation also requires:

* Secure applications.
* Correct configuration.
* Security updates.
* Monitoring.
* Appropriate administrative controls.

### Color

Use **Red `#B91C1C`** only for:

* Security boundaries.
* Security failures.
* Privilege violations.
* Access-denied conditions.
* Security warnings.

Do not make all security-related architecture red. Security mechanisms operating normally may remain Navy.

---

## Inter-Process Communication

Processes sometimes need to exchange information or coordinate activities.

The kernel supports mechanisms for:

* Process synchronization.
* Data exchange.
* Event notification.
* Shared resource coordination.

For example, a web service may communicate with a database service through network sockets or other operating-system-supported interfaces.

### Color

Keep the kernel-side mechanism Navy.

Use Blue for user-space processes.

Use Teal when the communication is specifically network-based.

---

# 8. System Services and Daemons

A **system service** is a software component that provides a function to other applications, users, or systems.

A **daemon** is a background process that usually performs a service without requiring continuous direct interaction from a user.

Examples include services that provide:

* Web hosting.
* DNS.
* Logging.
* Monitoring.
* Time synchronization.
* Remote access.
* Database operation.
* Network management.

The relationship can be represented as:

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

### Color mapping

* User or Application: Blue/Orange depending on context
* System Service: Blue
* Daemon Process: Blue
* Linux Kernel: Navy
* Network: Teal
* Storage: Amber
* Hardware: Slate

Services and daemons normally run in user space.

They use system calls to request kernel functions.

Service management belongs to a later topic.

---

# 9. How an Application Communicates with Hardware

Applications normally do not access hardware directly.

Consider an application that needs to read data from storage:

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
Filesystem Layer
     |
     v
Storage Subsystem
     |
     v
Device Driver
     |
     v
Storage Device
```

### Semantic color path

```text
Application
[BLUE]

     |

System Library / System Call
[PURPLE]

     |

Filesystem / Storage
[AMBER]

     |

Device Driver
[NAVY]

     |

Storage Device
[SLATE]
```

The `[COLOR]` labels are implementation guidance and should not appear in the final rendered diagram.

### Step-by-step flow

1. The application requests data.
2. A system library provides an interface for the request.
3. The application crosses the user-space boundary through a system call.
4. The kernel validates the request.
5. The filesystem layer identifies the required file and storage operation.
6. The storage subsystem coordinates access to the device.
7. The device driver communicates with the storage controller.
8. The hardware returns data.
9. The kernel processes the result.
10. The result returns to the application through the system-call interface.

The application sees a logical file or data resource.

It does not need to understand the physical storage protocol.

---

# 10. Architecture by Resource Type

## CPU and Memory

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
     +---- Process Management
     |
     +---- CPU Scheduler
     |
     +---- Memory Management
     |
     v
CPU / RAM
```

### Colors

* Application: Blue
* System interface: Purple
* Kernel: Navy
* CPU/RAM: Slate

The kernel decides when a process receives CPU time and which memory resources it can use.

---

## Storage and Filesystems

```text
Application
     |
     v
System Call
     |
     v
Filesystem Layer
     |
     v
Storage Subsystem
     |
     v
Device Driver
     |
     v
Storage Device
```

### Colors

* Application: Blue
* System Call: Purple
* Filesystem: Amber
* Storage subsystem: Amber/Navy
* Driver: Navy
* Storage device: Slate

---

## Networking

```text
Application
     |
     v
Network API
     |
     v
System Call
     |
     v
Linux Networking Subsystem
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

### Colors

* Application: Blue
* Network API/System Call: Purple
* Networking subsystem: Teal/Navy
* Network driver: Teal/Navy
* Network adapter: Teal
* External network: Teal

---

## Security Boundary

```text
User-Space Application
          |
          |
          | Controlled Request
          |
          v
     System Call Boundary
          |
          v
      Kernel Space
```

### Colors

* User-space application: Blue
* System-call boundary: Purple
* Kernel space: Navy
* Security violation/warning: Red

Purple represents the normal controlled boundary.

Red should only appear when discussing a security problem involving that boundary.

---

# 11. Architecture and Virtual Machines

Linux can run directly on physical hardware or inside a virtual machine.

## Physical System

```text
Linux Applications
        |
Linux User Space
        |
Linux Kernel
        |
Physical Hardware
```

### Colors

* Applications: Blue
* User space: Blue
* Kernel: Navy
* Hardware: Slate

---

## Virtual Machine

```text
Linux Applications
        |
Linux User Space
        |
Linux Kernel
        |
Virtual Hardware
        |
Hypervisor
        |
Physical Hardware
```

### Colors

* Applications: Blue
* User space: Blue
* Linux kernel: Navy
* Virtual hardware: Slate
* Hypervisor: Neutral dark/slate
* Physical hardware: Slate

From the guest Linux kernel's perspective, it may interact with:

* Virtual CPUs.
* Virtual memory.
* Virtual disks.
* Virtual network adapters.

The physical hardware is managed by the host operating system and hypervisor.

The guest kernel still performs its normal operating-system responsibilities within the resources presented to it.

---

# 12. Real-World Example: Linux Web Server

Consider a Linux web server receiving a request from a client.

```text
Client
   |
   v
Network Adapter
   |
   v
Network Driver
   |
   v
Linux Networking Subsystem
   |
   v
Web Server Process
   |
   v
Filesystem / Application Resources
```

### Color mapping

* Client: Orange or Blue depending on whether it represents a user or application.
* Network Adapter: Teal.
* Network Driver: Teal/Navy.
* Linux Networking Subsystem: Teal/Navy.
* Web Server Process: Blue.
* Filesystem: Amber.
* Kernel boundaries: Navy.
* System-call interfaces: Purple.

### What happens conceptually

1. The network adapter receives data.
2. The device driver transfers the data into kernel-managed structures.
3. The Linux networking subsystem processes the network traffic.
4. The web server process receives the request through a user-space interface.
5. The process may request data from storage or another application service.
6. The kernel schedules CPU time for the web server.
7. The kernel manages the web server's memory.
8. The response travels back through the networking subsystem.
9. The network driver sends the response through the network adapter.

The web server application handles HTTP-level logic.

The kernel handles CPU scheduling, memory, networking, storage access, and device communication.

---

# 13. Real-World Enterprise Perspective

A typical enterprise environment may look like this:

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
Storage
   |
   v
Backup Infrastructure
```

### Suggested semantic colors

* Internet: Teal
* Firewall/security boundary: Red accent
* Load balancer: Teal
* Linux web servers: Blue/Navy
* Application servers: Blue/Navy
* Database servers: Blue/Navy
* Storage: Amber
* Backup storage: Amber
* Hardware: Slate

Do not make the entire architecture red merely because security is involved.

Red should identify the **security function or security boundary**, not the entire infrastructure.

### Linux architecture appears at several layers

* The firewall may run Linux or a specialized network operating system.
* Web servers run applications in user space.
* The kernel manages network traffic, CPU, memory, storage, and processes.
* Application servers use system libraries and kernel interfaces.
* Database servers rely on kernel storage, memory, and networking subsystems.
* Monitoring and backup systems run as user-space services.
* Storage systems interact with kernel storage and device layers.

---

# 14. Engineering Roles

## System Administrator

The system administrator works with:

* Operating-system configuration.
* Users and access.
* Services.
* Storage.
* Logs.
* Updates.
* Performance.
* Troubleshooting.

Primary semantic areas:

**Blue + Navy + Amber**

---

## System Engineer

The system engineer designs:

* Server architectures.
* Operating-system standards.
* Virtualization platforms.
* Hardware and software integration.
* Availability and capacity plans.

Primary semantic areas:

**Navy + Slate + Blue**

---

## Network Engineer

The network engineer works with:

* Network interfaces.
* Routing.
* Firewalls.
* DNS.
* VPN systems.
* Monitoring.
* Network automation.

Primary semantic color:

**Teal**

---

## Security Engineer

The security engineer evaluates:

* Privilege boundaries.
* Access controls.
* Kernel and user-space exposure.
* System isolation.
* Logs and security events.
* Host and network protections.

Primary semantic color:

**Red for security concerns**, with Navy and Blue for the underlying architecture.

---

## DevOps Engineer

The DevOps engineer uses the architecture to design:

* Automated deployments.
* Configuration management.
* Container platforms.
* Monitoring.
* Infrastructure as Code.
* CI/CD environments.

Primary semantic areas:

**Blue + Navy + Teal**

---

## Cloud Engineer

The cloud engineer works with:

* Virtual CPUs.
* Virtual memory.
* Virtual disks.
* Virtual network interfaces.
* Linux images.
* Cloud services.
* Container hosts.

Primary semantic areas:

**Slate + Navy + Teal**

---

## Site Reliability Engineer

The SRE uses architecture knowledge to investigate:

* Resource exhaustion.
* Process failures.
* Storage delays.
* Network failures.
* Service latency.
* System availability.

Use the relevant semantic color depending on the failure domain.

---

## Backend Developer

The backend developer needs to understand:

* How applications use libraries.
* How applications access files.
* How network communication reaches the kernel.
* How memory and processes affect application behavior.
* How production environments differ from development environments.

Primary semantic areas:

**Blue + Purple + Navy**

---

# 15. Common Misconceptions

## "Applications run inside the kernel"

Most applications run in user space.

They request kernel services through system calls.

**Visual rule:** Application = Blue, Kernel = Navy.

---

## "The kernel is the same as the shell"

The shell is a user-space interface.

The kernel manages resources and hardware.

**Visual rule:** Shell = Blue, Kernel = Navy.

---

## "Libraries are part of the kernel"

System libraries normally run in user space.

They provide interfaces that may invoke system calls.

**Visual rule:** Libraries = Purple/Blue, Kernel = Navy.

---

## "System calls are ordinary application functions"

System calls are controlled interfaces to kernel services.

They cross from user space into kernel space.

**Visual rule:** Boundary = Purple.

---

## "Applications can directly access hardware"

Ordinary applications normally use kernel services and device drivers instead of controlling hardware directly.

**Visual rule:** Application = Blue, Driver = Navy, Hardware = Slate.

---

## "Every device driver is a separate operating system"

A device driver is a software component that allows the kernel to communicate with a device.

It is part of the broader operating-system architecture.

---

## "User space is unimportant"

User space contains:

* Applications.
* Libraries.
* Services.
* Daemons.
* Shells.

A Linux system requires both user space and kernel space.

---

## "Kernel space and user space are separate computers"

They are privilege domains within the same operating system and memory architecture.

They are not necessarily separate physical machines.

---

## "The kernel handles application business logic"

The kernel provides general operating-system services.

Application logic belongs to user-space software.

---

# 16. Practical Mental Model

Use this simplified model:

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
  +---- Security
  |
  v
Hardware / System Resource
```

### Semantic model

```text
User
  [ORANGE]
     |
     v
Application
  [BLUE]
     |
     v
System Library / System Call
  [PURPLE]
     |
     v
Linux Kernel
  [NAVY]
     |
     +---- Process Management
     +---- Memory Management
     +---- Filesystem [AMBER]
     +---- Networking [TEAL]
     +---- Device Drivers
     +---- Security [RED when security concern exists]
     |
     v
Hardware
  [SLATE]
```

The color labels above are implementation guidance only.

### When analyzing a Linux system, ask:

1. Which application or service initiated the request?
2. Which library or API is involved?
3. Does the request require a system call?
4. Which kernel subsystem handles it?
5. Is a device driver involved?
6. Which hardware or system resource is used?
7. What result or error returns to the application?

This mental model is useful for troubleshooting because it prevents treating Linux as one undivided component.

---

# 17. Summary

Linux architecture is based on cooperation between several layers:

* Users interact with applications and interfaces.
* Applications and services run primarily in user space.
* System libraries provide reusable interfaces.
* System calls provide controlled entry points into the kernel.
* The Linux kernel manages processes, memory, filesystems, networking, devices, security, and resources.
* Device drivers connect kernel subsystems to specific hardware.
* Hardware provides the physical or virtual resources used by the system.

The distinction between user space and kernel space is central to Linux architecture.

User-space software performs application and service work with restricted privileges.

Kernel-space software performs privileged operations and manages hardware.

The DEVSPIRE semantic color system reinforces these relationships:

| Layer / Concept          | Color  |
| ------------------------ | ------ |
| User                     | Orange |
| Application / User Space | Blue   |
| API / System Call        | Purple |
| Kernel / Kernel Space    | Navy   |
| Hardware                 | Slate  |
| Networking               | Teal   |
| Storage / Filesystem     | Amber  |
| Security Concern         | Red    |

Understanding this architecture helps explain real systems.

A web server, database server, cloud virtual machine, network appliance, or container host depends on the same basic relationship between applications, system interfaces, the kernel, and hardware.

---

# 18. Knowledge Check

1. What are the major layers of Linux architecture?
2. What is the difference between user space and kernel space?
3. Why do applications normally use system calls instead of accessing hardware directly?
4. What is the role of a system library?
5. What is a system call?
6. What responsibilities belong to the Linux kernel?
7. What is the role of a device driver?
8. How does a filesystem request reach a storage device?
9. How does a network application communicate with a network adapter?
10. Why is the user-space and kernel-space boundary important for security?
11. What is the difference between a service and the kernel?
12. How does Linux architecture appear in a web-server environment?
13. Why is color not sufficient by itself to explain an architecture diagram?
14. What semantic color represents the Linux kernel?
15. What semantic color represents networking?
16. What semantic color represents storage?
17. What semantic color represents the system-call boundary?

---

# 19. Completion Checklist

You should now understand:

* The major layers of Linux architecture.
* The role of users.
* The role of applications.
* The purpose of shells and user interfaces.
* The role of system libraries and APIs.
* The meaning of a system call.
* The role of the Linux kernel.
* The difference between user space and kernel space.
* The purpose of process management.
* The purpose of memory management.
* The purpose of filesystem support.
* The purpose of networking support.
* The purpose of device drivers.
* The role of kernel security mechanisms.
* Why applications normally do not directly control hardware.
* How storage requests move through Linux.
* How network requests move through Linux.
* How Linux architecture applies to virtual machines and enterprise servers.
* How system administrators, network engineers, security engineers, DevOps engineers, cloud engineers, SREs, and backend developers interact with Linux architecture.
* The semantic color system used throughout the DEVSPIRE Linux learning series.
* Why colors must reinforce, rather than replace, technical labels.

---

# 20. Visual Accessibility Requirements

The website implementation must satisfy these rules:

* Do not communicate technical meaning through color alone.
* Every colored architecture component must also have a text label.
* Maintain sufficient contrast between foreground and background.
* Do not use bright orange text on white backgrounds for long paragraphs.
* Do not use red for normal content.
* Do not use blue, purple, teal, amber, or red randomly.
* Maintain the same semantic meaning across all Linux tutorials.
* Diagrams must remain understandable in grayscale.
* Dark mode must preserve the semantic meaning of each color.
* Do not use gradients.
* Do not use neon colors.
* Do not use glowing effects.
* Do not use hacker-style green terminal aesthetics.
* Do not turn technical diagrams into decorative illustrations.

The visual style should resemble **professional engineering documentation** rather than a gaming interface, hacker interface, or generic AI/SaaS dashboard.

---

# 21. Next Topic

**Topic 03 — How Linux Works**
