# Practical Linux Server Administration Roadmap

> A practical beginner-to-advanced Linux Server Administration roadmap covering Linux fundamentals, system administration, networking, security, storage, services, automation, troubleshooting, containers, virtualization, and cloud operations.

---

## Roadmap Objective

The goal of this roadmap is not simply to learn Linux commands.

The goal is to become capable of:

* Deploying Linux servers
* Managing users and permissions
* Managing processes and services
* Configuring networking
* Administering SSH
* Managing storage and filesystems
* Installing and maintaining software
* Configuring DNS and DHCP
* Deploying web and application services
* Monitoring servers
* Reading and analyzing logs
* Securing Linux systems
* Automating administration tasks
* Performing backup and recovery
* Troubleshooting infrastructure failures
* Managing containers
* Working with Linux virtualization
* Operating Linux servers in cloud environments

### Target Roles

* Linux System Administrator
* Linux Server Administrator
* System Engineer
* System & Network Engineer
* Infrastructure Engineer
* Linux Support Engineer
* Junior DevOps Engineer
* Cloud Support Engineer
* Security Engineer — Linux/Infrastructure focused

---

# Learning Philosophy

```text
20% Concepts
30% Guided Labs
30% Independent Labs
10% Troubleshooting
10% Documentation
```

The learning cycle is:

```text
Understand
    ↓
Configure
    ↓
Test
    ↓
Break
    ↓
Troubleshoot
    ↓
Automate
    ↓
Secure
    ↓
Document
```

A topic is considered learned only when you can perform the task without following a tutorial step-by-step.

---

# Recommended Lab Environment

## Primary Platform

* Ubuntu Server LTS
* Ubuntu Desktop when GUI comparison is useful
* Kali Linux for authorized security testing
* Windows host
* VMware Workstation / VirtualBox
* Docker
* Git
* GitHub

## Optional

* Ubuntu Server 01
* Ubuntu Server 02
* Ubuntu Server 03
* Kali Linux
* Windows client VM

### Basic Lab

```text
                    Windows Host
                         |
                  Virtual Network
                         |
          +--------------+--------------+
          |              |              |
      Ubuntu 01      Ubuntu 02       Kali
      Server         Server           Linux
```

### Advanced Lab

```text
                         Lab Network
                              |
                +-------------+-------------+
                |             |             |
             Server01      Server02      Server03
                |             |             |
              DNS           Web          Backup
                |
              DHCP
                |
             Clients
```

---

# Phase 0 — Linux Lab Foundation

## Topics

### 0.1 Virtual Machine Environment

* VMware Workstation
* VirtualBox
* Virtual machines
* CPU allocation
* Memory allocation
* Virtual disks
* Virtual NICs
* NAT
* Bridged networking
* Host-only networking
* VM snapshots
* VM cloning
* VM templates
* Resource limitations

### 0.2 Linux Lab Architecture

Understand:

* Server VM
* Client VM
* Management VM
* Testing VM
* Internal network
* Internet access
* DNS
* DHCP
* SSH
* Web services

### Lab 0.1 — Build Ubuntu Server

Tasks:

* Create Ubuntu Server VM
* Configure hostname
* Create administrator
* Configure networking
* Update system
* Install OpenSSH
* Connect remotely
* Create VM snapshot

### Lab 0.2 — Build Multi-VM Network

Create:

```text
Ubuntu Server 01
Ubuntu Server 02
Kali Linux
```

Verify:

* IP connectivity
* SSH connectivity
* DNS resolution
* routing
* VM isolation

### Project

## Linux Home Lab Infrastructure

Document:

* VM architecture
* IP addressing
* hostname scheme
* network topology
* resource allocation
* snapshots
* administration procedure

---

# Phase 1 — Linux Fundamentals

## Topics

### 1.1 Linux Introduction

* What is Linux?
* Linux kernel
* GNU/Linux
* Linux distribution
* Ubuntu
* Debian
* Red Hat
* Fedora
* Arch
* Linux server vs desktop
* Open-source software
* Linux licensing
* Linux use cases

### 1.2 Linux Architecture

* Hardware
* Firmware
* Bootloader
* Kernel
* System calls
* User space
* Kernel space
* Shell
* System utilities
* Services
* Applications

### 1.3 Kernel Concepts

* Monolithic kernel
* Kernel modules
* Drivers
* System calls
* Processes
* Memory management
* Networking
* Filesystems
* Device management
* Security subsystem

### 1.4 Linux Distributions

Understand:

* Debian family
* Red Hat family
* Arch family
* Package management differences
* Filesystem similarities
* Administration differences

### Lab 1.1 — Linux System Identification

Commands:

```bash
uname
uname -a
hostnamectl
cat /etc/os-release
lsb_release -a
```

Collect:

* OS
* kernel
* architecture
* hostname
* uptime

### Lab 1.2 — Linux Architecture Investigation

Investigate:

```text
Firmware
   ↓
Bootloader
   ↓
Kernel
   ↓
systemd
   ↓
Services
   ↓
User applications
```

### Project

## Linux System Inventory Tool

Create a script that reports:

```text
Hostname
OS
Kernel
CPU
Memory
Disk
Network
IP address
Uptime
Logged-in users
Running services
```

---

# Phase 2 — Linux Installation & Boot

## Topics

* Linux installation
* ISO images
* UEFI
* BIOS
* GPT
* MBR
* Bootloader
* GRUB
* initramfs
* kernel loading
* boot targets
* systemd
* rescue mode
* emergency mode
* recovery mode
* kernel parameters
* boot logs

### Lab 2.1 — Linux Installation

Install Ubuntu Server manually.

Document:

* disk layout
* hostname
* users
* network
* SSH
* packages

### Lab 2.2 — Boot Investigation

Investigate:

```bash
systemd-analyze
systemd-analyze blame
journalctl -b
journalctl -k
```

### Lab 2.3 — Boot Failure Recovery

Create controlled failures using VM snapshots.

Practice:

* service failure
* incorrect configuration
* filesystem issue
* boot troubleshooting

### Project

## Linux Boot Troubleshooting Lab

Document at least:

* 3 boot problems
* symptoms
* evidence
* root cause
* recovery
* verification

---

# Phase 3 — Linux Filesystem

## Topics

### 3.1 Filesystem Fundamentals

* Files
* Directories
* Paths
* Absolute paths
* Relative paths
* File types
* Inodes
* Metadata
* Directory entries
* File descriptors

### 3.2 Filesystem Hierarchy

Master:

```text
/
├── boot
├── dev
├── etc
├── home
├── lib
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin
├── srv
├── sys
├── tmp
├── usr
└── var
```

### 3.3 Links

* Hard links
* Symbolic links
* inode relationships
* link counts
* broken symlinks

### 3.4 Virtual Filesystems

* `/proc`
* `/sys`
* `/dev`
* `/run`

### 3.5 File Metadata

* Permissions
* Ownership
* Size
* timestamps
* inode
* file type

### Lab 3.1 — Filesystem Investigation

Investigate:

```bash
ls
stat
file
df
du
find
```

### Lab 3.2 — Inode Investigation

Compare:

* normal file
* hard link
* symbolic link

### Lab 3.3 — `/proc` Investigation

Inspect:

```text
/proc/cpuinfo
/proc/meminfo
/proc/mounts
/proc/uptime
/proc/<PID>/
```

### Project

## Linux Filesystem Investigation Toolkit

Build a script that reports:

* largest files
* largest directories
* inode usage
* mounted filesystems
* recently modified files
* symbolic links
* broken symbolic links

---

# Phase 4 — Linux Command Line

## Topics

* Terminal
* Shell
* Bash
* Commands
* Arguments
* Options
* Environment variables
* PATH
* Command history
* Aliases
* Command substitution
* Quoting
* Wildcards
* Globbing
* stdin
* stdout
* stderr
* redirection
* pipes
* command chaining
* exit codes

### Commands

```bash
ls
cd
pwd
cp
mv
rm
mkdir
touch
cat
less
more
head
tail
grep
find
locate
sort
uniq
cut
tr
wc
xargs
file
stat
```

### Lab 4.1 — Command Pipeline

Build pipelines that:

* search logs
* filter records
* count results
* sort results
* generate reports

### Lab 4.2 — Redirection

Practice:

```bash
>
>>
<
2>
2>>
&
|
```

### Lab 4.3 — Log Analysis

Analyze:

```text
/var/log/
```

### Project

## Linux Log Analyzer

Input:

```text
auth logs
system logs
web logs
```

Output:

```text
Failed logins
Successful logins
Top source IPs
Top users
Error counts
Time distribution
```

---

# Phase 5 — Users & Groups

## Topics

* Users
* Groups
* UID
* GID
* Primary group
* Supplementary groups
* `/etc/passwd`
* `/etc/shadow`
* `/etc/group`
* `/etc/gshadow`
* root
* sudo
* su
* user lifecycle
* password management
* login shell
* home directory
* user environment

### Commands

```bash
useradd
usermod
userdel
passwd
id
groups
who
w
whoami
su
sudo
groupadd
groupmod
groupdel
```

### Lab 5.1 — Company Users

Create:

```text
admin
developer01
developer02
network01
support01
backup01
```

### Lab 5.2 — Department Groups

Create:

```text
developers
network
support
management
backup
```

### Lab 5.3 — User Lifecycle

Practice:

```text
Create
Modify
Lock
Unlock
Expire
Delete
```

### Project

## Linux Multi-User Enterprise Server

Simulate a company environment with departmental users and groups.

---

# Phase 6 — Linux Permissions & Access Control

## Topics

* Read
* Write
* Execute
* Owner
* Group
* Others
* chmod
* chown
* chgrp
* umask
* SUID
* SGID
* Sticky bit
* ACL
* POSIX ACL
* capabilities
* least privilege

### Commands

```bash
chmod
chown
chgrp
umask
getfacl
setfacl
getcap
setcap
```

### Lab 6.1 — Permission Matrix

Create:

```text
Management
Developers
Network
Support
Shared
```

Apply different permissions.

### Lab 6.2 — Shared Directory

Implement:

```text
Department → read/write
Other users → no access
```

### Lab 6.3 — ACL

