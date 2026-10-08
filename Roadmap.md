Linux System & Network Engineer — Practical Learning Roadmap

Practical, Lab-Driven Linux Engineering Curriculum

Target Role: System & Network Engineer
Secondary Roles: Linux Administrator, Infrastructure Engineer, Security Engineer, DevOps/Cloud Engineer
Primary Platform: Ubuntu Server 24.04 LTS
Lab Environment: VMware Workstation, Ubuntu Server, Kali Linux, Windows, Docker
Learning Model: 20–30% theory + 70–80% practical work
Recommended Duration: 24–28 weeks
Recommended Study Time: 1.5–2 hours/day, 5 days/week

---

1. Learning Philosophy

The goal of this roadmap is not to memorize Linux commands or complete a video course.

The objective is to become capable of:

Deploy
   ↓
Configure
   ↓
Administer
   ↓
Monitor
   ↓
Secure
   ↓
Troubleshoot
   ↓
Automate
   ↓
Recover
   ↓
Document
   ↓
Improve

Every phase therefore contains:

- Concepts
- Commands/tools
- Practical labs
- Troubleshooting exercises
- Automation tasks
- Documentation
- A project or portfolio artifact

The final outcome should be demonstrable Linux engineering capability rather than course completion.

---

2. Recommended Lab Architecture

Start with a small virtual environment.

                    Windows Host
                         │
                 VMware Workstation
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
    Ubuntu Server 01   Ubuntu Server 02   Kali Linux
       Linux Admin       Services         Security
          │              │
          └──────────────┘
                  │
             Docker Labs

Initial machines

Machine| Purpose
Ubuntu Server 01| Primary administration server
Ubuntu Server 02| Services/testing server
Kali Linux| Authorized security testing
Windows Host| Administration/client machine

With an 8 GB host, do not run every VM simultaneously. Use snapshots and start only the machines required for each lab.

---

3. Phase 0 — Lab Environment and Engineering Workflow

Duration

2–3 days

Topics

- VMware virtual machines
- Ubuntu Server installation
- VM networking
- NAT
- Bridged networking
- Host-only networking
- VM snapshots
- Resource allocation
- SSH access
- Linux terminal
- Git/GitHub workflow

Practical Labs

Lab 0.1 — Ubuntu Server Installation

Install Ubuntu Server and configure:

Hostname
User
Timezone
Network
SSH
Updates

Verify:

hostnamectl
uname -a
cat /etc/os-release
ip addr
ip route

Lab 0.2 — VM Network Modes

Test:

NAT
Host-only
Bridged

Document:

- IP address
- Gateway
- DNS
- Connectivity
- Use case for each mode

Lab 0.3 — Snapshot and Recovery

Create:

clean-install
baseline
configured-server

Intentionally modify the VM and restore a snapshot.

Project

Linux Engineering Lab Environment

Create a repository containing:

linux-lab-environment/
├── README.md
├── architecture/
├── vm-config/
├── networking/
├── snapshots/
└── troubleshooting/

---

4. Phase 1 — Linux Fundamentals and Architecture

Duration

1 week

Topics

Linux Fundamentals

- What is Linux?
- Linux kernel
- GNU/Linux
- Distribution
- Ubuntu
- Server vs desktop
- Open-source model
- Linux architecture
- User space
- Kernel space
- System calls
- Processes
- Filesystems
- Networking
- Device drivers

Linux Architecture

Applications
     ↓
Shell / Libraries / Utilities
     ↓
System Calls
     ↓
Linux Kernel
     ├── Process Management
     ├── Memory Management
     ├── Networking
     ├── VFS
     ├── Drivers
     ├── Security
     └── IPC
     ↓
Hardware

Boot Architecture

Firmware
   ↓
Bootloader
   ↓
Kernel
   ↓
initramfs
   ↓
PID 1
   ↓
systemd
   ↓
Services
   ↓
User Environment

Practical Labs

Lab 1.1 — Identify Linux System

uname -a
hostnamectl
cat /etc/os-release
lscpu
free -h
lsblk

Lab 1.2 — Explore Kernel Information

uname -r
cat /proc/version
cat /proc/cpuinfo
cat /proc/meminfo

Lab 1.3 — Inspect Boot Logs

journalctl -b
journalctl -k
dmesg

Lab 1.4 — Identify PID 1

ps -p 1 -f
systemctl status

Project

Linux System Inventory Tool

Build a Bash or Python tool that reports:

Hostname
OS
Kernel
CPU
Memory
Disk
Uptime
Network interfaces
IP addresses
Default route
DNS
Running services

---

5. Phase 2 — Linux Filesystem and Storage Concepts

Duration

1 week

Topics

- Linux filesystem hierarchy
- "/"
- "/etc"
- "/var"
- "/usr"
- "/home"
- "/opt"
- "/tmp"
- "/boot"
- "/dev"
- "/proc"
- "/sys"
- "/run"
- Files
- Directories
- Inodes
- Directory entries
- Hard links
- Symbolic links
- Mount points
- Filesystem types
- Block devices
- Virtual filesystems

