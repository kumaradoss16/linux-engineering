# Topic 04 — Linux Boot Process

## 1. Introduction

The Linux boot process is the sequence of events that changes a powered-off computer into a usable operating-system environment.

It begins when the machine receives power and ends when:

- The Linux kernel is running.
- Hardware has been initialized.
- The root filesystem is available.
- The initial system process is running.
- System services have started.
- A login environment or user session is available.

The boot process involves several distinct components:

- Hardware.
- Firmware.
- BIOS or UEFI.
- Boot device selection.
- Bootloader.
- Linux kernel.
- Initial RAM filesystem, or initramfs.
- Root filesystem.
- Init system.
- System services.
- Login environment.
- User applications.

These components are related, but they are not interchangeable. Firmware is not the bootloader, the bootloader is not the kernel, and the kernel is not the init system.

## 2. Learning Objectives

After completing this topic, you should be able to:

- Describe the Linux boot process from power-on to a user session.
- Explain the role of firmware.
- Distinguish BIOS from UEFI at a conceptual level.
- Explain POST and hardware initialization.
- Describe how the system selects a boot device.
- Explain the purpose of a bootloader.
- Identify GRUB as a common Linux bootloader.
- Explain what a Linux kernel image is.
- Describe the purpose of initramfs.
- Explain how the kernel initializes hardware and core subsystems.
- Explain the role of the root filesystem.
- Describe PID 1 and the init system.
- Explain how system services start.
- Distinguish the system boot process from a user login session.
- Apply the boot sequence to a Linux server.
- Identify where boot failures may occur conceptually.

## 3. Complete Boot Sequence

The conceptual Linux boot sequence is:

```text
Power On
   |
   v
BIOS / UEFI
   |
   v
POST
   |
   v
Boot Device Selection
   |
   v
Bootloader
   |
   v
Linux Kernel
   |
   v
initramfs
   |
   v
Kernel Initialization
   |
   v
Root Filesystem
   |
   v
PID 1 / Init System
   |
   v
System Services
   |
   v
Login / User Session
   |
   v
User Applications
```

Each stage prepares the resources required by the next stage.

## 4. Power On and Firmware

### Power on

When a computer receives power:

1. The processor begins execution from a predefined location.
2. Firmware stored on the motherboard starts.
3. The firmware initializes enough hardware to continue startup.
4. The system performs initial hardware checks.
5. The firmware identifies a device or operating-system entry to boot.

At this stage, Linux has not started yet.

### What is firmware?

**Firmware** is software stored in non-volatile memory associated with hardware.

It operates before the operating system and provides low-level initialization and startup functions.

Firmware may:

- Initialize the CPU.
- Initialize memory.
- Detect storage devices.
- Detect input devices.
- Configure hardware settings.
- Select a boot entry.
- Start a bootloader or operating-system component.

Firmware is not part of the Linux operating system.

## 5. BIOS and UEFI

### BIOS

**BIOS**, or Basic Input/Output System, is an older firmware interface used on many traditional systems.

At a high level, BIOS:

- Performs early hardware initialization.
- Runs POST.
- Selects a boot device according to its configuration.
- Loads initial bootloader code from that device.
- Transfers control to the bootloader.

Traditional BIOS systems commonly use the Master Boot Record, or MBR, as part of the early boot process.

### UEFI

**UEFI**, or Unified Extensible Firmware Interface, is a newer firmware interface.

UEFI can:

- Store boot entries in non-volatile firmware memory.
- Read supported filesystem structures.
- Load executable boot applications.
- Use an EFI System Partition.
- Support modern hardware and boot configurations.
- Provide a firmware configuration environment.

UEFI does not depend on the traditional BIOS MBR boot method in the same way. It normally loads a bootloader application from the EFI System Partition.

### BIOS and UEFI comparison