Create different permissions for individual users.

### Lab 6.4 — SUID/SGID/Sticky Bit

Investigate real-world use cases.

### Project

## Linux Access Control Lab

Document:

* permission model
* ACL model
* privilege boundaries
* security risks
* verification commands

---

# Phase 7 — Processes & Job Management

## Topics

* Program vs process
* PID
* PPID
* Process tree
* Process states
* CPU scheduling
* foreground process
* background process
* jobs
* signals
* daemon
* zombie
* orphan
* process priority
* nice
* renice
* resource consumption

### Commands

```bash
ps
top
htop
pstree
pgrep
pkill
kill
killall
jobs
fg
bg
nice
renice
```

### Lab 7.1 — Process Investigation

Identify:

* PID
* PPID
* CPU usage
* memory usage
* command
* process state

### Lab 7.2 — Signal Management

Practice:

```text
SIGINT
SIGTERM
SIGKILL
SIGSTOP
SIGCONT
```

### Lab 7.3 — High CPU Simulation

Create a controlled CPU workload.

Investigate and resolve it.

### Project

## Linux Process Monitoring Tool

Detect:

* high CPU
* high memory
* long-running processes
* stopped processes
* suspicious processes

---

# Phase 8 — systemd & Service Management

## Topics

* systemd
* PID 1
* units
* service units
* target units
* socket units
* timer units
* mount units
* dependencies
* service states
* startup
* shutdown
* enable
* disable
* restart
* daemon reload
* journal

### Commands

```bash
systemctl
journalctl
systemd-analyze
```

### Lab 8.1 — Service Lifecycle

Practice:

```text
start
stop
restart
enable
disable
mask
unmask
```

### Lab 8.2 — Create Custom Service

Create a systemd service for your own script.

### Lab 8.3 — Service Failure

Break a service configuration and troubleshoot it.

### Project

## Linux Service Health Manager

Monitor:

```text
SSH
Nginx
Docker
DNS
custom application
```

Automatically:

* detect failure
* collect logs
* restart service
* record event
* verify recovery

---

# Phase 9 — Package Management

## Topics

* APT
* dpkg
* repositories
* package metadata
* dependencies
* package installation
* package removal
* package updates
* upgrades
* security updates
* broken packages
* package verification
* package cache
* unattended upgrades
* Snap
* source compilation

### Commands

```bash
apt
apt-cache
apt-mark
dpkg
snap
```

### Lab 9.1 — Package Lifecycle

Install, inspect, update and remove packages.

### Lab 9.2 — Broken Package Recovery

Create a controlled package/configuration problem and recover it.

### Lab 9.3 — Patch Management

Create an update report.

### Project

## Linux Patch Management Tool

Report:

```text
Installed packages
Available updates
Security updates
Reboot required
Last update
```

---

# Phase 10 — Linux Networking Fundamentals

## Topics

* Network interfaces
* Ethernet
* MAC addresses
* IPv4
* IPv6
* subnetting
* gateway
* routing
* ARP
* DNS
* DHCP
* TCP
* UDP
* ports
* sockets
* loopback
* localhost
* network namespaces

### Commands

```bash
ip
ss
ping
tracepath
traceroute
mtr
arp
resolvectl
hostnamectl
```

### Lab 10.1 — Interface Investigation

Identify:

* interface
* MAC
* IP
* subnet
* gateway
* DNS

### Lab 10.2 — Routing Investigation

Inspect:

```bash
ip route
```

### Lab 10.3 — Socket Investigation

Use:

```bash
ss -tulpn
```

### Lab 10.4 — Packet Capture

Use:

```bash
tcpdump
```

and Wireshark from the host.

### Project

## Linux Network Diagnostic Toolkit

Automate:

```text
Interface check
IP check
Gateway check
DNS check
Internet check
Port check
Latency check
Packet loss
Routing check
```

---

# Phase 11 — Linux Network Configuration

## Topics

* Netplan
* static IP
* DHCP
* DNS configuration
* gateway
* routes
* VLAN concepts
* bonding concepts
* bridges
* NetworkManager
* network namespaces
* virtual interfaces
* MTU
* IPv4
* IPv6

### Lab 11.1 — Static Network

Configure static addressing.

### Lab 11.2 — DHCP

Configure DHCP client.

### Lab 11.3 — Network Failure

Break:

* gateway
* DNS
* IP configuration
* route

Troubleshoot each independently.

### Lab 11.4 — Network Namespace

Create isolated network namespaces.

### Project

## Linux Network Configuration Lab

Document:

```text
Network architecture
Addressing
Routing
DNS
Troubleshooting procedure
```

---

# Phase 12 — SSH Remote Administration

## Topics

* OpenSSH
* SSH client
* SSH server
* sshd
* public/private keys
* authorized_keys
* SSH configuration
* host keys
* known_hosts
* password authentication
* key authentication
* SCP
* SFTP
* rsync
* SSH tunneling
* local forwarding
* remote forwarding
* ProxyJump
* agent forwarding
* remote commands

### Lab 12.1 — SSH Server

Configure and test OpenSSH.

### Lab 12.2 — Key Authentication