Practical Labs

Lab 2.1 — Filesystem Exploration

Explore:

ls /
ls /etc
ls /var
ls /proc
ls /sys
ls /dev

Lab 2.2 — File Types

Create and identify:

Regular file
Directory
Symbolic link
Hard link
Device
Socket
FIFO

Use:

file
ls -l
stat

Lab 2.3 — Inode Investigation

ls -li
stat file.txt

Demonstrate that:

filename → directory entry → inode → data

Lab 2.4 — Hard Link vs Symbolic Link

Create both and document their behavior when the original filename is removed.

Lab 2.5 — "/proc" and "/sys"

Investigate:

/proc/cpuinfo
/proc/meminfo
/proc/loadavg
/proc/mounts
/sys/class/net

Project

Linux Filesystem Investigation Toolkit

Build scripts for:

- Largest directories
- Largest files
- Recent files
- Inode usage
- Mount information
- Filesystem type
- Broken symbolic links

---

6. Phase 3 — Command Line and Shell

Duration

1–1.5 weeks

Topics

- Bash
- Shell vs terminal
- Commands
- Arguments
- Options
- Environment variables
- PATH
- Command history
- Aliases
- stdin
- stdout
- stderr
- Redirection
- Pipes
- Command substitution
- Quoting
- Wildcards
- Exit status
- Command chaining

Core Commands

pwd
ls
cd
cp
mv
rm
mkdir
touch
cat
less
head
tail
grep
find
sort
uniq
cut
awk
sed
wc
xargs
file
stat

Practical Labs

Lab 3.1 — File Management

Create and manipulate a simulated company directory.

Lab 3.2 — Pipeline Engineering

Build command pipelines using:

|
>
>>
2>
2>&1

Lab 3.3 — Log Filtering

Extract useful information from:

/var/log/

Lab 3.4 — Command Exit Codes

Test:

echo $?

with successful and failed commands.

Project

Linux Log Analyzer

Analyze authentication logs and generate:

Failed login count
Successful login count
Top usernames
Top source IPs
Latest events

---

7. Phase 4 — Users, Groups and Identity Management

Duration

1 week

Topics

- Users
- Groups
- UID
- GID
- Primary groups
- Supplementary groups
- Root
- "/etc/passwd"
- "/etc/shadow"
- "/etc/group"
- "/etc/sudoers"
- "sudo"
- "su"
- Login shells
- Home directories
- Password policies

Commands

useradd
usermod
userdel
groupadd
groupmod
groupdel
passwd
id
groups
who
w
last
su
sudo

Practical Labs

Lab 4.1 — User Lifecycle

Create:

admin01
developer01
operator01

Configure different privileges.

Lab 4.2 — Department Groups

Create:

developers
network
security
management

Lab 4.3 — Sudo Delegation

Give specific administrative capabilities without giving unrestricted root access.

Project

Multi-User Enterprise Linux Server

Simulate:

Company
├── Management
├── Developers
├── Network
├── Security
└── Shared

Implement appropriate access controls.

---

8. Phase 5 — Permissions, ACLs and Linux Security Model

Duration

1 week

Topics

- "rwx"
- Owner
- Group
- Others
- Numeric permissions
- Symbolic permissions
- "chmod"
- "chown"
- "chgrp"
- "umask"
- SUID
- SGID
- Sticky bit
- POSIX ACL
- Linux capabilities
- Least privilege

Practical Labs

Lab 5.1 — Permission Matrix

Build directories with different access requirements.

Lab 5.2 — Shared Directory

Use:

SGID + group ownership

to create a team workspace.

Lab 5.3 — Sticky Directory

Demonstrate why shared temporary directories use the sticky bit.

Lab 5.4 — ACL

Use:

getfacl
setfacl

to provide user-specific access.

Lab 5.5 — SUID/SGID Investigation

Find:

find / -perm -4000
find / -perm -2000

and understand the security implications.

Project

Linux Access Control Lab

Create a documented permission model for a simulated company server.

---

9. Phase 6 — Process Management

Duration

1 week

Topics

- Program vs process
- PID
- PPID
- Process tree
- Process states
- Threads
- Foreground processes
- Background processes
- Jobs
- Signals
- Daemons
- Exit status
- Zombies
- Orphans
- CPU priority
- "nice"
- "renice"

Commands

ps
top
htop
pstree
pgrep
pkill
kill
jobs
fg
bg
nohup
nice
renice

Practical Labs

Lab 6.1 — Process Tree

Trace:

systemd
  └── service
       └── process
            └── shell
                 └── command

Lab 6.2 — Signal Handling

Test:

SIGINT
SIGTERM
SIGKILL
SIGSTOP
SIGCONT

Lab 6.3 — Zombie Process