| Feature | BIOS | UEFI |
|---|---|---|
| Generation | Older firmware interface. | Modern firmware interface. |
| Boot method | Commonly loads early boot code from an MBR. | Loads an EFI application using firmware boot entries. |
| Partition relationship | Traditionally associated with MBR partitioning. | Commonly used with GPT partitioning. |
| Boot environment | More limited. | More capable and extensible. |
| Linux role | Initializes hardware and starts boot code. | Initializes hardware and starts a bootloader or kernel entry. |

The exact boot path depends on the system firmware, configuration, storage layout, and operating-system installation.

## 6. POST and Hardware Initialization

**POST**, or Power-On Self-Test, is an early hardware-checking stage.

The firmware may check or initialize:

- CPU.
- RAM.
- Storage controllers.
- Keyboard.
- Display.
- USB devices.
- Network hardware.
- Other essential components.

POST is not a complete hardware test. It verifies enough of the system for the boot process to continue.

If a serious problem is detected, the system may:

- Display a firmware error.
- Produce diagnostic sounds.
- Stop before loading the bootloader.
- Enter a firmware setup or recovery mode.

### Important distinction

```text
Firmware and POST
    |
    | Hardware initialization before Linux
    v
Bootloader
    |
    | Loads Linux components
    v
Linux Kernel
```

The Linux kernel does not perform the first stage of power-on hardware initialization. Firmware starts that work.

## 7. Boot Device Selection

After initial hardware initialization, the firmware selects a boot source.

Possible boot sources include:

- Internal SSD.
- Hard disk.
- NVMe device.
- USB device.
- Optical media.
- Network boot server.
- Virtual disk in a virtual machine.

The firmware uses a configured boot order or boot entry.

### Conceptual flow

```text
Firmware
    |
    v
Configured Boot Entries
    |
    v
Selected Boot Device
    |
    v
Bootloader or Operating-System Entry
```

In a cloud virtual machine, the physical firmware may be hidden from the user. The cloud platform provides virtual hardware and a configured boot environment, but the logical sequence still includes boot firmware or a firmware-like startup layer.

## 8. Bootloader

A **bootloader** is software that loads the Linux kernel and related boot files into memory and transfers control to the kernel.

The bootloader operates after firmware and before the Linux kernel.

### Bootloader responsibilities

A bootloader may:

- Display boot choices.
- Select a kernel version.
- Load the kernel image.
- Load the initramfs image.
- Pass configuration parameters to the kernel.
- Select a root filesystem.
- Support multiple operating systems.
- Provide recovery or alternate boot entries.

### GRUB

**GRUB**, commonly expanded as Grand Unified Bootloader, is a widely used Linux bootloader.

GRUB can:

- Display a boot menu.
- Select among installed kernels.
- Load a kernel and initramfs.
- Pass boot parameters.
- Support multiple operating systems.
- Start a selected boot entry.

Other bootloaders and firmware-based boot managers are also used. GRUB is a common example, not a required component of every Linux system.

### Bootloader and kernel relationship

```text
Firmware
    |
    v
Bootloader
    |
    +---- Linux Kernel Image
    |
    +---- initramfs Image
    |
    v
Linux Kernel Starts
```

The bootloader does not become the operating system. Its primary role is to prepare the kernel and transfer control to it.

## 9. Linux Kernel Image

A **kernel image** is a bootable representation of the Linux kernel.

The bootloader loads the kernel image into memory and provides the information required for startup.

The kernel image may include or be accompanied by:

- Kernel code.
- Kernel data.
- Hardware-support components.
- Configuration information.
- Boot parameters.
- An initramfs image.

### Kernel handoff

The general sequence is:

1. The bootloader locates a kernel image.
2. The bootloader loads it into memory.
3. The bootloader loads an initramfs image if required.
4. The bootloader passes boot parameters.
5. The bootloader transfers control to the kernel.
6. The kernel begins initialization.

After the handoff, the Linux kernel becomes responsible for continuing the operating-system startup process.