Disable password authentication in a controlled lab.

### Lab 12.3 — Secure File Transfer

Use:

```bash
scp
sftp
rsync
```

### Lab 12.4 — SSH Troubleshooting

Investigate:

* authentication failure
* connection refused
* timeout
* wrong permissions
* wrong key

### Project

## Secure Linux Remote Administration Lab

Architecture:

```text
Administrator
     |
     +---- SSH ---- Server01
     |
     +---- SSH ---- Server02
     |
     +---- SSH ---- Server03
```

---

# Phase 13 — Linux Storage & Filesystems

## Topics

* HDD
* SSD
* block devices
* partitions
* GPT
* MBR
* filesystem
* ext4
* XFS concepts
* filesystem creation
* mounting
* unmounting
* mount options
* `/etc/fstab`
* UUID
* labels
* disk usage
* inode usage
* filesystem checks
* SMART
* quotas
* swap

### Commands

```bash
lsblk
blkid
fdisk
parted
mkfs
mount
umount
df
du
fsck
```

### Lab 13.1 — Additional Virtual Disk

Attach a virtual disk.

Perform:

```text
Partition
Format
Mount
Unmount
Persistent mount
```

### Lab 13.2 — `/etc/fstab`

Configure persistent storage using UUID.

### Lab 13.3 — Disk Full Simulation

Create a controlled disk-full condition.

Troubleshoot:

```text
disk blocks
inodes
large files
deleted-but-open files
```

### Project

## Linux Storage Administration Lab

---

# Phase 14 — LVM

## Topics

* Physical Volume
* Volume Group
* Logical Volume
* PE
* LE
* filesystem
* LVM snapshots
* extending volumes
* shrinking concepts
* filesystem resizing

### Commands

```bash
pvcreate
pvs
vgcreate
vgs
lvcreate
lvs
lvextend
lvreduce
```

### Lab 14.1 — Build LVM

```text
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
```

### Lab 14.2 — Extend Storage

Increase:

```text
LV
Filesystem
```

### Lab 14.3 — LVM Snapshot

Create and investigate a snapshot.

### Project

## Linux Dynamic Storage Lab

---

# Phase 15 — Swap & Memory Management

## Topics

* RAM
* virtual memory
* swap
* page cache
* buffers
* memory pressure
* OOM
* swap file
* swap partition
* swappiness

### Commands

```bash
free
swapon
swapoff
vmstat
cat /proc/meminfo
```

### Lab

Create a swap file.

Measure:

* memory usage
* swap usage
* system load

### Project

## Linux Memory Troubleshooting Lab

---

# Phase 16 — Linux Monitoring & Observability

## Topics

* CPU monitoring
* memory monitoring
* disk monitoring
* network monitoring
* load average
* process monitoring
* file descriptors
* sockets
* system uptime
* resource utilization

### Commands

```bash
top
htop
free
uptime
vmstat
iostat
sar
lsof
ss
df
du
```

### Lab

Create resource exhaustion scenarios.

Investigate:

```text
CPU
Memory
Disk
Network
Process
```

### Project

## Linux Server Health Monitor

Output:

```text
CPU
RAM
Swap
Disk
Load
Network
Processes
Services
Uptime
```

---

# Phase 17 — Linux Logging

## Topics

* syslog
* journald
* journalctl
* `/var/log`
* authentication logs
* kernel logs
* service logs
* application logs
* log levels
* log rotation
* retention
* centralized logging concepts

### Commands

```bash
journalctl
dmesg
tail
grep
less
```

### Lab

Investigate:

```text
SSH failures
service failures
kernel events
network events
authentication events
```

### Project

## Linux Log Investigation Platform

Create automated reports from:

```text
auth logs
system logs
service logs
kernel logs
web logs
```

---

# Phase 18 — Bash Automation

## Topics

* variables
* input/output
* conditions
* loops
* functions
* arrays
* strings
* command substitution
* positional arguments
* exit codes
* error handling
* logging
* debugging
* traps
* cron
* scheduled tasks

### Lab 18.1 — Bash Fundamentals

Create:

```bash
system-info.sh
disk-check.sh
service-check.sh
backup.sh
user-manager.sh
```

### Lab 18.2 — Cron Automation

Automate:

* backups
* reports
* cleanup
* monitoring

### Project

## Linux Administration Toolkit

```text
linux-admin-toolkit/
├── system-info.sh
├── health-check.sh
├── disk-monitor.sh
├── service-check.sh
├── backup.sh
├── log-analyzer.sh
├── user-manager.sh
└── network-check.sh
```

---

# Phase 19 — Linux Firewall & Security

## Topics

* Security model
* least privilege
* attack surface
* UFW
* nftables
* netfilter
* iptables concepts
* ports
* service exposure
* SSH hardening
* password security
* sudo
* ACL
* capabilities
* SUID
* SGID
* sticky bit
* AppArmor
* audit concepts
* security updates
* file integrity
* security baseline

### Lab 19.1 — UFW

Configure:

```text
SSH
HTTP
HTTPS
DNS
```

### Lab 19.2 — Service Exposure

Identify:

```bash
ss -tulpn
```

Then remove unnecessary exposed services.