Create and identify a controlled zombie process.

Lab 6.4 — CPU Priority

Experiment with:

nice
renice

Project

Linux Process Watchdog

Monitor selected processes and report:

PID
CPU
Memory
State
Runtime
Process status

Optionally restart a failed service.

---

10. Phase 7 — systemd, Services and Boot Management

Duration

1 week

Topics

- systemd
- PID 1
- Units
- Service units
- Target units
- Socket units
- Timer units
- Mount units
- Dependencies
- Service lifecycle
- Boot targets
- Service failures
- Journald

Commands

systemctl
journalctl
systemd-analyze

Practical Labs

Lab 7.1 — Service Lifecycle

Practice:

systemctl start
systemctl stop
systemctl restart
systemctl enable
systemctl disable
systemctl status

Lab 7.2 — Create a Custom systemd Service

Create a service that runs your own script.

Lab 7.3 — systemd Timer

Replace a simple cron task with a systemd timer.

Lab 7.4 — Boot Analysis

systemd-analyze
systemd-analyze blame

Project

Linux Service Health Manager

Features:

Service status
Failure detection
Restart
Journal extraction
Health report

---

11. Phase 8 — Package Management and System Updates

Duration

4–5 days

Topics

- APT
- DPKG
- Repositories
- Package dependencies
- Package configuration
- Package removal
- Package verification
- Broken packages
- Security updates
- Automatic updates

Commands

apt
apt-cache
dpkg
apt-mark

Labs

Lab 8.1

Install and remove packages.

Lab 8.2

Investigate dependencies.

Lab 8.3

Recover from a controlled package configuration problem.

Lab 8.4

Configure automatic security updates.

Project

Linux Patch Management Tool

Generate:

Installed packages
Available updates
Security updates
Reboot requirement
Patch timestamp

---

12. Phase 9 — Linux Networking Fundamentals

Duration

1.5 weeks

Topics

- Network interfaces
- Ethernet
- MAC address
- IPv4
- IPv6
- Subnetting
- Default gateway
- Routing
- ARP
- DNS
- DHCP
- TCP
- UDP
- Ports
- Sockets
- Network namespaces

Commands

ip
ss
ping
traceroute
tracepath
mtr
arp
resolvectl
hostnamectl

Practical Labs

Lab 9.1 — Interface Investigation

ip addr
ip link

Lab 9.2 — Routing

ip route

Identify:

Connected routes
Default route
Gateway

Lab 9.3 — Port Investigation

ss -tulpn

Map:

Process → Socket → Port → Service

Lab 9.4 — DNS Investigation

dig
nslookup
resolvectl

Lab 9.5 — Packet Capture

Use:

tcpdump
Wireshark

to observe:

DNS
ICMP
TCP
HTTP
SSH

Project

Linux Network Diagnostic Toolkit

Output:

Interface      OK
IP             OK
Gateway        OK
DNS            OK
Internet       OK
Latency        12 ms
Packet Loss    0%
Listening Ports

---

13. Phase 10 — Netplan and Advanced Linux Networking

Duration

1 week

Topics

- Netplan
- Static IP
- DHCP
- Routes
- DNS configuration
- Multiple interfaces
- VLAN concepts
- Bonding concepts
- Bridges
- Network namespaces
- Virtual Ethernet pairs
- Policy routing
- Network troubleshooting

Practical Labs

Lab 10.1 — Static IP

Configure persistent networking using Netplan.

Lab 10.2 — Multiple Interfaces

Configure two interfaces and test routing.

Lab 10.3 — Network Namespace

Create isolated network namespaces.

Lab 10.4 — Linux Bridge

Create a software bridge and understand its forwarding behavior.

Lab 10.5 — VLAN Lab

Create a VLAN interface in a controlled virtual environment.

Project

Linux Virtual Network Lab

Document:

Namespaces
Interfaces
VLANs
Routes
Bridges
DNS
Connectivity

---

14. Phase 11 — SSH and Remote Administration

Duration

4–5 days

Topics

- SSH client/server
- SSH keys
- Public/private key authentication
- "sshd_config"
- SSH hardening
- SCP
- SFTP
- rsync
- SSH tunneling
- Local forwarding
- Remote forwarding
- ProxyJump
- Agent forwarding
- Remote commands

Practical Labs

Lab 11.1 — Key Authentication

Disable password authentication in a controlled environment after validating key access.

Lab 11.2 — Secure Administration

Restrict SSH access to appropriate users.

Lab 11.3 — SSH Tunneling

Build a safe internal tunnel between lab VMs.

Lab 11.4 — Remote Automation

Execute commands remotely using SSH.

Project

Secure Linux Remote Administration Environment

Admin
 ├── SSH → Web Server
 ├── SSH → DNS Server
 └── SSH → Monitoring Server

---

15. Phase 12 — Storage, Filesystems and LVM

Duration

1 week

Topics

