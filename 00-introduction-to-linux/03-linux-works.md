# Topic 03 - How Linux Works

## 1. Introduction

Linux works as a coordinated system of layers.

When a user asks an application to perform an operation, the application usually does not control the hardware directly. Instead, the request passes through user-space libraries and system interfaces to the Linux kernel. The kernel checks the request, selects the appropriate subsystem, accesses the required resource, and returns a result.

The general model is:

```text
User
  |
Application
  |
System Library
  |
System Call
  |
Linux Kernel
  |
Kernel Subsystem
  |
Hardware / Resource
  |
Linux Kernel
  |
Application
  |
User
```

This topic answers the question:

> What is actually happening inside Linux when I ask the system to do something?

It focuses on operational concepts rather than commands or kernel programming.

## 2. Learning Objectives

After completing this topic, you should be able to:

- Explain how an application interacts with the Linux kernel.
- Describe the role of system libraries.
- Explain what a system call does.
- Describe how Linux creates and manages processes conceptually.
- Explain how the kernel allocates CPU time.
- Describe how applications receive memory.
- Explain how an application accesses data from storage.
- Describe how Linux handles network communication.
- Explain how applications interact with hardware devices.
- Describe the role of permission and security checks.
- Explain how the kernel returns results or errors to applications.
- Trace several common operations from an application to a hardware or system resource.
- Apply this model to a practical Linux server.

## 3. Core Operational Model

The central Linux operating model is:

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

Each layer has a different responsibility.

### Application

An application performs a useful task.

Examples include:

- Web server.
- Database server.
- Text editor.
- Browser.
- Monitoring agent.
- Backup program.
- Python application.

The application decides what it needs, but it normally does not manage hardware directly.

### System library

A system library provides reusable functions that applications can call.

It may provide interfaces for:

- Files.
- Memory.
- Processes.
- Network connections.
- Time.
- Threads.
- Devices.

The library hides some low-level details and prepares requests for the operating system.

### System call

A **system call** is a controlled entry point from user space into the Linux kernel.