## 10. initramfs

### What is initramfs?

**initramfs**, or initial RAM filesystem, is a temporary early-user-space environment used during boot.

It is loaded into memory before the main root filesystem is available.

The Linux kernel documentation describes initramfs as an archive that is unpacked into an in-memory root filesystem during boot. Early user space provides programs and libraries needed while the kernel is coming up. [docs.kernel](https://docs.kernel.org/driver-api/early-userspace/early_userspace_support.html)

### Why initramfs is needed

The kernel may need additional software or drivers before it can access the real root filesystem.

For example, the root filesystem may be located on:

- An encrypted device.
- A software RAID array.
- A logical volume.
- A storage controller requiring a driver.
- A network device.
- A complex storage arrangement.

The initramfs provides the temporary environment needed to prepare access to that root filesystem.

### What initramfs may contain

An initramfs can contain:

- Essential programs.
- Libraries.
- Kernel modules.
- Storage drivers.
- Filesystem support.
- Encryption support.
- RAID or volume-management support.
- Configuration files.
- An early initialization program.

It is not normally the permanent root filesystem. It exists to help the system reach the permanent root filesystem.

### Conceptual relationship

```text
Bootloader
    |
    +---- Linux Kernel
    |
    +---- initramfs
              |
              v
       Prepare access to
       the real root filesystem
```

### Early user space

The term **early user space** refers to programs that run in user mode before the normal root filesystem and complete user-space environment are available.

This allows some startup work to occur outside the kernel while still happening early in the boot process.

## 11. Kernel Initialization

After the bootloader transfers control, the kernel begins its own initialization.

### Kernel startup responsibilities

The kernel initializes:

- CPU-related structures.
- Memory management.
- Interrupt handling.
- Timers.
- Core process-management structures.
- Device subsystems.
- Storage support.
- Networking support.
- Security mechanisms.
- Virtual filesystems and kernel interfaces.

The kernel also detects and configures hardware according to the system architecture and available drivers.

### Conceptual flow

```text
Linux Kernel Starts
        |
        v
Memory and CPU Initialization
        |
        v
Core Kernel Subsystems
        |
        v
Hardware and Device Detection
        |
        v
Early User Space
        |
        v
Root Filesystem Preparation
```

The exact order contains many internal details, but this model shows the main transition from a loaded kernel to a functioning operating-system core.

### Hardware detection

The kernel identifies available hardware and activates relevant support.

This may include:

- Processors.
- Memory.
- Storage devices.
- Network adapters.
- USB devices.
- Input devices.
- Virtual devices.

A device may be physically present but unusable if appropriate kernel support or firmware is missing.

## 12. Root Filesystem

The **root filesystem** is the main filesystem mounted as the top of the Linux filesystem hierarchy.

It is represented conceptually by:

```text
/
```

The root filesystem contains the operating-system environment, including areas for:

- System programs.
- Libraries.
- Configuration.
- Device interfaces.
- Runtime data.
- User data.
- Temporary files.
- System logs.

Filesystem hierarchy will be taught in a later topic. Here, the important point is that the root filesystem provides the permanent operating-system environment after the temporary initramfs stage.

### Switching to the real root filesystem

The early boot environment helps the kernel and init system locate and prepare the real root filesystem.

Conceptually:

```text
Temporary initramfs Environment
              |
              v
Find and prepare root device
              |
              v
Mount real root filesystem
              |
              v
Continue normal system startup
```

The initramfs may remain involved briefly during the transition, but the normal system operates from the real root filesystem.

### Root filesystem versus initramfs

| initramfs | Root filesystem |
|---|---|
| Temporary early boot environment. | Main permanent operating-system filesystem. |
| Loaded into memory. | Usually stored on persistent storage. |
| Helps access the real root filesystem. | Contains the full operating-system environment. |
| Used before normal system startup. | Used during normal system operation. |
| Usually smaller and specialized. | Contains system software, services, configuration, and applications. |

## 13. PID 1 and the Init System

### What is PID 1?

A process identifier, or PID, identifies a running process.

**PID 1** is the first user-space process started during normal Linux system initialization.

It has special responsibilities, including:

- Starting system services.
- Coordinating system initialization.
- Managing service relationships.
- Receiving orphaned processes.
- Helping coordinate shutdown.

The kernel starts the initial user-space process after its early initialization work. The root filesystem or initramfs provides the program that becomes PID 1.

### The init system

The **init system** is the software responsible for bringing the user-space system into an operational state.

It may:

- Start essential services.
- Establish service dependencies.
- Mount required filesystems.
- Configure runtime environments.
- Start login services.
- Monitor or restart services.
- Coordinate shutdown.

### systemd

**systemd** is a widely used init system and system manager. When it runs as PID 1, it starts and maintains user-space services and manages system startup. [documentation.suse](https://documentation.suse.com/smart/systems-management/html/systemd-basics/index.html)

Many modern Linux distributions use systemd by default, but not every Linux system uses it. Other init systems exist.

### PID 1 flow

```text
Linux Kernel
     |
     v
Initial User-Space Process
     |
     v
PID 1 / Init System
     |
     v
System Services
```

PID 1 is not the same as the Linux kernel. It is a user-space process started by the kernel.

## 14. System Initialization

Once PID 1 is running, the system begins normal user-space initialization.

### System initialization tasks

The init system may coordinate:

- Required filesystem mounts.
- Device management.
- Logging.
- Networking.
- Time synchronization.
- Service startup.
- User-session support.
- Login services.
- Background infrastructure.

The exact order depends on:

- Distribution.
- Init system.
- Service dependencies.
- Hardware.
- Configuration.
- Boot target.
- Availability of required resources.

### Dependency handling

Some services depend on other services or resources.

For example:

```text
Network Configuration
        |
        v
Database Service
        |
        v
Application Service
```

The application service may not start correctly until the database and network are available.

Modern init systems can represent these dependencies and coordinate startup accordingly.

### Boot targets

A system may have different startup states for different purposes, such as:

- Minimal maintenance.
- Text-based server operation.
- Graphical desktop operation.
- Recovery or emergency operation.

The selected state determines which services and user interfaces are started.

Detailed service and init-system administration belongs to later topics.

## 15. System Services

A **system service** is a background program that performs a function for the operating system or other applications.

Examples include:

- Network services.
- Logging services.
- Web servers.
- Database systems.
- Monitoring agents.
- Backup systems.
- Remote-access services.
- Time synchronization services.

### Service startup flow

```text
PID 1 / Init System
        |
        v
Service Dependency Evaluation
        |
        v
Service Process Started
        |
        v
Service Uses Kernel Resources
        |
        v
System Becomes Operational
```

A service normally runs in user space and uses the Linux kernel for:

- CPU time.
- Memory.
- Network access.
- Storage access.
- Process management.
- Device access.
- Security enforcement.

### Service failure

If a service fails during boot:

- The system may continue with reduced functionality.
- Dependent services may fail.
- The init system may retry the service.
- The system may report a degraded state.
- Users may be unable to log in or access applications.

A boot process can complete even when one nonessential service fails. Conversely, failure of a critical service may prevent the system from being usable.

## 16. Login Environment and User Session

After essential system services start, the system provides a login environment.

This may be:

- A text-based login service.
- A graphical login manager.
- A remote-access endpoint.
- A cloud-console login environment.
- An automated service-only environment.

### Login flow

```text
System Services
       |
       v
Login Service
       |
       v
User Authentication
       |
       v
User Session
       |
       v
User Applications
```

### User session

A user session may include:

- User identity.
- Environment information.
- Access to user files.
- User-specific services.
- Shell or graphical interface.
- Application processes.

A server may complete boot without any interactive user session. For example, a database server or web server can start its services and wait for network requests without a human logging in.

### Login is not the same as boot

Booting prepares the operating system and system services.

Logging in starts a user-specific environment after the system is operational.

```text
Boot
  |
  v
Operating-System Initialization
  |
  v
System Services
  |
  v
Login Environment
  |
  v
User Session
```

## 17. Full Boot Process Walkthrough

The complete conceptual sequence is:

### Stage 1: Power on

The system receives power and the processor begins execution.

### Stage 2: Firmware starts

BIOS or UEFI performs early hardware initialization.

### Stage 3: POST

The firmware performs initial checks on essential hardware.

### Stage 4: Boot device selection

The firmware selects an internal device, removable device, network source, or virtual boot entry.

### Stage 5: Bootloader starts

The firmware loads a bootloader such as GRUB or starts another configured boot application.

### Stage 6: Bootloader selects boot files

The bootloader selects a kernel image and, when required, an initramfs image.

### Stage 7: Kernel loads

The bootloader places the kernel and initramfs in memory and transfers control to the kernel.

### Stage 8: Kernel initializes

The kernel initializes memory, processors, core subsystems, and available devices.

### Stage 9: initramfs runs

The temporary early-user-space environment provides the support needed to find and prepare the real root filesystem.

### Stage 10: Root filesystem becomes available

The main root filesystem is located and mounted as the permanent operating-system environment.

### Stage 11: PID 1 starts

The init system begins user-space initialization.

### Stage 12: System services start

The init system starts required services according to dependencies and system configuration.

### Stage 13: Login environment becomes available

The system provides text-based, graphical, remote, or service-oriented access.

### Stage 14: User applications run

Users or automated systems start applications that use the operating-system environment.

## 18. Real-World Example: Ubuntu Server

Consider an Ubuntu Server virtual machine hosting a web application.

```text
Cloud Virtual Hardware
        |
        v
Virtual Firmware
        |
        v
Bootloader
        |
        v
Linux Kernel
        |
        v
initramfs
        |
        v
Root Filesystem
        |
        v
PID 1 / systemd
        |
        +---- Network Service
        +---- Logging Service
        +---- Web Server
        +---- Monitoring Agent
        |
        v
Application Requests
```

### What happens conceptually

1. The cloud platform presents virtual CPU, memory, storage, and network devices.
2. Virtual firmware begins the boot process.
3. The bootloader loads the kernel and initramfs.
4. The kernel initializes virtual hardware and core subsystems.
5. The initramfs helps locate and prepare the root filesystem.
6. The root filesystem becomes available.
7. systemd starts as PID 1.
8. Networking and logging services start.
9. The web server starts.
10. Monitoring begins collecting system and service information.
11. The server accepts application requests.

No human needs to log in for the web application to serve requests.

## 19. Boot Process in an Enterprise Environment

In an enterprise data center, a server may boot as part of a larger architecture:

```text
Power
  |
  v
Server Firmware
  |
  v
Bootloader
  |
  v
Linux Kernel
  |
  v
Initramfs
  |
  v
Root Filesystem
  |
  v
Init System
  |
  +---- Network Configuration
  +---- Storage Services
  +---- Monitoring Agent
  +---- Logging Agent
  +---- Security Agent
  +---- Application Services
  |
  v
Production Workload
```

Different engineering roles may care about different boot stages:

- **System administrators** investigate kernel, initramfs, root filesystem, and service startup issues.
- **System engineers** design boot standards, operating-system images, and recovery processes.
- **Network engineers** depend on network services becoming available after boot.
- **Security engineers** review firmware settings, boot integrity, kernel security, and startup services.
- **Cloud engineers** manage images, virtual hardware, initialization, and automated provisioning.
- **DevOps engineers** ensure systems reach the expected operational state during automated deployment.
- **SREs** monitor boot reliability and service readiness.
- **Backend developers** depend on application services becoming available after system initialization.

## 20. Common Boot Failure Locations

The boot sequence can be divided into failure areas.

```text
Firmware
   |
Bootloader
   |
Kernel
   |
initramfs
   |
Root Filesystem
   |
PID 1 / Init System
   |
System Services
   |
Login / Application
```

### Firmware-stage failures

Possible symptoms include:

- No display.
- Hardware diagnostic errors.
- No usable boot device.
- Incorrect boot order.
- Firmware configuration problems.

### Bootloader-stage failures

Possible symptoms include:

- Boot menu does not appear.
- Kernel entry is missing.
- Bootloader cannot locate files.
- The system returns to firmware.

### Kernel-stage failures

Possible symptoms include:

- Kernel startup stops.
- Hardware is not detected.
- Kernel cannot initialize a required subsystem.
- The system reports a severe kernel error.

### initramfs-stage failures

Possible symptoms include:

- Root storage cannot be found.
- Required storage drivers are unavailable.
- Encryption or volume preparation fails.
- The system cannot continue to the real root filesystem.

### Root-filesystem-stage failures

Possible symptoms include:

- Root filesystem cannot be mounted.
- Filesystem errors prevent access.
- Required operating-system files are unavailable.
- Startup reaches an emergency environment.

### Init-system failures

Possible symptoms include:

- PID 1 does not start.
- Services are not launched.
- The system reaches a limited state.
- Login is unavailable.

### Service-stage failures

Possible symptoms include:

- The operating system boots but an application is unavailable.
- Networking does not start.
- Logging is incomplete.
- Dependent services fail.

A system can boot successfully while still having service-level problems.

## 21. Practical Mental Model

Remember the boot process as a transfer of responsibility:

```text
Firmware
  |
  | Initializes hardware and selects a boot entry
  v
Bootloader
  |
  | Loads the kernel and initramfs
  v
Linux Kernel
  |
  | Initializes the operating-system core
  v
initramfs
  |
  | Prepares access to the real root filesystem
  v
Root Filesystem
  |
  | Provides the permanent operating-system environment
  v
PID 1 / Init System
  |
  | Starts and coordinates user-space services
  v
System Services
  |
  | Provide networking, logging, applications, and other functions
  v
Login Environment / User Session
```

A shorter memory aid is:

```text
Firmware
    |
Bootloader
    |
Kernel
    |
initramfs
    |
Root Filesystem
    |
PID 1
    |
Services
    |
User Session
```

Each stage prepares the next stage.

## 22. Common Misconceptions

### “BIOS or UEFI is Linux”

BIOS and UEFI are firmware interfaces. They start before Linux and help locate the bootloader or operating-system entry.

### “The bootloader is the kernel”

The bootloader loads the kernel and transfers control to it. It is not the operating-system kernel.

### “The initramfs is the permanent root filesystem”

The initramfs is a temporary early boot environment. It helps the system reach the real root filesystem.

### “The kernel starts all applications directly”

The kernel starts the initial user-space process. The init system then coordinates the startup of services and other user-space programs.

### “PID 1 is the Linux kernel”

PID 1 is a user-space process. It is started by the kernel and performs system initialization.

### “Systemd is the Linux kernel”

systemd is an init system and system manager that commonly runs as PID 1. It is separate from the kernel.

### “Boot complete means every service is healthy”

The system may reach a login environment while individual services remain failed or degraded.

### “A server must have a human login session to be operational”

A server can boot and provide network services without any interactive user logging in.

### “Every Linux system uses GRUB”

GRUB is common, but other bootloaders and firmware-based boot methods are also used.

### “Every Linux system uses systemd”

Many major distributions use systemd by default, but alternative init systems exist.

## 23. Professional Relevance

### System administration

Boot knowledge helps administrators understand:

- Why a server does not start.
- Why storage is unavailable during boot.
- Why a service fails after reboot.
- Why an initramfs is required.
- Where to investigate early startup failures.

### System engineering

System engineers use boot concepts to design:

- Standard operating-system images.
- Reliable server startup.
- Boot policies.
- Storage layouts.
- Service dependencies.
- Recovery and maintenance processes.

### Cloud engineering

Cloud engineers work with:

- Virtual firmware.
- Machine images.
- Virtual disks.
- Cloud-init or similar initialization systems.
- Automated service startup.
- Instance health checks.

The underlying principles remain similar even when the physical hardware is hidden by the provider.

### DevOps

DevOps engineers need to understand:

- How images become running systems.
- How services become ready.
- How deployment automation waits for readiness.
- Why a machine may be reachable but not operational.
- How boot-time configuration affects applications.

### Security engineering

Security engineers may review:

- Firmware configuration.
- Secure Boot.
- Bootloader integrity.
- Kernel loading.
- Initramfs contents.
- Startup services.
- Boot-time trust boundaries.

This topic introduces these areas conceptually without teaching security procedures.

### Site reliability engineering

SREs care about:

- Reboot reliability.
- Service readiness.
- Startup duration.
- Failed dependencies.
- Automated recovery.
- Monitoring during system initialization.

## 24. Summary

The Linux boot process begins when the machine powers on and ends when the operating system and required services are ready.

The major stages are:

1. Firmware starts.
2. POST checks and initializes hardware.
3. A boot device is selected.
4. The bootloader starts.
5. The bootloader loads the Linux kernel.
6. The initramfs provides early user space.
7. The kernel initializes memory, processors, devices, and subsystems.
8. The real root filesystem becomes available.
9. PID 1 starts.
10. The init system starts system services.
11. A login environment or service-only state becomes available.
12. Users and applications interact with the running system.

The most important distinctions are:

- Firmware starts before Linux.
- The bootloader loads the kernel.
- The kernel manages the operating-system core.
- The initramfs helps locate and prepare the root filesystem.
- PID 1 initializes user space.
- Services provide background functionality.
- A user session is separate from system boot.

## 25. Knowledge Check

1. What is the first software layer that normally runs after power is applied?
2. What is firmware?
3. What is the difference between BIOS and UEFI?
4. What is POST?
5. How does the system select a boot device?
6. What is the role of a bootloader?
7. What does GRUB commonly do?
8. What is a Linux kernel image?
9. Why is initramfs needed?
10. What is the difference between initramfs and the real root filesystem?
11. What does the kernel initialize after the bootloader transfers control?
12. What is PID 1?
13. What is the role of an init system?
14. What is systemd?
15. Why can a system boot successfully while an application service remains unavailable?
16. Why does a cloud virtual machine still have a conceptual boot process even when physical hardware is hidden?
17. Which boot stage would you investigate if the root storage device could not be found?
18. Which boot stage would you investigate if the operating system started but the web service did not?

## 26. Completion Checklist

You should now understand:

- [ ] The complete Linux boot sequence.
- [ ] The role of power-on and firmware.
- [ ] The difference between BIOS and UEFI.
- [ ] The purpose of POST.
- [ ] Boot device selection.
- [ ] The role of a bootloader.
- [ ] GRUB as a common Linux bootloader.
- [ ] The purpose of a Linux kernel image.
- [ ] The purpose of initramfs.
- [ ] The concept of early user space.
- [ ] Kernel initialization.
- [ ] Hardware detection during kernel startup.
- [ ] The purpose of the root filesystem.
- [ ] The difference between initramfs and the real root filesystem.
- [ ] The meaning of PID 1.
- [ ] The role of the init system.
- [ ] The role of systemd at a conceptual level.
- [ ] How system services start.
- [ ] The difference between boot completion and user login.
- [ ] The difference between a user session and a service-only server.
- [ ] Common locations where boot failures can occur.
- [ ] How the boot process appears in a Linux server and cloud environment.

## 27. Next Topic

**Topic 05 — Linux Kernel**