- HDD
- SSD
- Block devices
- Partitions
- GPT
- MBR
- Filesystems
- ext4
- XFS concepts
- Mounting
- "/etc/fstab"
- Mount options
- "lsblk"
- "blkid"
- "fdisk"
- "parted"
- Filesystem checks
- Disk usage
- Inode usage
- Swap
- LVM
- PV
- VG
- LV
- Resize operations
- Disk health
- SMART concepts
- RAID concepts

Practical Labs

Lab 12.1 — Partition Investigation

lsblk
blkid
fdisk -l

Lab 12.2 — Filesystem Creation

Create a filesystem on a disposable virtual disk.

Lab 12.3 — Persistent Mount

Configure "/etc/fstab".

Lab 12.4 — LVM

Build:

Disk
 ↓
PV
 ↓
VG
 ↓
LV
 ↓
Filesystem
 ↓
Mount

Lab 12.5 — LVM Resize

Practice extending a logical volume and filesystem.

Lab 12.6 — Swap

Create and manage a swap area.

Project

Linux Storage Administration Lab

Document the complete storage lifecycle:

Disk
→ Partition
→ Filesystem
→ Mount
→ LVM
→ Monitoring
→ Expansion
→ Recovery

---

16. Phase 13 — Logging, Monitoring and Observability

Duration

1 week

Topics

- journald
- journalctl
- syslog
- "/var/log"
- Authentication logs
- Kernel logs
- Service logs
- Log rotation
- CPU monitoring
- Memory monitoring
- Disk monitoring
- Network monitoring
- Load average
- Open file descriptors
- Socket monitoring

Commands

journalctl
dmesg
top
htop
free
uptime
vmstat
iostat
df
du
lsof
ss

Practical Labs

Lab 13.1 — Journal Investigation

Find service failures using "journalctl".

Lab 13.2 — Resource Monitoring

Create CPU/memory/disk stress in a controlled VM and investigate.

Lab 13.3 — Log Rotation

Understand and test logrotate.

Lab 13.4 — Network Observation

Use:

ss
tcpdump

to investigate active connections.

Project

Linux Server Health Monitor

Monitor:

CPU
Memory
Disk
Load
Network
Processes
Services

Generate periodic reports.

---

17. Phase 14 — Linux Security and Hardening

Duration

1 week

Topics

- Linux security model
- Least privilege
- User security
- Sudo
- Permissions
- ACL
- Capabilities
- SUID/SGID
- SSH hardening
- Firewall
- UFW
- nftables concepts
- Netfilter
- AppArmor
- SELinux concepts
- PAM concepts
- Audit logging
- File integrity
- Patch management
- Vulnerability management
- Security baselines

Practical Labs

Lab 14.1 — UFW

Configure:

SSH
HTTP
HTTPS

and deny unnecessary inbound traffic.

Lab 14.2 — SSH Hardening

Implement:

Key authentication
Restricted users
Reduced attack surface
Logging

Lab 14.3 — AppArmor

Inspect active profiles and understand enforcement.

Lab 14.4 — SUID Audit

Identify unusual SUID files.

Lab 14.5 — Security Baseline

Create a checklist for:

Users
SSH
Firewall
Services
Updates
Permissions
Logging
AppArmor

Project

Linux Server Security Hardening

Produce:

before-hardening.md
hardening.sh
security-check.sh
after-hardening.md

Do not claim the server is “fully secure”; document the specific controls implemented and their limitations.

---

18. Phase 15 — DNS Administration

Duration

4–5 days

Topics

- DNS architecture
- Resolver
- Authoritative server
- Recursive server
- Forward lookup
- Reverse lookup
- A
- AAAA
- CNAME
- MX
- TXT
- PTR
- SOA
- TTL
- DNS caching
- BIND
- DNS troubleshooting

Practical Labs

Lab 15.1 — Local DNS Server

Deploy BIND in the lab.

Lab 15.2 — Forward Zone

Create:

server01.lab.local
web01.lab.local

Lab 15.3 — Reverse DNS

Configure PTR records.

Lab 15.4 — DNS Troubleshooting

Intentionally introduce:

- Wrong record
- Missing record
- Wrong nameserver
- Resolution failure

Investigate using:

dig
nslookup
resolvectl

Project

Enterprise DNS Infrastructure

Create:

Authoritative DNS
Forward zone
Reverse zone
Clients
DNS troubleshooting documentation

---

19. Phase 16 — DHCP and Network Services

Duration

4–5 days

Topics

- DHCP process
- Discover
- Offer
- Request
- Acknowledge
- Address pools
- Leases
- Reservations
- Gateway assignment
- DNS assignment
- DHCP troubleshooting

Practical Labs

Lab 16.1 — DHCP Server

Deploy a DHCP service in an isolated network.

Lab 16.2 — Reservations

Assign a predictable address to a test client.

Lab 16.3 — DHCP Troubleshooting

Investigate:

No lease
Wrong gateway
Wrong DNS
Address exhaustion

Project

Linux Network Infrastructure

Combine:

DHCP
DNS
SSH
Firewall
Web Server

This becomes one of the flagship portfolio projects.

---

20. Phase 17 — Web and Network Services

Duration

1 week

Topics

Nginx

- HTTP
- HTTPS
- Server blocks
- Static files
- Reverse proxy
- Access logs
- Error logs
- TLS concepts

Apache

- Virtual hosts
- DocumentRoot
- Modules
- Logs
- Permissions

Other Services

- NFS
- Samba/SMB
- SFTP
- Database services
- Application services

Practical Labs

Lab 17.1 — Nginx

Deploy a static website.

Lab 17.2 — Virtual Hosts

Host multiple test domains.

Lab 17.3 — Reverse Proxy

Client
 ↓
Nginx
 ↓
Python Application

Lab 17.4 — NFS

Share a directory between Ubuntu servers.

Lab 17.5 — Samba

Create a controlled SMB share for a Windows client.

Project

Linux Application Server

Build:

Nginx
 ↓
Python Application
 ↓
Database

Managed using systemd.

---

21. Phase 18 — Bash Scripting and Automation

Duration

1 week

Topics

- Variables
- Input
- Output
- Conditions
- Case
- Loops
- Functions
- Arrays
- Arguments
- Exit codes
- Command substitution
- Error handling
- Logging
- Debugging
- Cron
- systemd timers
- Automation design

Practical Labs

Lab 18.1 — Backup Script

Lab 18.2 — User Management Script

Lab 18.3 — Service Health Script

Lab 18.4 — Disk Alert Script

Lab 18.5 — Network Diagnostic Script

Project

Linux Administration Toolkit

linux-admin-toolkit/
├── health-check.sh
├── backup.sh
├── disk-monitor.sh
├── network-check.sh
├── service-check.sh
├── user-manager.sh
├── log-analyzer.sh
└── README.md

---

22. Phase 19 — Backup, Recovery and Disaster Recovery

Duration

1 week

Topics

- Backup strategy
- Full backup
- Incremental backup
- Differential backup
- "tar"
- "rsync"
- Local backup
- Remote backup
- Backup scheduling
- Retention
- Verification
- Restore
- Disaster recovery
- Configuration backup

Practical Labs

Lab 19.1 — Full Backup

Lab 19.2 — Incremental Backup

Lab 19.3 — Remote Backup

Lab 19.4 — Backup Verification

Lab 19.5 — Data Recovery

Delete test data and restore it.

Major Project

Linux Backup and Disaster Recovery System

Architecture:

Production Server
       │
       ├── Daily Backup
       ├── Retention
       └── Verification
               │
               ▼
         Backup Server

The project is incomplete until a restore has been successfully tested.

---

23. Phase 20 — Linux Troubleshooting

Duration

1 week

Topics

- Troubleshooting methodology
- Boot failures
- Service failures
- Permission failures
- DNS failures
- Network failures
- Storage failures
- High CPU
- High memory
- Full disk
- Broken mounts
- Port conflicts
- Application failures
- Authentication failures

Troubleshooting Workflow

Problem
 ↓
Collect symptoms
 ↓
Identify affected layer
 ↓
Check logs
 ↓
Check service
 ↓
Check process
 ↓
Check network
 ↓
Check resources
 ↓
Form hypothesis
 ↓
Apply controlled fix
 ↓
Verify
 ↓
Document

Practical Incident Labs

Create at least:

1. SSH failure
2. DNS failure
3. Nginx failure
4. Full filesystem
5. Permission failure
6. High CPU
7. High memory
8. Broken mount
9. Wrong route
10. Port conflict
11. Failed systemd service
12. Package configuration failure

Major Project

Linux Incident Response Lab

For every incident document:

Incident
Symptoms
Impact
Evidence
Commands
Investigation
Root Cause
Fix
Verification
Prevention

This is one of the most important portfolio projects in the roadmap.

---

24. Phase 21 — Linux Performance Engineering

Duration

4–5 days

Topics

- CPU bottlenecks
- Memory pressure
- Swap
- Load average
- Disk I/O
- Network bottlenecks
- Process bottlenecks
- File descriptors
- Inode exhaustion
- Disk latency
- Cache
- Performance baselines

Practical Labs

Lab 21.1 — CPU Bottleneck

Generate controlled CPU load and identify the process.

Lab 21.2 — Memory Pressure

Investigate:

free
vmstat
top

Lab 21.3 — Disk Bottleneck

Use:

iostat

Lab 21.4 — Network Bottleneck

Capture and analyze traffic.

Project

Linux Performance Investigation Report

Produce a before/after performance report.

---

25. Phase 22 — Linux Internals

Duration

1–2 weeks

This is an advanced phase.

Topics