### Lab 19.3 — SSH Hardening

Implement:

* key authentication
* restricted users
* firewall
* logging
* configuration validation

### Lab 19.4 — AppArmor

Inspect:

```bash
aa-status
```

### Project

# Linux Server Security Hardening

Create:

```text
Before
  ↓
Security Assessment
  ↓
Hardening
  ↓
Verification
  ↓
After
```

Document:

* attack surface
* exposed ports
* users
* permissions
* SSH
* firewall
* AppArmor
* updates
* logs

---

# Phase 20 — DNS Administration

## Topics

* DNS architecture
* resolver
* authoritative DNS
* recursive DNS
* caching
* zones
* forward zones
* reverse zones
* A
* AAAA
* CNAME
* MX
* TXT
* NS
* PTR
* TTL
* DNS troubleshooting
* BIND
* DNSSEC concepts

### Commands

```bash
dig
nslookup
host
resolvectl
```

### Lab 20.1 — Local DNS

Create:

```text
server01.lab.local
server02.lab.local
web.lab.local
```

### Lab 20.2 — Reverse DNS

Implement PTR records.

### Lab 20.3 — DNS Troubleshooting

Break:

* zone
* record
* resolver
* service

Diagnose each.

### Project

## Linux DNS Infrastructure

---

# Phase 21 — DHCP Administration

## Topics

* DHCP
* DORA process
* scopes
* leases
* reservations
* subnet
* gateway
* DNS
* hostname assignment
* DHCP troubleshooting
* DHCP logging

### Lab

Build:

```text
DHCP Server
     |
     +---- Client01
     +---- Client02
     +---- Client03
```

### Project

## Linux DHCP Infrastructure

Integrate:

```text
DHCP
+
DNS
+
SSH
+
Firewall
```

---

# Phase 22 — Web Server Administration

## Topics

### Apache

* installation
* configuration
* virtual hosts
* document root
* permissions
* logs
* modules

### Nginx

* installation
* server blocks
* static content
* reverse proxy
* upstream
* logs
* TLS
* headers
* access control

### Lab 22.1 — Apache

Deploy a static website.

### Lab 22.2 — Nginx

Deploy:

```text
Nginx
 ↓
Static Website
```

### Lab 22.3 — Reverse Proxy

Deploy:

```text
Nginx
  ↓
Flask/FastAPI
```

### Project

## Linux Web Server Platform

---

# Phase 23 — Application Server Administration

## Topics

* application processes
* WSGI
* ASGI
* Gunicorn
* Uvicorn
* systemd
* environment variables
* application logs
* reverse proxy
* database connection
* service restart
* health checks

### Major Project

# Production-Like Python Application Server

Architecture:

```text
Client
  |
  v
Nginx
  |
  v
Gunicorn/Uvicorn
  |
  v
Flask/FastAPI
  |
  v
Database
```

Add:

* systemd
* firewall
* logging
* backup
* health endpoint
* monitoring
* restart policy

---

# Phase 24 — File Sharing & Network Services

## Topics

### NFS

* NFS server
* NFS client
* exports
* mounts
* permissions
* troubleshooting

### Samba

* SMB
* Windows interoperability
* shares
* users
* permissions
* configuration
* troubleshooting

### Other Services

* SFTP
* rsync
* FTP concepts
* network shares

### Lab

Create:

```text
Linux Server
     |
     +---- NFS ---- Linux Client
     |
     +---- SMB ---- Windows Client
```

### Project

## Linux File Server

Implement:

* departmental shares
* permissions
* access control
* backup
* logging

---

# Phase 25 — Backup & Recovery

## Topics

* backup strategy
* full backup
* incremental backup
* differential backup
* local backup
* remote backup
* rsync
* tar
* compression
* retention
* checksums
* backup verification
* restore
* disaster recovery
* recovery testing
* configuration backup

### Commands

```bash
tar
rsync
sha256sum
```

### Lab 25.1 — Local Backup

### Lab 25.2 — Remote Backup

### Lab 25.3 — Automated Backup

### Lab 25.4 — Restore

Delete test data.

Restore it.

### Major Project

# Linux Backup & Disaster Recovery System

```text
Production
    |
    v
Backup Automation
    |
    v
Backup Server
    |
    v
Verification
    |
    v
Restore Test
```

---

# Phase 26 — Linux Troubleshooting

## Topics

* troubleshooting methodology
* evidence collection
* hypothesis
* layer-based troubleshooting
* boot failures
* service failures
* permission failures
* storage failures
* DNS failures
* network failures
* SSH failures
* web failures
* CPU problems
* memory problems
* disk problems
* configuration problems

### Troubleshooting Model

```text
Identify
   ↓
Observe
   ↓
Collect evidence
   ↓
Determine affected layer
   ↓
Form hypothesis
   ↓
Test
   ↓
Fix
   ↓
Verify
   ↓
Document
```

### Labs

Create controlled failures:

1. SSH failure
2. DNS failure
3. DHCP failure
4. Nginx failure
5. permission failure
6. full disk
7. broken mount
8. high CPU
9. high memory
10. broken network route
11. wrong DNS
12. failed systemd service