The kernel documentation describes system calls as interaction points between user space and the kernel. [docs.kernel](https://docs.kernel.org/process/adding-syscalls.html)

A system call allows an application to request an operating-system service such as:

- Reading data.
- Writing data.
- Creating a process.
- Allocating memory.
- Sending network information.
- Accessing a device.
- Waiting for an event.

### Linux kernel

The kernel receives the request and determines how to handle it.

It may:

- Validate the request.
- Check permissions.
- Check resource limits.
- Select a kernel subsystem.
- Schedule work.
- Communicate with a driver.
- Access cached data.
- Return a result or error.

### Kernel subsystem

A kernel subsystem handles a particular category of work.

Examples include:

- Process management.
- Memory management.
- Filesystem management.
- Networking.
- Device management.
- Security enforcement.

### Hardware or system resource

The requested operation may involve:

- CPU.
- RAM.
- Storage.
- Network interface.
- USB device.
- GPU.
- A resource already held in memory.

Not every request requires immediate physical hardware access. The kernel may satisfy a request from memory, a cache, a virtual device, or another software-managed resource.

## 4. User Space and Kernel Space

### User space

User space is where ordinary applications and services run.

Examples include:

- Web servers.
- Databases.
- Shells.
- System utilities.
- Monitoring agents.
- Graphical applications.
- System libraries.

User-space programs normally have restricted privileges.

### Kernel space

Kernel space is the privileged execution area used by the Linux kernel.

The kernel can:

- Manage memory.
- Schedule CPU work.
- Access hardware through drivers.
- Control filesystems.
- Process network traffic.
- Enforce security boundaries.

### Why the separation exists

The separation protects the system from unrestricted application behavior.

If every application could directly modify hardware or kernel memory:

- One program could corrupt another program.
- Applications could bypass access controls.
- A programming error could crash the entire system.
- Multiple applications could conflict over devices.
- Hardware-specific code would be repeated in many applications.

The separation does not make Linux immune to failures or attacks. Kernel vulnerabilities, unsafe drivers, insecure applications, and configuration mistakes can still affect the whole system.

## 5. System Calls in Operation

A system call creates a controlled transition from user space to kernel space.

Conceptually:

```text
User-Space Application
          |
          v
System Library Function
          |
          v
System Call Entry
          |
          v
Kernel Validates Request
          |
          v
Kernel Performs Operation
          |
          v
Kernel Returns Result
          |
          v
Application Continues
```

### What happens during a system call

1. The application requests an operating-system service.
2. A system library prepares the request.
3. The processor transfers execution to a controlled kernel entry point.
4. The kernel identifies the requested service.
5. The kernel validates the input.
6. The kernel checks permissions and resource limits.
7. The relevant subsystem performs or schedules the operation.
8. The kernel prepares a result or error.
9. Execution returns to user space.
10. The application continues using the result.

The kernel must not blindly trust values supplied by user-space programs. It validates addresses, parameters, identities, and resource requests before performing privileged work.

### System calls are not ordinary library functions

A library function runs in user space. A system call crosses the boundary into kernel space.

A library may:

- Prepare parameters.
- Perform user-space processing.
- Call one or more system calls.
- Convert kernel results into a form convenient for the application.

This means one application-level function does not always correspond to exactly one system call.

## 6. Starting an Application

When a user starts an application, Linux must create a process and prepare it to run.

### Conceptual flow

```text
User requests application
          |
          v
Existing process receives the request
          |
          v
Kernel creates or prepares a process
          |
          v
Program code and required data are mapped
          |
          v
Process receives identity and resources
          |
          v
Scheduler assigns CPU time
          |
          v
Application begins execution
```

### Process creation concept

A **process** is a running instance of a program.

Starting an application may involve:

- Creating a new process.
- Establishing a parent-child relationship.
- Setting the process identity.
- Preparing memory mappings.
- Loading program code.
- Preparing environment information.
- Connecting input and output.
- Assigning initial resource limits.

The exact sequence depends on the application, the user interface, and the software launching the program.

### Parent and child processes

Processes can create other processes.

```text
Parent Process
      |
      +---- Child Process
      |
      +---- Child Process
```

The parent may:

- Wait for a child to finish.
- Receive the child’s exit status.
- Communicate with the child.
- Restart the child if it fails.
- Manage the child’s input and output.

This relationship is important for shells, service managers, application servers, and automation systems.

### Loading program code

The kernel prepares the process’s virtual address space and maps the required program code and libraries.

The application sees a logical address space. The kernel manages the relationship between that virtual address space and physical memory.

## 7. CPU Scheduling

A computer may have many runnable processes but only a limited number of CPU cores.

The kernel’s scheduler decides which process should run on which CPU and for how long.

### Scheduling concept

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

### Why scheduling is required

Without scheduling:

- One process could consume all CPU time.
- Other applications might not run.
- Interactive programs could become unresponsive.
- Background services could be starved.
- System responsiveness would be unpredictable.

### Conceptual scheduling flow

1. A process becomes runnable.
2. The kernel places it in a scheduling structure.
3. The scheduler evaluates runnable processes.
4. A process receives CPU time.
5. The process runs or waits for an event.
6. The scheduler gives CPU time to another runnable process.
7. The process may later run again.

The scheduler considers factors such as priority, fairness, processor availability, and workload behavior.

### CPU waiting and I/O waiting

A process may not always need CPU time. It may wait for:

- Storage data.
- Network data.
- User input.
- Another process.
- A timer.
- A hardware device.

While one process waits, the CPU can run another process.

This is one reason a multitasking operating system can handle many activities at once.

## 8. Memory Allocation

Applications need memory for:

- Program instructions.
- Variables.
- Buffers.
- Libraries.
- Network data.
- Temporary results.
- Runtime structures.

### Conceptual memory flow

```text
Application Requests Memory
          |
          v
System Library
          |
          v
System Call or Kernel Interface
          |
          v
Linux Memory Manager
          |
          v
Virtual Memory Assigned
          |
          v
Physical Memory Used as Needed
```

### Virtual memory

Linux gives processes a controlled virtual address space.

This allows:

- Process isolation.
- Flexible memory allocation.
- Shared libraries.
- Memory mapping.
- Efficient use of physical memory.
- Moving or reclaiming memory when necessary.

Each process generally operates as though it has its own memory environment, while the kernel manages the mapping to physical RAM and other storage-backed resources.

### Memory allocation process

1. An application requests memory.
2. A library prepares the request.
3. The request reaches the kernel or a memory-management interface.
4. The kernel checks limits and available resources.
5. The kernel creates or extends a virtual memory region.
6. Physical memory may be assigned immediately or when the memory is first used.
7. The application receives a usable memory area.

The Linux memory-management subsystem handles memory allocation for both kernel structures and user-space programs. [docs.kernel](https://docs.kernel.org/6.11/admin-guide/mm/index.html)

### Memory protection

The kernel prevents ordinary processes from freely accessing:

- Another process’s memory.
- Kernel memory.
- Protected hardware mappings.
- Restricted system resources.

This protects both system stability and data confidentiality.

## 9. Opening and Reading a File

A file operation demonstrates how several Linux components work together.

### Conceptual flow

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
Permission and Resource Checks
     |
     v
Filesystem Layer
     |
     v
Storage Cache or Storage Driver
     |
     v
Storage Device
```

### Step-by-step process

Suppose an application needs to read configuration data.

1. The application requests access to the file.
2. A library prepares the request.
3. The request enters the kernel.
4. The kernel identifies the process and its credentials.
5. The kernel checks whether access is allowed.
6. The filesystem layer locates the file.
7. The kernel checks whether the data is already in memory.
8. If cached, the kernel may return it without immediate storage access.
9. If not cached, the storage subsystem requests the data.
10. A storage driver communicates with the device.
11. The device returns the data.
12. The kernel places the data in a buffer.
13. The result is returned to the application.

### Why caching matters

Linux can keep frequently used data in memory.

If the requested data is already cached:

```text
Application
     |
     v
Kernel
     |
     v
Memory Cache
     |
     v
Application
```

The storage device may not need to be accessed for every request.

### File descriptors

A process commonly uses a kernel-managed reference to an opened file or resource. This reference allows the process to perform later operations without repeatedly identifying the physical storage location.

At this level, the important idea is that the application works with an operating-system interface, while the kernel manages the underlying resource.

## 10. Sending Network Data

Network communication follows a similar pattern.

Suppose a web application sends a response to a client.

### Conceptual flow

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
Routing and Security Checks
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

### Step-by-step process

1. The application prepares data.
2. A network library prepares a communication request.
3. The request enters the kernel.
4. The kernel identifies the network endpoint.
5. The kernel checks routing and security-related rules.
6. The networking subsystem adds protocol information.
7. The kernel places the data into network buffers.
8. The network driver prepares the device.
9. The network adapter transmits the data.
10. The remote system receives and processes the network traffic.

### Receiving network data

For incoming data:

1. The network adapter receives traffic.
2. The device signals the kernel or provides data for processing.
3. The network driver transfers the data into kernel-managed buffers.
4. The networking subsystem processes the packet.
5. The kernel identifies the destination application.
6. The application receives the data through its network interface.

```text
Network
   |
   v
Network Adapter
   |
   v
Device Driver
   |
   v
Linux Network Stack
   |
   v
Application Socket
   |
   v
Application
```

### Why applications use the kernel network stack

The kernel provides common handling for:

- Network interfaces.
- Protocols.
- Routing.
- Buffers.
- Multiple applications.
- Security controls.
- Virtual network devices.

Applications do not each need to implement complete hardware-specific network communication.

## 11. Accessing a Device

Applications may need to communicate with hardware such as:

- A camera.
- A USB device.
- A storage controller.
- A sensor.
- A GPU.
- A serial interface.

### Device-access flow

```text
Application
     |
     v
User-Space Library or API
     |
     v
System Call or Device Interface
     |
     v
Linux Kernel
     |
     v
Device Subsystem
     |
     v
Device Driver
     |
     v
Hardware Device
```

### What the driver does

The driver translates general operating-system requests into device-specific operations.

For example, an application may request that data be read from a device. The driver knows:

- Which device registers are involved.
- How to start the operation.
- How to detect completion.
- How to handle device-specific errors.
- How to transfer the result to the kernel.

The application does not normally need to know these details.

## 12. Permission and Security Checks

Before performing an operation, the kernel may check whether the requesting process is allowed to perform it.

Checks may involve:

- User identity.
- Group membership.
- File ownership.
- File permissions.
- Process credentials.
- Security policies.
- Resource limits.
- Namespace boundaries.
- Device access rules.
- Network security controls.

### Conceptual security flow

```text
Application Request
        |
        v
Kernel Receives Request
        |
        v
Identity and Permission Check
        |
   +----+----+
   |         |
Allowed    Denied
   |         |
   v         v
Perform    Return Error
Operation
```

### Why checks happen in the kernel

The kernel is the trusted control point for many system resources.

If applications could decide their own access without kernel enforcement, a program could simply ignore restrictions.

Kernel-level checks do not replace application-level security. Applications must also validate users, inputs, sessions, and business rules.

## 13. Kernel Response to Applications

After processing a request, the kernel returns a result.

Possible results include:

- Successful data.
- A numeric return value.
- A status code.
- A process identifier.
- A memory address.
- An indication that the operation is still pending.
- An error.

### Successful operation

```text
Application Request
        |
        v
Kernel Performs Operation
        |
        v
Result Returned
        |
        v
Application Continues
```

### Failed operation

```text
Application Request
        |
        v
Kernel Checks Request
        |
        v
Operation Cannot Be Completed
        |
        v
Error Result Returned
        |
        v
Application Handles the Error
```

The application may respond by:

- Retrying.
- Displaying an error.
- Selecting another resource.
- Logging the failure.
- Terminating.
- Continuing with reduced functionality.

### Asynchronous operations

Some operations do not finish immediately.

A process may:

- Wait for a hardware device.
- Wait for network data.
- Wait for storage.
- Wait for another process.
- Register interest in a future event.

The kernel can place the process in a waiting state and allow other work to run.

## 14. What Happens When a User Starts an Application?

The complete conceptual flow may look like this:

```text
User
  |
  v
User Interface or Shell
  |
  v
Application Launch Request
  |
  v
Existing Process
  |
  v
Process Creation
  |
  v
Memory Setup
  |
  v
Program and Libraries Loaded
  |
  v
Scheduler Assigns CPU Time
  |
  v
Application Runs
  |
  v
Application Requests Kernel Services
  |
  v
Kernel Performs Operations
  |
  v
Results Return to Application
  |
  v
Application Produces Output
  |
  v
User Sees the Result
```

The application may repeat this cycle many times while running.

For example, a web server may repeatedly:

1. Wait for a network request.
2. Receive the request through the kernel.
3. Read configuration or application data.
4. Process the request.
5. Access a database.
6. Send a response.
7. Wait for the next request.

## 15. Real-World Example: Linux Web Server

Consider a Linux server hosting a web application.

```text
Client
  |
  v
Network Interface
  |
  v
Linux Network Stack
  |
  v
Web Server Process
  |
  +---- Application Logic
  |
  +---- Database Connection
  |
  +---- Filesystem Access
  |
  v
Response Returned Through Kernel Networking
```

### Request processing

1. A client sends a request.
2. The network adapter receives the traffic.
3. The Linux driver transfers data to the kernel.
4. The networking subsystem processes the traffic.
5. The web server process receives the request.
6. The application reads configuration or content.
7. The application may request data from a database.
8. The database may access storage.
9. The application creates a response.
10. The web server sends the response through the kernel networking stack.
11. The network adapter transmits the response.

### What Linux contributes

The Linux kernel coordinates:

- CPU time for web and database processes.
- Memory for application data.
- Network traffic.
- Storage access.
- Process isolation.
- Permission checks.
- Device communication.

The web application supplies business logic, while Linux provides the operating-system environment in which that logic runs.

## 16. Practical Mental Model

Use this model whenever you want to understand what Linux is doing:

```text
1. An application requests something.
2. A library prepares the request.
3. A system call enters the kernel.
4. The kernel checks identity, permissions, and resources.
5. The correct kernel subsystem handles the request.
6. A driver may communicate with hardware.
7. The kernel waits, schedules, or completes the operation.
8. A result or error returns to the application.
9. The application produces a result for the user or another system.
```

A shorter version is:

```text
Request
  |
  v
Library
  |
  v
System Call
  |
  v
Kernel Check
  |
  v
Kernel Subsystem
  |
  v
Resource or Hardware
  |
  v
Result
```

The key principle is:

> Applications request services. The kernel controls access. Hardware performs physical operations. Results return through the kernel.

## 17. Common Misconceptions

### “The application directly reads the disk”

Normally, the application requests file access. The kernel, filesystem layer, storage subsystem, and driver handle the details.

### “Every file read requires the storage device”

Not necessarily. Data may already be available in memory or a filesystem cache.

### “The kernel gives every application unlimited memory”

The kernel controls memory allocation and applies process limits, protection, and system-wide resource policies.

### “A process always uses the CPU”

A process may be running, ready to run, sleeping, waiting for I/O, stopped, or terminated.

### “A system call is the same as a normal function call”

A system call crosses from user space into kernel space and requests a privileged operating-system service.

### “The kernel automatically fixes application errors”

The kernel can validate requests and return errors, but application logic, invalid input, memory bugs, and incorrect business behavior must be handled by the application.

### “If a request is slow, the CPU must be overloaded”

Slow operations can be caused by storage latency, network delays, memory pressure, locks, remote services, or application design.

### “The kernel performs all application work”

The kernel manages resources and provides operating-system services. Application logic remains in user space.

### “A library is just a smaller kernel”

A library is user-space software that provides reusable functions. It does not have the same privileges or responsibilities as the kernel.

## 18. Professional Relevance

Understanding how Linux works is essential for diagnosing real systems.

### System administration

System administrators use this mental model to understand:

- Why an application cannot access a file.
- Why a service is waiting.
- Why memory usage increases.
- Why storage access is slow.
- Why a process is not receiving CPU time.
- Why network requests fail.

### System engineering

System engineers use it to design:

- Application tiers.
- Resource boundaries.
- Storage layouts.
- Network paths.
- Service dependencies.
- Performance and availability models.

### Network engineering

Network engineers need to understand:

- How applications use network interfaces.
- How packets move through the kernel.
- Where routing and security checks occur.
- How network delays affect applications.
- How virtual network devices fit into the system.

### Security engineering

Security engineers analyze:

- User-space and kernel-space boundaries.
- Permission checks.
- Process isolation.
- Device access.
- System-call behavior.
- Resource restrictions.
- Security events.

### DevOps and cloud engineering

DevOps and cloud engineers use this model to understand:

- Application deployment.
- Container behavior.
- Resource limits.
- Virtual machines.
- Network paths.
- Storage performance.
- Monitoring data.

### Backend development

Backend developers benefit from knowing:

- How applications access files.
- How network requests reach processes.
- How memory is allocated.
- Why processes wait.
- How operating-system limits affect applications.

## 19. Summary

Linux works through cooperation between user-space programs and the privileged kernel.

The general flow is:

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

The kernel:

- Creates and schedules processes.
- Allocates and protects memory.
- Provides filesystem access.
- Processes network communication.
- Controls device access.
- Checks permissions and resource limits.
- Returns results or errors.

Applications normally do not control hardware directly. They use system libraries and system calls to request operating-system services.

The kernel may complete an operation using memory, caches, software-managed resources, or physical hardware. A process may run, wait, or be rescheduled depending on the operation.

The most useful troubleshooting question is:

> Which layer is handling this request, and where is the operation waiting or failing?

## 20. Knowledge Check

1. What is the general flow from an application to a hardware resource?
2. What role does a system library play?
3. What is a system call?
4. Why does an application normally use a system call instead of controlling hardware directly?
5. What happens when a process needs CPU time?
6. Why might a process wait instead of running?
7. What happens when an application requests memory?
8. How does Linux read data from storage on behalf of an application?
9. How does Linux send network data?
10. What role does a device driver play?
11. What types of checks may the kernel perform before an operation?
12. What can happen when the kernel cannot complete a request?
13. Why might a file request be served from memory instead of storage?
14. How does a Linux web server use kernel services while processing a request?
15. Which Linux layers would you investigate if an application could not reach a database?

## 21. Completion Checklist

You should now understand:

- [ ] How an application requests operating-system services.
- [ ] The role of system libraries.
- [ ] The purpose of system calls.
- [ ] The difference between user space and kernel space.
- [ ] How Linux creates a process conceptually.
- [ ] How the scheduler assigns CPU time.
- [ ] How applications receive memory.
- [ ] How virtual memory protects processes.
- [ ] How an application reads data from storage.
- [ ] How filesystem caches can affect file access.
- [ ] How an application sends network data.
- [ ] How incoming network data reaches an application.
- [ ] The role of device drivers.
- [ ] How the kernel checks permissions and resources.
- [ ] How the kernel returns results and errors.
- [ ] Why processes may wait for resources.
- [ ] How a Linux web server uses kernel services.
- [ ] How this model helps system administrators and engineers troubleshoot systems.
- [ ] Why understanding internal flow is more useful than memorizing isolated commands.

## 22. Next Topic

**Topic 04 - Linux Boot Process**