- Kernel architecture
- User space
- Kernel space
- System calls
- Processes
- Threads
- Scheduling
- Virtual memory
- Page tables
- TLB
- Page cache
- Swap
- VFS
- Inodes
- Dentries
- Drivers
- Interrupts
- Kernel modules
- "/proc"
- "/sys"
- "sysctl"

Practical Labs

Lab 22.1 — System Call Observation

Use tools such as:

strace

to understand how applications interact with the kernel.

Lab 22.2 — Process Investigation

Trace process creation and file/network operations.

Lab 22.3 — Kernel Module Investigation

Inspect:

lsmod
modinfo

Lab 22.4 — Kernel Parameters

Inspect selected:

sysctl

parameters.

Project

Linux System Internals Investigation

Document:

Application
 ↓
Library
 ↓
System Call
 ↓
Kernel Subsystem
 ↓
Hardware

for several real operations such as:

Opening a file
Creating a process
Opening a network connection
Reading system information

---

26. Phase 23 — Containers and Docker

Duration

1–2 weeks

Topics

- Containers vs VMs
- Docker architecture
- Images
- Containers
- Layers
- Namespaces
- PID namespace
- Network namespace
- Mount namespace
- User namespace
- cgroups
- Container networking
- Volumes
- Dockerfile
- Compose
- Container security
- Container logging

Practical Labs

Lab 23.1 — Basic Container

docker run
docker ps
docker exec
docker logs

Lab 23.2 — Docker Networking

Create a custom network.

Lab 23.3 — Persistent Volumes

Deploy a service with persistent storage.

Lab 23.4 — Dockerfile

Containerize a Python application.

Lab 23.5 — Docker Compose

Deploy:

Nginx
FastAPI
Database

Major Project

Containerized Linux Application Platform

Nginx
   ↓
FastAPI
   ↓
PostgreSQL

Include:

- Custom network
- Persistent volumes
- Health checks
- Logs
- Environment variables
- Restart policies
- Security considerations

---

27. Phase 24 — Virtualization

Duration

4–5 days

Topics

- Virtual machines
- Hypervisors
- KVM
- QEMU
- libvirt
- Virtual networking
- Virtual storage
- VM resource allocation
- Linux guest administration

Practical Labs

Lab 24.1

Understand:

Host
 ↓
Hypervisor
 ↓
VM
 ↓
Guest Kernel
 ↓
Applications

Lab 24.2

Explore KVM/libvirt concepts where hardware resources permit.

Lab 24.3

Create virtual network configurations.

Project

Linux Virtual Infrastructure Lab

Document:

VM
CPU
Memory
Storage
Network
Snapshots
Backup

---

28. Phase 25 — Cloud Linux Administration

Duration

1 week

Topics

- Cloud VM
- SSH
- Virtual networking
- Security groups
- Storage
- IAM concepts
- Instance management
- Monitoring
- Cloud-init
- Linux server hardening

The first objective is understanding the Linux administration concepts. Do not make an expensive cloud subscription a requirement.

Practical Labs

Simulate cloud administration locally first:

Virtual Server
 ↓
SSH
 ↓
Firewall
 ↓
Web Server
 ↓
Monitoring
 ↓
Backup

Then optionally reproduce the architecture on a free-tier cloud account.

Project

Cloud-Ready Linux Server

Create a deployment checklist for:

New Server
 ↓
User
 ↓
SSH
 ↓
Firewall
 ↓
Updates
 ↓
Monitoring
 ↓
Application
 ↓
Backup

---

29. Phase 26 — Ansible and Configuration Management

Duration

1 week

Topics

- Configuration management
- Inventory
- YAML
- Playbooks
- Variables
- Tasks
- Handlers
- Templates
- Idempotency
- Roles
- Secrets management concepts

Practical Labs

Automate:

User creation
Package installation
SSH configuration
Nginx installation
Firewall configuration
Directory creation
Service management

Project

Linux Server Provisioning with Ansible

Input:

New Ubuntu Server

Output:

Configured Server
├── Users
├── SSH
├── Firewall
├── Nginx
├── Monitoring
└── Security baseline

---

30. Phase 27 — Cloud/DevOps Integration

Duration

1 week

Topics

- Git
- Linux
- Docker
- CI/CD concepts
- Infrastructure as Code concepts
- Configuration management
- Secrets
- Application deployment
- Reverse proxy
- TLS
- Monitoring

Practical Lab

Create:

GitHub
   ↓
Application
   ↓
Build
   ↓
Test
   ↓
Docker Image
   ↓
Linux Server
   ↓
Nginx

Project

Linux Application Deployment Pipeline

Deploy your Python application using:

Git
Docker
Linux
Nginx
systemd/Compose

---

31. Capstone Project 1 — Enterprise Linux Infrastructure

Objective

Build a complete simulated small-company infrastructure.

                         Linux Infrastructure
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                 │
             DNS              DHCP              SSH
              │                 │                 │
              └─────────────────┼─────────────────┘
                                │
                         Network Services
                                │
                    ┌───────────┴───────────┐
                    │                       │
                  Nginx                  Samba
                    │
                 Python App
                    │
                Database