### Major Project

# Linux Incident Response Lab

For every incident record:

```text
Incident ID
Symptoms
Impact
Evidence
Commands
Root Cause
Resolution
Verification
Prevention
```

---

# Phase 27 — Linux Performance Engineering

## Topics

* CPU bottlenecks
* memory pressure
* swap
* disk I/O
* network bottlenecks
* load average
* process analysis
* file descriptors
* socket exhaustion
* inode exhaustion
* disk latency
* service latency

### Lab

Simulate:

```text
High CPU
High RAM
Disk I/O
Full disk
High network activity
Too many processes
```

### Project

## Linux Performance Investigation Lab

Produce:

```text
Performance Baseline
Problem
Metrics
Root Cause
Optimization
Before/After
```

---

# Phase 28 — Linux Containers

## Topics

* containers
* images
* Docker
* Dockerfile
* Docker Compose
* container lifecycle
* namespaces
* cgroups
* overlay filesystem
* container networking
* volumes
* port mapping
* container logs
* health checks
* restart policies
* container security

### Commands

```bash
docker pull
docker run
docker ps
docker exec
docker logs
docker inspect
docker stop
docker start
docker rm
```

### Lab 28.1 — Basic Container

### Lab 28.2 — Custom Image

### Lab 28.3 — Volumes

### Lab 28.4 — Networks

### Lab 28.5 — Compose

### Major Project

# Containerized Linux Service Platform

```text
Nginx
  |
  v
FastAPI
  |
  v
Database
```

Using:

```text
Docker Compose
```

Implement:

* custom network
* volumes
* environment configuration
* health checks
* logs
* restart policies

---

# Phase 29 — Linux Internals

## Topics

* user space
* kernel space
* system calls
* processes
* threads
* scheduling
* virtual memory
* page cache
* VFS
* inode
* dentry
* device drivers
* interrupts
* kernel modules
* `/proc`
* `/sys`
* sysctl

### Lab

Trace common operations:

```text
ls
cat
ping
ssh
```

Understand:

```text
Application
    ↓
Library
    ↓
System Call
    ↓
Kernel
    ↓
Hardware
```

### Project

## Linux System Call Investigation

Document system behavior using tracing tools such as:

```bash
strace
```

---

# Phase 30 — Advanced Linux Networking

## Topics

* network namespaces
* bridges
* virtual Ethernet
* VLAN
* bonding
* routing
* policy routing
* NAT
* forwarding
* firewall chains
* nftables
* packet filtering
* packet capture
* tcpdump
* connection tracking

### Lab

Build:

```text
Namespace A
     |
   Bridge
     |
Namespace B
```

### Lab

Configure:

* virtual interfaces
* routing
* NAT
* firewall rules

### Project

## Linux Virtual Network Lab

Document the packet path:

```text
Application
 ↓
Socket
 ↓
Network Namespace
 ↓
Interface
 ↓
Bridge
 ↓
Firewall
 ↓
Routing
 ↓
Physical/Virtual NIC
```

---

# Phase 31 — Virtualization

## Topics

* virtualization
* hypervisor
* KVM
* QEMU
* libvirt
* virtual CPU
* virtual memory
* virtual disk
* virtual network
* VM lifecycle
* snapshots
* resource allocation

### Lab

Learn:

```text
KVM
QEMU
libvirt
virsh
```

### Project

## Linux Virtualization Lab

Create and manage:

```text
VM
Network
Storage
Snapshot
Resource allocation
```

---

# Phase 32 — Cloud Linux Administration

## Topics

* cloud VM
* SSH
* cloud networking
* security groups
* IAM concepts
* block storage
* object storage
* DNS
* monitoring
* backups
* cloud firewall
* cloud-init

### Lab

Deploy a Linux VM in a free/low-cost cloud environment when available.

Practice:

* SSH
* firewall
* web server
* logs
* monitoring
* backup

### Project

## Cloud Linux Server Deployment

Document:

```text
Architecture
Network
Security
Server configuration
Monitoring
Backup
Recovery
```

---

# Phase 33 — Configuration Management

## Topics

* configuration drift
* infrastructure automation
* Ansible
* inventory
* playbooks
* variables
* templates
* handlers
* roles
* idempotency
* secrets

### Lab

Use Ansible to configure:

```text
Server01
Server02
Server03
```

Automate:

* users
* SSH
* packages
* firewall
* Nginx
* monitoring

### Project

## Linux Server Provisioning with Ansible

Input:

```text
Inventory
```

Output:

```text
Configured Linux Servers
```

---

# Phase 34 — Linux Infrastructure Automation

Combine:

```text
Bash
+
Python
+
Ansible
+
SSH
+
systemd
+
Docker
```

### Project

# Linux Infrastructure Automation Platform

Features:

* server inventory
* health checks
* remote command execution
* service status
* package status
* disk monitoring
* backup status
* log collection
* configuration checks

---

# Phase 35 — Capstone Enterprise Linux Environment

This is the final practical project.

## Project

# Enterprise Linux Infrastructure Lab

Build:

```text
                         Internet
                            |
                         Firewall
                            |
                     Linux Gateway
                            |
        +-------------------+-------------------+
        |                   |                   |
      DNS                 DHCP                SSH
        |                   |                   |
        +-------------------+-------------------+
                            |
             +--------------+--------------+
             |              |              |
           Web            App           File
          Server         Server         Server
             |              |              |
             +--------------+--------------+
                            |
                       Monitoring
                            |
                         Backup
```

### Required Services

* DNS
* DHCP
* SSH
* Nginx
* Python application
* File sharing
* Firewall
* Monitoring
* Logging
* Backup
* Docker

### Required Administration

* users
* groups
* permissions
* ACLs
* services
* storage
* LVM
* networking
* patching
* security

### Required Automation

* Bash
* Python
* Ansible

### Required Documentation

```text
Architecture
IP Address Plan
Server Inventory
Installation
Configuration
Security
Monitoring
Backup
Troubleshooting
Disaster Recovery
```

---

# Practical Project Matrix

| Phase | Project                      | Main Skills              |
| ----- | ---------------------------- | ------------------------ |
| 0     | Linux Home Lab               | VMware, networking       |
| 1     | System Inventory             | Linux fundamentals, Bash |
| 2     | Boot Troubleshooting Lab     | boot, systemd            |
| 3     | Filesystem Toolkit           | filesystem, storage      |
| 4     | Log Analyzer                 | CLI, grep, awk, logs     |
| 5     | Multi-User Server            | users, groups            |
| 6     | Access Control Lab           | permissions, ACL         |
| 7     | Process Monitor              | processes, signals       |
| 8     | Service Health Manager       | systemd                  |
| 9     | Patch Management             | APT, updates             |
| 10    | Network Diagnostic Toolkit   | Linux networking         |
| 11    | Network Configuration Lab    | Netplan, routing         |
| 12    | Secure Remote Administration | SSH                      |
| 13    | Storage Administration       | filesystems, mounts      |
| 14    | Dynamic Storage Lab          | LVM                      |
| 15    | Memory Troubleshooting       | RAM, swap                |
| 16    | Server Health Monitor        | monitoring               |
| 17    | Log Investigation            | journald, syslog         |
| 18    | Admin Toolkit                | Bash automation          |
| 19    | Server Hardening             | firewall, AppArmor       |
| 20    | DNS Infrastructure           | BIND, DNS                |
| 21    | DHCP Infrastructure          | DHCP                     |
| 22    | Web Server                   | Nginx/Apache             |
| 23    | Python App Server            | Nginx, systemd, Python   |
| 24    | Linux File Server            | NFS/Samba                |
| 25    | Backup & DR                  | rsync, restore           |
| 26    | Incident Response            | troubleshooting          |
| 27    | Performance Lab              | performance              |
| 28    | Container Platform           | Docker                   |
| 29    | Linux Internals              | kernel, syscalls         |
| 30    | Virtual Network Lab          | namespaces, routing      |
| 31    | Virtualization Lab           | KVM/QEMU                 |
| 32    | Cloud Linux Server           | cloud                    |
| 33    | Ansible Provisioning         | configuration management |
| 34    | Infrastructure Automation    | Bash/Python/Ansible      |
| 35    | Enterprise Capstone          | complete administration  |

---

# Flagship Portfolio Projects

The following projects should receive the highest documentation quality.

## Project 1 — Enterprise Linux Network Infrastructure

```text
DNS
DHCP
SSH
Firewall
Nginx
Routing
Monitoring
```

---

## Project 2 — Linux Backup & Disaster Recovery

```text
Backup
Rsync
Automation
Retention
Verification
Restore
Recovery
```

---

## Project 3 — Production-Like Python Application Server

```text
Nginx
FastAPI/Flask
Gunicorn/Uvicorn
systemd
Database
Firewall
Logging
Backup
```

---

## Project 4 — Linux Security Hardening

```text
Users
Permissions
SSH
Firewall
AppArmor
Updates
Logging
Service exposure
```

---

## Project 5 — Linux Incident Response

```text
10–15 controlled failures
        ↓
Investigation
        ↓
Root Cause
        ↓
Recovery
        ↓
Verification
        ↓
Documentation
```

---

## Project 6 — Linux Container Platform

```text
Nginx
FastAPI
Database
Docker
Compose
Volumes
Networks
Health Checks
```

---

## Project 7 — Enterprise Linux Capstone

Combine everything into one infrastructure environment.

---

# Lab Difficulty System

Every lab should be classified as:

### Level 1 — Guided

Follow documented instructions.

```text
Step → Command → Result
```

### Level 2 — Semi-Guided

Given:

```text
Goal
Requirements
Expected result
```

You determine the commands.

### Level 3 — Independent

Only the problem is provided.

Example:

> Users cannot connect to the web server.

You determine:

* what to inspect
* what evidence to collect
* root cause
* solution

### Level 4 — Incident

The environment is deliberately broken.

You receive only:

```text
INCIDENT-014

Web server unavailable.
```

You must investigate everything.

---

# Required Evidence for Every Major Lab

Each GitHub lab should contain:

```text
README.md
```

with:

```text
1. Objective
2. Environment
3. Architecture
4. Requirements
5. Configuration
6. Commands
7. Expected Results
8. Verification
9. Troubleshooting
10. Security Considerations
11. Screenshots
12. Lessons Learned
```

---

# GitHub Repository Structure

Recommended repository:

```text
linux-server-administration/
│
├── README.md
│
├── 00-lab-environment/
│
├── 01-linux-fundamentals/
│
├── 02-installation-and-boot/
│
├── 03-filesystem/
│
├── 04-command-line/
│
├── 05-users-and-groups/
│
├── 06-permissions/
│
├── 07-process-management/
│
├── 08-systemd/
│
├── 09-package-management/
│
├── 10-linux-networking/
│
├── 11-network-configuration/
│
├── 12-ssh/
│
├── 13-storage/
│
├── 14-lvm/
│
├── 15-memory-and-swap/
│
├── 16-monitoring/
│
├── 17-logging/
│
├── 18-bash-automation/
│
├── 19-linux-security/
│
├── 20-dns/
│
├── 21-dhcp/
│
├── 22-web-server/
│
├── 23-application-server/
│
├── 24-file-services/
│
├── 25-backup-recovery/
│
├── 26-troubleshooting/
│
├── 27-performance/
│
├── 28-docker/
│
├── 29-linux-internals/
│
├── 30-advanced-networking/
│
├── 31-virtualization/
│
├── 32-cloud-linux/
│
├── 33-ansible/
│
├── 34-infrastructure-automation/
│
└── projects/
    │
    ├── enterprise-linux-network/
    ├── backup-disaster-recovery/
    ├── python-application-server/
    ├── linux-security-hardening/
    ├── linux-incident-response/
    ├── container-platform/
    └── enterprise-linux-capstone/
```

---

# Final Competency Checklist

## Linux Fundamentals

* [ ] Linux architecture
* [ ] Kernel
* [ ] Distribution
* [ ] Boot process
* [ ] systemd
* [ ] filesystem hierarchy

## CLI

* [ ] Bash
* [ ] pipes
* [ ] redirection
* [ ] grep
* [ ] find
* [ ] sed
* [ ] awk
* [ ] xargs
* [ ] command substitution
* [ ] exit codes

## System Administration

* [ ] users
* [ ] groups
* [ ] permissions
* [ ] ACL
* [ ] processes
* [ ] services
* [ ] packages
* [ ] logs
* [ ] monitoring

## Networking

* [ ] IP addressing
* [ ] routing
* [ ] DNS
* [ ] DHCP
* [ ] TCP/UDP
* [ ] ports
* [ ] sockets
* [ ] SSH
* [ ] tcpdump
* [ ] network troubleshooting

## Storage

* [ ] partitions
* [ ] filesystems
* [ ] mounting
* [ ] fstab
* [ ] LVM
* [ ] swap
* [ ] disk troubleshooting
* [ ] storage monitoring

## Security

* [ ] least privilege
* [ ] sudo
* [ ] permissions
* [ ] ACL
* [ ] capabilities
* [ ] SSH hardening
* [ ] firewall
* [ ] AppArmor
* [ ] patching
* [ ] logging
* [ ] security baseline

## Services

* [ ] DNS
* [ ] DHCP
* [ ] Nginx
* [ ] Apache
* [ ] SSH
* [ ] NFS
* [ ] Samba
* [ ] Python application server

## Automation

* [ ] Bash
* [ ] cron
* [ ] Python
* [ ] SSH automation
* [ ] Ansible

## Reliability

* [ ] monitoring
* [ ] logging
* [ ] backup
* [ ] restore
* [ ] disaster recovery
* [ ] troubleshooting
* [ ] performance analysis

## Modern Infrastructure

* [ ] Docker
* [ ] Docker Compose
* [ ] containers
* [ ] namespaces
* [ ] cgroups
* [ ] KVM
* [ ] QEMU
* [ ] cloud Linux

---

# Final Learning Progression

```text
Linux Fundamentals
        ↓
Filesystem + CLI
        ↓
Users + Permissions
        ↓
Processes + systemd
        ↓
Package Management
        ↓
Networking
        ↓
SSH
        ↓
Storage + LVM
        ↓
Monitoring + Logging
        ↓
Bash Automation
        ↓
Firewall + Security
        ↓
DNS + DHCP
        ↓
Web + Application Services
        ↓
Backup + Recovery
        ↓
Troubleshooting
        ↓
Performance
        ↓
Docker
        ↓
Linux Internals
        ↓
Advanced Networking
        ↓
Virtualization
        ↓
Cloud Linux
        ↓
Ansible
        ↓
Infrastructure Automation
        ↓
Enterprise Linux Capstone
```

---

# The Real Completion Criteria

Do not mark a topic complete because you watched a video.

Mark it complete when you can:

```text
Explain it
    ↓
Configure it
    ↓
Verify it
    ↓
Break it
    ↓
Troubleshoot it
    ↓
Secure it
    ↓
Automate it
    ↓
Document it
```

The final objective is:

> **Deploy → Configure → Secure → Monitor → Troubleshoot → Automate → Recover → Document a Linux server independently.**

That is the practical competency this roadmap is designed to build.