Include:

- DNS
- DHCP
- SSH
- Firewall
- Nginx
- Python application
- Database
- Users/groups
- Permissions
- Monitoring
- Logs
- Backup

---

32. Capstone Project 2 — Linux Security Engineering Lab

Build:

Ubuntu Server
     │
     ├── User Security
     ├── SSH Hardening
     ├── Firewall
     ├── AppArmor
     ├── Permissions
     ├── Patch Management
     ├── Logging
     ├── Service Audit
     └── Security Baseline

Then perform authorized testing from Kali.

Document:

Before
 ↓
Assessment
 ↓
Hardening
 ↓
Validation
 ↓
After

Do not turn this into uncontrolled scanning of external systems. Keep all security testing inside your own lab or systems for which you have authorization.

---

33. Capstone Project 3 — Linux Incident Response Environment

Create failures deliberately.

Server
 │
 ├── SSH Failure
 ├── DNS Failure
 ├── Nginx Failure
 ├── Disk Full
 ├── Permission Failure
 ├── Network Failure
 ├── High CPU
 ├── High Memory
 ├── Broken Mount
 └── Package Failure

For each:

Detect
 ↓
Investigate
 ↓
Identify Root Cause
 ↓
Recover
 ↓
Verify
 ↓
Document

This should become a major interview demonstration.

---

34. Capstone Project 4 — Linux Automation Platform

Combine Bash and Python.

                    Linux Automation
                           │
       ┌───────────────────┼───────────────────┐
       │                   │                   │
   Monitoring           Backup             Network
       │                   │                   │
       └───────────────────┼───────────────────┘
                           │
                     Alert / Report

Features:

- Server health
- Service health
- Disk monitoring
- Backup
- Network diagnostics
- Log analysis
- User management
- Report generation

---

35. Capstone Project 5 — Production-Like Linux Application Server

Build:

Client
  │
  ▼
Nginx
  │
  ▼
FastAPI / Flask
  │
  ▼
Database

Infrastructure:

systemd
Firewall
SSH
Logging
Monitoring
Backup
Docker

Create both:

Traditional deployment

and:

Docker deployment

Then document the differences.

---

36. Practical Lab Progression

The labs should become progressively harder.

Level 1 — Configure

Install
Configure
Verify

Level 2 — Operate

Monitor
Maintain
Update
Backup

Level 3 — Troubleshoot

Break
Investigate
Repair
Verify

Level 4 — Automate

Script
Schedule
Monitor
Report

Level 5 — Engineer

Design
Deploy
Secure
Monitor
Recover
Document

---

37. Required GitHub Evidence for Every Major Lab

Each important lab should contain:

README.md

with:

1. Objective
2. Environment
3. Architecture
4. Requirements
5. Configuration
6. Commands
7. Expected output
8. Validation
9. Troubleshooting
10. Security considerations
11. Screenshots
12. Lessons learned

For troubleshooting labs additionally include:

Problem
Symptoms
Evidence
Root Cause
Solution
Verification
Prevention

---

38. Recommended GitHub Repository Structure

linux-system-network-engineering/
│
├── README.md
│
├── 00-lab-environment/
│
├── 01-linux-fundamentals/
│
├── 02-filesystem/
│
├── 03-cli-and-shell/
│
├── 04-users-and-groups/
│
├── 05-permissions/
│
├── 06-process-management/
│
├── 07-systemd-and-services/
│
├── 08-package-management/
│
├── 09-linux-networking/
│
├── 10-advanced-networking/
│
├── 11-ssh/
│
├── 12-storage-and-lvm/
│
├── 13-logging-and-monitoring/
│
├── 14-linux-security/
│
├── 15-dns/
│
├── 16-dhcp/
│
├── 17-web-and-network-services/
│
├── 18-bash-automation/
│
├── 19-backup-and-recovery/
│
├── 20-troubleshooting/
│
├── 21-performance/
│
├── 22-linux-internals/
│
├── 23-docker-and-containers/
│
├── 24-virtualization/
│
├── 25-cloud-linux/
│
├── 26-ansible/
│
├── 27-devops-integration/
│
├── labs/
│   ├── beginner/
│   ├── intermediate/
│   ├── advanced/
│   └── troubleshooting/
│
├── projects/
│   ├── linux-inventory/
│   ├── network-diagnostic-tool/
│   ├── enterprise-linux-infrastructure/
│   ├── backup-disaster-recovery/
│   ├── linux-security-hardening/
│   ├── linux-incident-response/
│   ├── linux-automation-toolkit/
│   ├── python-linux-server/
│   └── containerized-application/
│
└── docs/
    ├── architecture/
    ├── troubleshooting/
    ├── security/
    └── runbooks/

---

39. Skill Priority

Not every topic has equal importance for a System & Network Engineer.

Tier 1 — Must Master

Linux Fundamentals
Filesystem
CLI
Bash
Users/Groups
Permissions
Processes
systemd
Packages
Networking
SSH
Storage
Logs
Monitoring
Troubleshooting

Tier 2 — Strong Working Knowledge

Firewall
Linux Security
DNS
DHCP
Nginx
Web Services
Backup
LVM
Performance
Docker

Tier 3 — Advanced

Linux Internals
Namespaces
cgroups
Advanced Networking
Advanced Storage
AppArmor
SELinux
Audit
Virtualization
KVM

Tier 4 — Infrastructure Expansion

Cloud Linux
Ansible
CI/CD
Infrastructure as Code
Container Deployment
Cloud Networking
DevOps

---

40. Final Competency Map

After completing the roadmap, your capability should look like:

                    LINUX ENGINEERING
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   Administration      Networking         Security
        │                  │                  │
   Users               TCP/IP             SSH
   Permissions         Routing            Firewall
   Processes            DNS               AppArmor
   systemd              DHCP              Hardening
   Packages             SSH               Logging
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                    Infrastructure
                           │
             ┌─────────────┼─────────────┐
             │             │             │
          Storage       Services       Backup
             │             │             │
           LVM           Nginx          Rsync
           Filesystems   Apache         Recovery
           RAID          Python         DR
             │             │             │
             └─────────────┼─────────────┘
                           │
                       Automation
                           │
                 ┌─────────┴─────────┐
                 │                   │
                Bash               Python
                 │                   │
                 └─────────┬─────────┘
                           │
                     Modern Linux
                           │
             ┌─────────────┼─────────────┐
             │             │             │
          Docker       Virtualization   Cloud
             │             │             │
          Compose          KVM         Ansible
             │             │             │
             └─────────────┼─────────────┘
                           │
                      Troubleshooting
                           │
                       Engineering

---

41. Final Portfolio Outcome

The roadmap should produce the following practical evidence:

Foundation Projects

1. Linux System Inventory
2. Filesystem Investigation Toolkit
3. Linux Log Analyzer
4. Multi-User Enterprise Server
5. Process Watchdog
6. Service Health Manager

Infrastructure Projects

7. Linux Network Diagnostic Toolkit
8. Secure SSH Administration
9. Enterprise DNS
10. Linux DHCP Infrastructure
11. Linux Storage Administration
12. Nginx/Python Application Server

Security Projects

13. Linux Security Hardening
14. Linux Access Control Lab
15. Linux Security Baseline
16. Linux Incident Response Lab

Automation Projects

17. Linux Administration Toolkit
18. Backup Automation
19. Server Health Monitor
20. Ansible Server Provisioning

Advanced Projects

21. Linux Virtual Network Lab
22. Linux Internals Investigation
23. Containerized Application Platform
24. Linux Virtual Infrastructure
25. Cloud-Ready Linux Server

Flagship Projects

26. Enterprise Linux Infrastructure
27. Linux Backup & Disaster Recovery
28. Linux Security Engineering Lab
29. Linux Incident Response Environment
30. Production-Like Linux Application Platform

---

42. Completion Standard

Do not mark a topic as complete merely because you watched a lesson.

Use this standard:

Understand
    ↓
Perform manually
    ↓
Repeat without notes
    ↓
Break it intentionally
    ↓
Troubleshoot it
    ↓
Automate part of it
    ↓
Document it
    ↓
Add evidence to GitHub

A topic is considered engineer-ready when you can explain it, perform it, troubleshoot it and document it.

---

43. Recommended Learning Sequence

The complete sequence is:

Linux Fundamentals
        ↓
Filesystem
        ↓
CLI / Bash
        ↓
Users / Groups
        ↓
Permissions
        ↓
Processes
        ↓
systemd
        ↓
Packages
        ↓
Networking
        ↓
SSH
        ↓
Storage / LVM
        ↓
Logging / Monitoring
        ↓
Linux Security
        ↓
DNS
        ↓
DHCP
        ↓
Web / Network Services
        ↓
Automation
        ↓
Backup / Recovery
        ↓
Troubleshooting
        ↓
Performance
        ↓
Linux Internals
        ↓
Docker
        ↓
Virtualization
        ↓
Cloud Linux
        ↓
Ansible
        ↓
DevOps

The most important progression for your System & Network Engineer → Security Engineer direction is:

Linux
  ↓
Networking
  ↓
SSH
  ↓
DNS/DHCP
  ↓
Services
  ↓
Security
  ↓
Monitoring
  ↓
Troubleshooting
  ↓
Automation
  ↓
Containers
  ↓
Cloud
  ↓
Security Engineering

This keeps the original GitHub roadmap practical while expanding it into a complete Linux engineering competency map. The original roadmap should remain the core learning sequence, while the advanced phases, labs and capstone projects provide the depth needed beyond basic Linux administration.
