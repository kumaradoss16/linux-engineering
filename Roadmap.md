# Practical Linux Server Administration Roadmap

> A command-driven, hands-on roadmap for learning Linux server setup, configuration, administration, networking, security, services, automation, monitoring, troubleshooting, containers, virtualization, and cloud operations.

Reference: this roadmap expands the phase sequence and projects in this repository into practical setup/configuration tasks, commands, verification steps, and troubleshooting exercises.

## How to use this roadmap

For every task, follow this cycle:

1. **Prepare** the VM or service and record its initial state.
2. **Configure** one change at a time.
3. **Run commands** and capture relevant output.
4. **Verify** expected behavior from the client side where possible.
5. **Troubleshoot** using evidence rather than guessing.
6. **Secure** the configuration and remove unnecessary exposure.
7. **Document** the commands, results, cause of any failure, and recovery.

Do not mark a topic complete after reading it. Complete the lab, verify the result, explain the configuration, and document a failure-and-recovery exercise.

## Learning method

- 20% concepts
- 30% guided labs
- 30% independent labs
- 10% troubleshooting
- 10% documentation

## Target roles

- Linux System Administrator / Linux Server Administrator
- System Engineer / System & Network Engineer
- Infrastructure Engineer / Linux Support Engineer
- Junior DevOps Engineer / Cloud Support Engineer
- Security Engineer focused on Linux and infrastructure

## Lab environment

### Recommended low-cost layout

Start with one Ubuntu Server LTS VM. Add a second Linux VM when practicing client/server networking. A third VM is optional for DNS, backup, or security testing.

| VM | Purpose | Suggested resources |
|---|---|---|
| Ubuntu Server 01 | Main server and administration target | 2 vCPU, 2–3 GB RAM, 25–40 GB disk |
| Ubuntu Server 02 | Client or second server | 1–2 vCPU, 1–2 GB RAM, 20 GB disk |
| Kali Linux or Ubuntu Server 03 | Authorized testing, backup, or service client | 1–2 vCPU, 1–2 GB RAM |

On an 8 GB host, run only the VMs needed for the current lab. Use VMware Workstation or VirtualBox. KVM/libvirt and Proxmox are optional later platforms.

### Network modes

- **NAT:** best default for VM internet access through the host.
- **Host-only:** isolated host-to-VM practice network; usually no internet by itself.
- **Bridged:** places the VM on the physical LAN; use only when appropriate for that network.
- **Internal/private network:** isolates lab VMs from the host LAN, depending on hypervisor configuration.

Record the VM's IP, prefix, gateway, DNS server, adapter mode, and subnet before troubleshooting.

### Initial Ubuntu setup

Run these commands on a newly installed Ubuntu Server VM:

~~~bash
hostnamectl
cat /etc/os-release
ip -br address
ip route
resolvectl status
df -hT
free -h
sudo apt update
sudo apt full-upgrade
sudo apt install -y openssh-server curl wget vim nano git tree dnsutils traceroute tcpdump lsof
sudo systemctl enable --now ssh
systemctl status ssh --no-pager
~~~

Confirm that internet access works before installing packages. If package installation fails, investigate the IP address, default route, gateway reachability, DNS resolution, and hypervisor NAT/bridged configuration in that order.

Create a VM snapshot after a clean, updated baseline. Take additional snapshots before risky boot, storage, firewall, and network labs. Never use a production system as a failure-injection target.

---

# Phase 0 — Linux Lab Foundation

**Goal:** Build a repeatable and recoverable environment before learning administration.

## Setup and configuration

1. Create an Ubuntu Server VM with a hostname such as `server01`.
2. Choose NAT for initial internet access.
3. Record CPU, RAM, disk, adapter mode, IP address, gateway, and DNS.
4. Create a second VM later for client/server practice.
5. Take a clean snapshot before changing network or storage settings.

## Commands

~~~bash
hostnamectl
ip -br link
ip -br address
ip route
resolvectl status
ping -c 3 1.1.1.1
getent hosts ubuntu.com
lsblk
df -hT
free -h
~~~

## Labs

- **Lab 0.1 — Build Ubuntu Server:** install Ubuntu Server, set hostname, create an administrator, update packages, enable SSH, and snapshot the VM.
- **Lab 0.2 — Verify internet access:** test interface state, IP, gateway, public IP reachability, and DNS separately. Record which layer fails if internet access does not work.
- **Lab 0.3 — Build a multi-VM network:** create a second VM, document the subnet, and verify connectivity only after checking that the VMs share a reachable network.

## Verification and evidence

- [ ] VM starts and shuts down cleanly.
- [ ] Hostname and OS version are documented.
- [ ] Default route and DNS configuration are recorded.
- [ ] Package updates complete successfully.
- [ ] SSH service is running.
- [ ] Baseline snapshot exists.

**Project:** Linux Home Lab Infrastructure — document VM layout, IP plan, hypervisor network mode, snapshots, and recovery procedure.

---

# Phase 1 — Linux Fundamentals

**Goal:** Understand the operating system and identify the machine accurately.

## Commands

~~~bash
uname -a
hostnamectl
cat /etc/os-release
lscpu
free -h
lsblk
uptime
who
systemctl --type=service --state=running
~~~

## Practical tasks

1. Identify the distribution, kernel, architecture, CPU, memory, disks, uptime, and running services.
2. Trace the boot path conceptually: firmware → bootloader → kernel/initramfs → PID 1/systemd → services.
3. Compare a kernel, a distribution, system utilities, services, and applications.
4. Review the distribution family and package manager before using installation instructions.

## Lab and verification

Build a system inventory report from real command output. Re-run it after a reboot and identify which values changed.

**Project:** Linux System Inventory Tool — report hostname, OS, kernel, CPU, memory, disk, network, uptime, users, and services. Avoid collecting secrets.

---

# Phase 2 — Linux Installation and Boot

**Goal:** Understand installation choices and diagnose startup problems safely.

## Commands

~~~bash
lsblk -f
findmnt
systemctl get-default
systemd-analyze
systemd-analyze blame
journalctl -b
journalctl -k -b
systemctl --failed
~~~

## Practical tasks

1. Record the VM disk layout before installation.
2. Identify UEFI/BIOS mode, partition scheme, bootloader, initramfs, kernel, and systemd.
3. Compare normal boot with rescue or emergency mode using documentation and a disposable VM.
4. Inspect boot logs after a controlled service failure.

## Labs

- **Lab 2.1:** Install Ubuntu Server and document storage, hostname, user, networking, SSH, and package choices.
- **Lab 2.2:** Inspect the current boot using `journalctl -b`, `journalctl -k -b`, and `systemd-analyze`.
- **Lab 2.3:** Disable a noncritical test service, reboot, identify the failure, restore the service, and verify recovery.

**Safety:** Do not experiment with bootloader, partition, or root filesystem changes without a snapshot and a recovery path.

**Project:** Linux Boot Troubleshooting Lab — document three symptoms, evidence, root cause, recovery, and verification steps.

---

# Phase 3 — Linux Filesystem

**Goal:** Investigate paths, inodes, metadata, links, mount points, and virtual filesystems.

## Commands

~~~bash
pwd
ls -la /
ls -li
stat /etc/hostname
file /etc/hostname
findmnt
df -hT
df -i
du -sh "$HOME"
find "$HOME" -maxdepth 2 -type f
readlink -f /bin/sh
ls -ld /proc /sys /dev /run
~~~

## Practical tasks

1. Explain the purpose of `/boot`, `/dev`, `/etc`, `/home`, `/lib`, `/media`, `/mnt`, `/opt`, `/proc`, `/root`, `/run`, `/sbin`, `/srv`, `/sys`, `/tmp`, `/usr`, and `/var`.
2. Create a test file, hard link, and symbolic link in a directory under your home directory.
3. Compare inode numbers, link counts, ownership, permissions, and timestamps using `ls -li` and `stat`.
4. Inspect `/proc/cpuinfo`, `/proc/meminfo`, `/proc/mounts`, `/proc/uptime`, and a live `/proc/PID/` directory.
5. Distinguish disk-backed files from virtual filesystem interfaces.

## Labs

- **Lab 3.1 — Filesystem investigation:** use `ls`, `stat`, `file`, `df`, `du`, and `find` to investigate a test directory.
- **Lab 3.2 — Inodes and links:** create normal, hard-linked, and symbolic-linked names; remove one name and verify the remaining links.
- **Lab 3.3 — /proc investigation:** inspect CPU, memory, mounts, uptime, and a running process.

**Project:** Linux Filesystem Investigation Toolkit — report mount points, capacity, inode usage, largest files/directories, file metadata, and broken symlinks.

---

# Phase 4 — Linux Command Line

**Goal:** Use Bash and standard utilities to inspect, filter, and transform system information.

## Commands

~~~bash
pwd
ls -lah
mkdir -p ~/lab-cli
touch ~/lab-cli/example.txt
cp
mv
rm
cat
less
head
tail
grep
find
sort
uniq
cut
tr
wc
xargs
file
stat
~~~

Use destructive commands only with an explicit test path; do not copy a bare `rm -rf` example into a live shell.

## Practical tasks

1. Practice arguments, options, environment variables, `PATH`, quoting, globbing, command substitution, and exit codes.
2. Redirect standard output and errors separately.
3. Build pipelines that search a log, sort results, count repeated values, and save a report.
4. Use `grep`, `sed`, and `awk` to extract useful fields from sample text.
5. Explain the difference between shell built-ins and external commands.

## Labs

- **Lab 4.1 — Command pipeline:** filter and count records from a test log.
- **Lab 4.2 — Redirection:** send output and errors to separate files, then verify each.
- **Lab 4.3 — Log analysis:** inspect permitted files under `/var/log` and explain any permission limits.

**Project:** Linux Log Analyzer — summarize error counts, failed login events, and timestamps from available logs without exposing credentials or tokens.

---

# Phase 5 — Users and Groups

**Goal:** Manage accounts and group membership using least privilege.

## Commands

~~~bash
whoami
id
groups
getent passwd
getent group
sudo adduser developer01
sudo groupadd developers
sudo usermod -aG developers developer01
sudo passwd -l developer01
sudo passwd -u developer01
sudo chage -l developer01
sudo userdel -r developer01
~~~

Use a disposable lab account. Do not lock or delete your only administrator account.

## Practical tasks

1. Inspect account records with `getent passwd`, `getent group`, and `id`.
2. Create department groups and test users.
3. Add users to supplementary groups using `usermod -aG`; verify membership with `id`.
4. Practice account expiry and locking on test accounts.
5. Review sudo access and login shells.

## Labs

- **Lab 5.1 — Company users:** create test accounts for developers, network, support, and backup roles.
- **Lab 5.2 — Department groups:** create groups and assign test users.
- **Lab 5.3 — User lifecycle:** create, modify, lock, unlock, expire, and remove a test account.

**Project:** Linux Multi-User Server — document the account matrix, group membership, sudo policy, and account removal procedure.

---

# Phase 6 — Linux Permissions and Access Control

**Goal:** Configure and verify file access using Unix permissions, ACLs, and capabilities.

## Commands

~~~bash
umask
ls -l
chmod 640 file.txt
chown user:group file.txt
chgrp group file.txt
getfacl file.txt
setfacl -m u:developer01:r-- file.txt
getcap /usr/bin/ping
~~~

Run examples against files and directories created under your lab directory.

## Practical tasks

1. Translate symbolic permissions and numeric modes.
2. Configure a shared directory with group ownership and setgid where appropriate.
3. Use ACLs to grant a specific test user access without broadening access for everyone.
4. Inspect SUID, SGID, sticky bit, and Linux capabilities.
5. Verify access by switching to a test user or using a controlled command such as `sudo -u developer01 -- test -r path`.

## Labs

- **Lab 6.1:** Build a permission matrix for management, developers, network, support, and shared directories.
- **Lab 6.2:** Configure department-only read/write access.
- **Lab 6.3:** Apply and verify a user-specific ACL.
- **Lab 6.4:** Inspect special permission bits and explain their security implications.

**Security rule:** Avoid recursive ownership or permission changes on system directories.

**Project:** Linux Access Control Lab — document expected access, actual test results, privilege boundaries, and remediation.

---

# Phase 7 — Processes and Job Management

**Goal:** Inspect process state and safely control workloads.

## Commands

~~~bash
ps -eo pid,ppid,user,stat,%cpu,%mem,cmd --sort=-%cpu | head
top
pgrep -a ssh
pstree -p
jobs
sleep 300 &
jobs
kill -TERM %1
nice -n 10 sleep 60 &
pgrep -a sleep
~~~

Use `htop` only if installed. Confirm a PID before sending a signal.

## Practical tasks

1. Identify PID, PPID, state, owner, CPU, and memory.
2. Compare foreground/background jobs and shell job control.
3. Explain SIGINT, SIGTERM, SIGKILL, SIGSTOP, and SIGCONT.
4. Inspect zombies and orphaned processes from observation or a controlled example.
5. Use `nice` and `renice` on a noncritical test process.

## Labs

- **Lab 7.1:** Record process metadata and explain the process tree.
- **Lab 7.2:** Start a test process in the background and terminate it gracefully.
- **Lab 7.3:** Create a controlled workload, observe resource use, and stop it safely.

**Project:** Linux Process Monitor — report top CPU/memory processes, process state, and timestamped observations.

---

# Phase 8 — systemd and Service Management

**Goal:** Manage services, units, startup behavior, and service failures.

## Commands

~~~bash
systemctl status ssh --no-pager
systemctl is-enabled ssh
systemctl list-units --type=service --state=running
systemctl --failed
journalctl -u ssh --since today
systemctl cat ssh
systemctl get-default
systemd-analyze critical-chain
~~~

## Practical tasks

1. Inspect service, target, socket, timer, and mount units.
2. Distinguish start/stop from enable/disable.
3. Inspect service dependencies and recent logs.
4. Create a simple custom service for a harmless script in a lab VM.
5. Validate unit syntax and reload systemd after editing a unit.

## Labs

- **Lab 8.1:** Practice the lifecycle of a noncritical service.
- **Lab 8.2:** Create a custom systemd service with a dedicated script and log output.
- **Lab 8.3:** Introduce a reversible error in the test service, diagnose it with `systemctl status` and `journalctl`, and restore it.

**Project:** Linux Service Health Manager — check selected services, collect logs on failure, and record recovery evidence. Do not automatically restart every failed service without considering the cause.

---

# Phase 9 — Package Management

**Goal:** Install, inspect, update, and troubleshoot software packages.

## Commands

~~~bash
sudo apt update
apt list --upgradable
apt-cache policy openssh-server
apt show openssh-server
dpkg -l openssh-server
dpkg -S /usr/sbin/sshd
sudo apt install tree
sudo apt remove tree
sudo apt --fix-broken install
sudo dpkg --configure -a
~~~

## Practical tasks

1. Inspect repositories and package candidates before installing.
2. Distinguish package index updates from package upgrades.
3. Inspect installed files and package ownership.
4. Review security updates and unattended-upgrades configuration.
5. Explain when Snap or source compilation may be appropriate.

## Labs

- **Lab 9.1:** Install a small package, inspect its metadata and files, then remove it.
- **Lab 9.2:** Diagnose a simulated package-management problem in a disposable VM.
- **Lab 9.3:** Produce a patch report with available updates and reboot status.

**Project:** Linux Patch Management Report — include installed versions, available updates, security considerations, maintenance window, and verification.

---

# Phase 10 — Linux Networking Fundamentals

**Goal:** Understand addressing, routes, DNS, transport protocols, sockets, and packet flow.

## Commands

~~~bash
ip -br link
ip -br address
ip route
ip neigh
ss -tulpn
ping -c 4 127.0.0.1
ping -c 4 <gateway-ip>
ping -c 4 1.1.1.1
getent hosts ubuntu.com
resolvectl status
tracepath 1.1.1.1
sudo tcpdump -ni ens33
~~~

Replace placeholders such as `<gateway-ip>` with actual values. Do not assume a particular interface name.

## Practical tasks

1. Identify MAC address, IPv4/IPv6 addresses, prefix, gateway, and DNS resolver.
2. Explain ARP/neighbor discovery, TCP/UDP, ports, sockets, and routing.
3. Test connectivity in layers: interface → address → local gateway → external IP → DNS → application.
4. Capture only traffic you are authorized to inspect.
5. Learn how network namespaces isolate interfaces and routes.

## Labs

- **Lab 10.1:** Inventory interface and route configuration.
- **Lab 10.2:** Diagnose a missing default route using read-only commands first.
- **Lab 10.3:** Identify listening sockets and the process using a selected port.
- **Lab 10.4:** Capture DNS or ICMP traffic in your own lab.

**Project:** Linux Network Diagnostic Toolkit — report interface state, IP, route, gateway test, DNS test, listening ports, and timestamped results.

---

# Phase 11 — Linux Network Configuration

**Goal:** Configure and troubleshoot IP addressing, routes, DNS, virtual interfaces, and network services.

## Inspect before changing

~~~bash
ip -br address
ip route
resolvectl status
networkctl status
nmcli device status
ls -l /etc/netplan/
sudo cat /etc/netplan/*.yaml
~~~

Some commands may not be installed or active on every Ubuntu Server configuration. Identify whether Netplan uses systemd-networkd or NetworkManager before editing configuration.

## Practical tasks

1. Document the current IP, gateway, DNS, and interface before a change.
2. Inspect the active Netplan YAML and renderer.
3. Configure a static IP only after confirming the correct subnet, gateway, and DNS.
4. Use a console or VM snapshot before applying changes that could disconnect SSH.
5. Learn the concepts of VLANs, bridges, bonding, MTU, IPv6, and network namespaces.

## Labs

- **Lab 11.1:** Configure a static address in an isolated lab and verify address, route, and DNS.
- **Lab 11.2:** Observe DHCP-assigned addressing and lease behavior.
- **Lab 11.3:** Diagnose separate simulated failures: wrong gateway, DNS failure, and missing route.
- **Lab 11.4:** Create and inspect network namespaces in a disposable VM.

**Project:** Linux Network Configuration Lab — provide the topology, address plan, configuration file, verification output, and rollback procedure.

---

# Phase 12 — SSH Remote Administration

**Goal:** Administer Linux servers remotely with authenticated and restricted access.

## Commands

~~~bash
sudo systemctl status ssh --no-pager
sudo sshd -t
sudo sshd -T
ssh user@server-address
ssh-keygen -t ed25519
ssh-copy-id user@server-address
scp file.txt user@server-address:~
sftp user@server-address
rsync -av ./lab-data/ user@server-address:~/lab-data/
ssh -v user@server-address
~~~

## Practical tasks

1. Install and enable OpenSSH Server.
2. Configure key-based authentication between two lab VMs.
3. Verify host keys and understand `known_hosts` and `authorized_keys`.
4. Inspect `/etc/ssh/sshd_config` and configuration drop-ins.
5. Test SSH configuration with `sshd -t` before reloading.
6. Practice SCP/SFTP/rsync, remote commands, local/remote forwarding, ProxyJump, and agent-forwarding concepts.

## Security and recovery

- Keep the current SSH session open while testing changes.
- Verify a second login before closing the known-good session.
- Do not disable password authentication until key login is confirmed.
- Restrict access with least privilege and firewall rules.

## Labs

- **Lab 12.1:** Connect to the server from a second VM.
- **Lab 12.2:** Configure and test Ed25519 key authentication.
- **Lab 12.3:** Transfer a test file and verify its checksum.
- **Lab 12.4:** Diagnose wrong key, permissions, refusal, and timeout separately.

**Project:** Secure Linux Remote Administration Lab — document authentication, permitted users, firewall rules, and troubleshooting procedure.

---

# Phase 13 — Linux Storage and Filesystems

**Goal:** Provision, mount, inspect, and troubleshoot virtual storage.

## Setup

Attach an additional virtual disk to a disposable VM. Identify it before making changes; device names can differ.

## Commands

~~~bash
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINTS,UUID
sudo blkid
sudo fdisk -l
findmnt
df -hT
df -i
du -xhd1 /var 2>/dev/null
sudo parted -l
sudo smartctl -a /dev/sdX
~~~

Use `/dev/sdX` only as a placeholder. Confirm the actual target with `lsblk`; do not copy a formatting command until the correct disk is identified. Install `smartmontools` only if needed and supported by the virtual device.

## Practical tasks

1. Explain block devices, partitions, GPT/MBR, ext4, XFS concepts, UUIDs, labels, and mount points.
2. Partition and format only the additional test disk.
3. Mount it temporarily and verify with `findmnt`.
4. Configure a persistent mount using UUID in `/etc/fstab`.
5. Test the fstab entry with `sudo mount -a` before rebooting.
6. Inspect disk blocks, inode use, quotas, filesystem checks, and swap concepts.

## Labs

- **Lab 13.1:** Partition, format, and mount an additional virtual disk.
- **Lab 13.2:** Configure a persistent mount by UUID and verify it after reboot.
- **Lab 13.3:** Diagnose a simulated disk-space or inode-space issue using read-only tools first.

**Project:** Linux Storage Administration Lab — document device identity, partition table, filesystem, mount options, persistent configuration, and recovery steps.

---

# Phase 14 — LVM

**Goal:** Create and extend logical storage volumes safely.

## Commands to learn

~~~bash
pvs
vgs
lvs
sudo pvcreate /dev/<test-disk-partition>
sudo vgcreate vg_lab /dev/<test-disk-partition>
sudo lvcreate -L 2G -n lv_data vg_lab
sudo mkfs.ext4 /dev/vg_lab/lv_data
sudo mkdir -p /mnt/lab-data
sudo mount /dev/vg_lab/lv_data /mnt/lab-data
sudo lvextend -L +1G /dev/vg_lab/lv_data
sudo resize2fs /dev/vg_lab/lv_data
~~~

Use these mutating commands only after confirming that the selected partition is the dedicated lab disk. The example assumes ext4; resizing commands differ by filesystem.

## Practical tasks

1. Explain physical volumes, volume groups, logical volumes, physical extents, and logical extents.
2. Build PV → VG → LV → filesystem → mount.
3. Extend a logical volume and its filesystem.
4. Learn snapshots and shrinking concepts without risking valuable data.
5. Capture LVM state before and after every operation.

**Project:** Linux Dynamic Storage Lab — demonstrate a safe volume extension and document verification and recovery.

---

# Phase 15 — Swap and Memory Management

**Goal:** Inspect RAM, virtual memory, swap, and memory pressure.

## Commands

~~~bash
free -h
swapon --show
vmstat 1 5
cat /proc/meminfo
grep -E 'MemAvailable|SwapTotal|SwapFree' /proc/meminfo
sysctl vm.swappiness
~~~

## Practical tasks

1. Distinguish total RAM, available memory, page cache, buffers, swap, and OOM behavior.
2. Inspect existing swap before adding any.
3. If required, create a small swap file in a disposable VM following Ubuntu documentation, set restrictive permissions, configure persistence, and verify with `swapon --show`.
4. Monitor memory and swap during a controlled workload.

**Project:** Linux Memory Troubleshooting Lab — record baseline memory, observed pressure, evidence, and a safe remediation.

---

# Phase 16 — Linux Monitoring and Observability

**Goal:** Establish a resource baseline and detect abnormal behavior.

## Commands

~~~bash
uptime
top
free -h
vmstat 1 5
df -hT
df -i
iostat -xz 1 3
lsof
ss -s
ps -eo pid,comm,%cpu,%mem --sort=-%cpu | head
~~~

Install `sysstat` if you need `iostat` and it is not present.

## Practical tasks

1. Record CPU, memory, swap, disk, inode, network, load, process, socket, and file-descriptor metrics.
2. Compare idle and workload measurements.
3. Identify which metric supports a suspected bottleneck.
4. Avoid claiming that high load alone proves CPU saturation; compare CPU, I/O, and process evidence.

## Labs

- Collect a baseline.
- Generate a controlled workload.
- Compare metrics before, during, and after.
- Write a short incident report with evidence and verification.

**Project:** Linux Server Health Monitor — output timestamped CPU, memory, disk, load, network, service, and uptime data.

---

# Phase 17 — Linux Logging

**Goal:** Find relevant events and correlate logs during investigation.

## Commands

~~~bash
journalctl -b
journalctl -p warning..alert
journalctl -u ssh --since today
journalctl -k
dmesg --level=err,warn
sudo tail -n 100 /var/log/syslog
sudo tail -n 100 /var/log/auth.log
~~~

Ubuntu log files vary by installed logging service and configuration. Check which files exist before using them.

## Practical tasks

1. Distinguish journald, syslog, kernel logs, authentication logs, service logs, and application logs.
2. Filter by service, boot, time range, and priority.
3. Inspect log rotation and retention.
4. Protect logs from unauthorized access and avoid collecting secrets in reports.

**Project:** Linux Log Investigation Platform — create reports for authentication, system, service, kernel, and web events with timestamps and evidence.

---

# Phase 18 — Bash Automation

**Goal:** Automate repeatable administration tasks with safe error handling.

## Commands and concepts

~~~bash
bash --version
shellcheck --version
chmod +x script.sh
./script.sh
bash -n script.sh
printf '%s\n' "$PATH"
crontab -l
systemctl list-timers
~~~

## Practical tasks

1. Learn variables, input/output, conditions, loops, functions, arrays, positional arguments, exit codes, command substitution, traps, logging, and debugging.
2. Quote variables and validate input.
3. Use explicit paths and safe temporary-file handling.
4. Use cron or systemd timers based on task requirements.
5. Test failure paths, not only successful runs.

## Labs

- Build `system-info.sh`, `disk-check.sh`, `service-check.sh`, and `backup.sh`.
- Add argument validation and meaningful exit codes.
- Run scripts manually, then schedule a non-destructive report.
- Inspect logs and verify the scheduled execution.

**Project:** Linux Administration Toolkit — combine inventory, health checks, service status, and reporting with documented usage and tests.

---

# Phase 19 — Linux Security Hardening

**Goal:** Reduce attack surface and verify security controls.

## Commands

~~~bash
sudo ufw status verbose
sudo ufw app list
sudo ss -tulpn
sudo systemctl --type=service --state=running
sudo apt list --upgradable
sudo aa-status
sudo journalctl -p warning..alert
~~~

## Practical tasks

1. Review accounts, sudo access, permissions, exposed services, SSH settings, firewall rules, updates, and logs.
2. Configure UFW in a VM console session. Allow required SSH before enabling the firewall to avoid locking yourself out.
3. Inspect AppArmor status and profiles.
4. Learn SELinux concepts when working with distributions that use it by default.
5. Learn auditd, authentication controls, integrity monitoring, security baselines, and vulnerability management.
6. Record before/after state and validate that required services remain available.

**Project:** Linux Security Hardening — deliver a baseline, hardening checklist, evidence, exceptions, and rollback procedure.

---

# Phase 20 — DNS Administration

**Goal:** Understand DNS records and configure a lab DNS server.

## Initial investigation commands

~~~bash
resolvectl status
getent hosts example.com
dig example.com
dig A example.com
dig AAAA example.com
dig MX example.com
dig @<dns-server-ip> <lab-domain>
~~~

## Practical tasks

1. Understand resolver configuration, zones, authoritative versus recursive service, and A/AAAA/CNAME/MX/NS/PTR/TXT records.
2. Draw the lab subnet and choose a reserved private IP for the DNS VM.
3. Choose a DNS implementation such as BIND9 only after deciding whether the lab needs authoritative DNS, caching, or forwarding.
4. Validate zone configuration with the appropriate tool before restarting the service.
5. Test queries from the server and a separate client.

## Labs

- Install and inspect DNS packages.
- Configure a private lab zone and a host record.
- Query the record from a client.
- Break a test record and diagnose the failure from logs and DNS responses.

**Project:** DNS Infrastructure — document zone files, records, clients, forwarding/recursion policy, access control, and test results. Do not expose an open recursive resolver to the public internet.

---

# Phase 21 — DHCP Administration

**Goal:** Understand dynamic addressing and configure a private lab DHCP service.

## Practical tasks

1. Learn scopes, leases, reservations, exclusions, subnet masks, gateways, DNS options, and lease logs.
2. Use an isolated virtual network. Do not start a second DHCP server on a bridged network where it could affect real clients.
3. Choose a DHCP implementation appropriate to the lab and inspect its configuration before starting it.
4. Verify leases and client address, route, and DNS information.
5. Test renewal and troubleshoot a client that receives no lease.

## Commands

~~~bash
ip -br address
ip route
resolvectl status
journalctl -u <dhcp-service-name> --since today
~~~

Service names and client lease commands vary by DHCP implementation. Use the implementation's documented tools.

**Project:** DHCP Infrastructure — connect several lab clients on an isolated network and document the scope, reservations, options, leases, logs, and recovery.

---

# Phase 22 — Web Server Administration

**Goal:** Deploy and troubleshoot Apache and Nginx.

## Nginx workflow

~~~bash
sudo apt update
sudo apt install -y nginx
sudo nginx -t
sudo systemctl enable --now nginx
systemctl status nginx --no-pager
curl -I http://127.0.0.1/
sudo journalctl -u nginx --since today
~~~

## Apache workflow

~~~bash
sudo apt install -y apache2
sudo apache2ctl configtest
sudo systemctl enable --now apache2
curl -I http://127.0.0.1/
~~~

Use one web server at a time on the same IP/port unless deliberately configuring separate ports or hosts.

## Practical tasks

1. Deploy a static page.
2. Configure a virtual host/server block and document root.
3. Check file ownership, permissions, service status, listening sockets, and access/error logs.
4. Configure a reverse proxy to a lab application.
5. Learn TLS certificates, security headers, and restricted admin paths before exposing a service.

**Project:** Linux Web Server Platform — deliver site configuration, test results, logs, firewall policy, TLS plan, and troubleshooting notes.

---

# Phase 23 — Application Server Administration

**Goal:** Run a Python application behind a web server using a managed service.

## Practical tasks

1. Create a small Flask or FastAPI application in a virtual environment.
2. Run it locally on a loopback address during initial testing.
3. Configure Gunicorn for WSGI or Uvicorn for ASGI as appropriate.
4. Use a dedicated unprivileged service account and a systemd unit.
5. Configure Nginx as a reverse proxy.
6. Store configuration in environment files with restrictive permissions; do not commit secrets.
7. Add health checks, logs, backup, and service recovery.

## Commands

~~~bash
python3 --version
python3 -m venv .venv
. .venv/bin/activate
python -m pip install --upgrade pip
systemctl status <app-service> --no-pager
journalctl -u <app-service> --since today
curl -i http://127.0.0.1:<app-port>/health
sudo nginx -t
~~~

**Project:** Production-like Python Application Server — document the request path, service user, systemd unit, reverse proxy, logs, firewall, backup, and verification.

---

# Phase 24 — File Sharing and Network Services

**Goal:** Share files between Linux and Windows clients with controlled access.

## Practical tasks

1. Learn NFS exports and mounts, Samba/SMB shares, SFTP, rsync, and FTP's security limitations.
2. Create a dedicated test directory and group.
3. Configure only the protocol needed for the lab.
4. Restrict access to the lab subnet and authorized users.
5. Verify permissions from the client, not only on the server.
6. Test logs and service restart behavior.

## Commands

~~~bash
findmnt
df -hT
ss -tulpn
getfacl /srv/lab-share
journalctl --since today
~~~

Use the service-specific validation commands for NFS or Samba after installation.

**Project:** Linux File Server — implement departmental shares, least-privilege access, logging, and backup/restore verification.

---

# Phase 25 — Backup and Recovery

**Goal:** Produce backups that can be verified and restored.

## Commands

~~~bash
tar -czf lab-backup.tar.gz lab-data/
tar -tzf lab-backup.tar.gz
sha256sum lab-backup.tar.gz
rsync -av --dry-run lab-data/ backup-data/
rsync -av lab-data/ backup-data/
~~~

## Practical tasks

1. Define what must be backed up, recovery point objective (RPO), recovery time objective (RTO), retention, and destination.
2. Compare full, incremental, and differential backup concepts.
3. Back up configuration and application data without copying volatile pseudo-filesystems.
4. Verify archive contents and checksums.
5. Restore to a separate directory and compare the recovered data.
6. Schedule backups only after manual backup and restore work correctly.

**Project:** Linux Backup and Disaster Recovery — demonstrate local and remote backup, retention, integrity verification, restore testing, and recovery documentation.

---

# Phase 26 — Linux Troubleshooting

**Goal:** Diagnose failures systematically instead of changing settings at random.

## Workflow

~~~text
Identify symptom
    ↓
Establish scope and impact
    ↓
Collect read-only evidence
    ↓
Choose the affected layer
    ↓
Form and test a hypothesis
    ↓
Make one controlled change
    ↓
Verify service from the client side
    ↓
Document root cause and prevention
~~~

## Commands by failure type

~~~bash
# Service
systemctl status <service> --no-pager
journalctl -u <service> --since today

# Network
ip -br address
ip route
ip neigh
ping -c 3 <gateway-ip>
getent hosts <hostname>
ss -tulpn

# Storage
df -hT
df -i
findmnt
lsblk -f

# Permissions
namei -l /path/to/file
stat /path/to/file
getfacl /path/to/file

# Resources
uptime
free -h
vmstat 1 5
ps -eo pid,stat,%cpu,%mem,cmd --sort=-%cpu | head
~~~

## Labs

Create reversible incidents in a disposable VM: SSH failure, DNS failure, wrong route, Nginx failure, permission denial, full test filesystem, broken test mount, and high CPU/memory workload.

**Project:** Linux Incident Response Lab — document incident ID, symptoms, impact, commands, evidence, root cause, resolution, verification, and prevention.

---

# Phase 27 — Linux Performance Engineering

**Goal:** Use measurements to identify CPU, memory, disk, process, and network bottlenecks.

## Commands

~~~bash
uptime
top
vmstat 1 10
free -h
iostat -xz 1 5
df -hT
df -i
ss -s
lsof
ps -eo pid,stat,%cpu,%mem,cmd --sort=-%cpu | head
~~~

## Practical tasks

1. Establish a baseline before generating load.
2. Run a controlled workload and capture metrics.
3. Correlate load average with CPU, I/O wait, memory pressure, and process state.
4. Investigate file descriptor and socket exhaustion.
5. Compare measurements before and after a change.

**Project:** Linux Performance Investigation — provide baseline, symptom, metrics, root cause, optimization, and before/after evidence.

---

# Phase 28 — Linux Containers

**Goal:** Build and operate containers while understanding their isolation and security boundaries.

## Commands

~~~bash
docker version
docker info
docker pull nginx:stable
docker run -d --name web-lab -p 8080:80 nginx:stable
docker ps
docker logs web-lab
docker inspect web-lab
curl -I http://127.0.0.1:8080/
docker stop web-lab
docker rm web-lab
~~~

Use the supported Docker installation instructions for Ubuntu; do not assume Docker is preinstalled.

## Practical tasks

1. Learn images, containers, Dockerfiles, Compose, namespaces, cgroups, overlay filesystems, networks, volumes, and port mapping.
2. Build a small image and inspect its configuration.
3. Persist data with a named volume.
4. Use a dedicated network and health checks where appropriate.
5. Avoid privileged containers and unnecessary host mounts.
6. Inspect logs, resource use, and restart policies.

**Project:** Containerized Linux Service Platform — deploy a reverse proxy and application with separate networking, persistent data where required, health checks, and documented recovery.

---

# Phase 29 — Linux Internals

**Goal:** Connect user-space commands to kernel services and system behavior.

## Commands

~~~bash
uname -r
lsmod
modinfo <module-name>
sysctl -a
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/mounts
strace -c ls
strace -e trace=openat,read,write cat /etc/hostname
~~~

## Practical tasks

1. Explain user space, kernel space, system calls, processes, threads, scheduling, virtual memory, page cache, VFS, inodes, dentries, drivers, interrupts, and kernel modules.
2. Inspect `/proc`, `/sys`, and sysctl settings.
3. Use strace on simple, non-sensitive commands.
4. Distinguish observing a setting from safely changing it.

**Project:** Linux System Call Investigation — trace a small command, identify representative system calls, and document what evidence supports each conclusion.

---

# Phase 30 — Advanced Linux Networking

**Goal:** Build isolated virtual networks and understand packet flow.

## Practical tasks

1. Learn network namespaces, veth pairs, bridges, VLANs, bonding, forwarding, NAT, policy routing, nftables, packet filtering, and connection tracking.
2. Build namespaces and virtual interfaces in a disposable VM.
3. Verify interface state, addresses, routes, and neighbor entries at each step.
4. Add firewall rules incrementally and document their effect.
5. Capture traffic with tcpdump and map it to the packet path.

## Commands

~~~bash
ip netns list
ip link show
ip route
ip rule
sudo nft list ruleset
sudo tcpdump -ni any
~~~

Only run configuration commands for namespaces, bridges, NAT, or firewall rules in an isolated lab with a recovery path.

**Project:** Linux Virtual Network Lab — diagram namespaces, interfaces, bridges, firewall, routes, and packet flow; include connectivity tests and rollback steps.

---

# Phase 31 — Virtualization

**Goal:** Understand Linux virtualization and manage test VMs where hardware supports it.

## Practical tasks

1. Learn hypervisors, KVM, QEMU, libvirt, virtual CPU/memory, disks, networks, snapshots, and VM lifecycle.
2. Check whether hardware virtualization is available.
3. Install KVM/libvirt only if the host and available resources support it.
4. Create a small test VM and inspect its CPU, memory, disk, and network configuration.
5. Compare VM and container isolation models.

## Commands

~~~bash
lscpu
lsmod | grep kvm
virsh list --all
virsh net-list --all
virsh dominfo <vm-name>
~~~

These libvirt commands require a working libvirt setup and appropriate permissions.

**Project:** Linux Virtualization Lab — document VM creation, network, storage, resource allocation, lifecycle, and recovery.

---

# Phase 32 — Cloud Linux Administration

**Goal:** Deploy and operate a Linux instance using cloud networking and identity controls.

## Practical tasks

1. Learn cloud VMs, SSH keys, security groups, IAM concepts, block/object storage, cloud-init, DNS, monitoring, backup, and cloud firewalls.
2. Prefer local VMs first; use a free/low-cost cloud tier only after checking current pricing and limits.
3. Apply least privilege and restrict inbound traffic.
4. Install a simple service, verify it remotely, and inspect logs.
5. Test backup and recovery; delete resources when finished to avoid unexpected charges.

## Commands

~~~bash
cloud-init status --long
ip -br address
ip route
systemctl --failed
journalctl -b
ss -tulpn
~~~

**Project:** Cloud Linux Server Deployment — document architecture, network rules, identity permissions, service setup, monitoring, backup, and teardown.

---

# Phase 33 — Configuration Management

**Goal:** Configure multiple servers consistently with Ansible.

## Practical tasks

1. Install Ansible on a management VM or control host.
2. Create an inventory for two lab servers.
3. Configure SSH keys and verify connectivity.
4. Write playbooks for package installation, users, SSH, firewall, and Nginx.
5. Use handlers, variables, templates, roles, and secrets management as the project grows.
6. Run playbooks twice and verify idempotency.

## Commands

~~~bash
ansible --version
ansible-inventory -i inventory.ini --list
ansible all -i inventory.ini -m ping
ansible-playbook -i inventory.ini site.yml --syntax-check
ansible-playbook -i inventory.ini site.yml --check
ansible-playbook -i inventory.ini site.yml
~~~

Use `--check` as a useful preview, not a guarantee that every task is fully simulated.

**Project:** Linux Server Provisioning with Ansible — build repeatable server setup and verify the resulting configuration.

---

# Phase 34 — Linux Infrastructure Automation

**Goal:** Combine Bash, Python, SSH, systemd, Ansible, and containers into a maintainable workflow.

## Practical tasks

1. Define a server inventory and a clear configuration source.
2. Automate health checks, package status, service status, disk use, backup status, and log collection.
3. Use SSH keys and least-privilege accounts for remote administration.
4. Add timeouts, input validation, logging, retries where justified, and meaningful exit codes.
5. Keep credentials out of source control.
6. Test partial failure and verify that automation reports failures accurately.

## Suggested repository

~~~text
linux-infrastructure-toolkit/
├── README.md
├── inventory/
├── scripts/
├── ansible/
├── systemd/
├── configs/
├── tests/
└── docs/
~~~

**Project:** Linux Infrastructure Automation Platform — deliver server inventory, remote health checks, configuration checks, backup status, log collection, and documented test results.

---

# Phase 35 — Enterprise Linux Capstone

**Goal:** Build, secure, monitor, troubleshoot, and document a small multi-server environment.

## Suggested architecture

~~~text
                 Lab / Internet Gateway
                         |
                 Firewall / Router
                         |
              Private Lab Network
                         |
       +-----------------+-----------------+
       |                 |                 |
     DNS/DHCP           Web/App          File/Backup
       |                 |                 |
       +-----------------+-----------------+
                         |
                 Monitoring / Logs
~~~

Use separate VMs or combine services only when host resources require it. Keep the environment private unless there is a specific, secured reason to expose a service.

## Required capabilities

- Linux installation, boot, filesystem, CLI, users, groups, and permissions
- systemd services and package management
- IP addressing, routing, DNS, DHCP, SSH, and firewall
- Storage, filesystems, mounts, and LVM
- Web server and a simple application service
- File sharing where required
- Monitoring, logging, backup, restore, and troubleshooting
- Bash automation and Ansible
- Docker and cloud concepts where resources permit
- Security baseline and documented recovery plan

## Required evidence

For each server, commit documentation containing:

1. Purpose and role
2. VM resources and network details
3. Installation and configuration steps
4. Commands used and their purpose
5. Verification output or screenshots with secrets removed
6. Firewall and access-control rules
7. Logs and monitoring checks
8. Backup and restore evidence
9. One failure scenario and its recovery
10. Known limitations and next improvements

**Final project:** Enterprise Linux Infrastructure Lab — demonstrate deployment, configuration, security, monitoring, troubleshooting, automation, recovery, and documentation from a clean snapshot.

---

# Practical Project Matrix

| Phase | Project | Main evidence |
|---|---|---|
| 0 | Linux Home Lab | VM layout, IP plan, baseline |
| 1 | System Inventory Tool | Host and OS inventory |
| 2 | Boot Troubleshooting Lab | Boot evidence and recovery |
| 3 | Filesystem Investigation Toolkit | Metadata, mounts, links |
| 4 | Linux Log Analyzer | Pipelines and report |
| 5 | Multi-User Server | Accounts and groups |
| 6 | Access Control Lab | Permission matrix and ACL tests |
| 7 | Process Monitor | Process and resource report |
| 8 | Service Health Manager | systemd status and logs |
| 9 | Patch Management Report | Package/update evidence |
| 10 | Network Diagnostic Toolkit | IP, route, DNS, sockets |
| 11 | Network Configuration Lab | Netplan and rollback |
| 12 | Secure Remote Administration | SSH keys and access policy |
| 13 | Storage Administration | Disk, filesystem, fstab |
| 14 | Dynamic Storage Lab | LVM extension |
| 15 | Memory Troubleshooting | RAM and swap evidence |
| 16 | Server Health Monitor | Baseline and resource report |
| 17 | Log Investigation | Correlated events |
| 18 | Administration Toolkit | Safe Bash scripts |
| 19 | Security Hardening | Baseline and validation |
| 20 | DNS Infrastructure | Zone and query tests |
| 21 | DHCP Infrastructure | Lease and client verification |
| 22 | Web Server Platform | Site, logs, TLS plan |
| 23 | Python Application Server | systemd and reverse proxy |
| 24 | Linux File Server | Client-side access tests |
| 25 | Backup and Disaster Recovery | Restore evidence |
| 26 | Incident Response Lab | Root-cause reports |
| 27 | Performance Investigation | Before/after metrics |
| 28 | Container Platform | Image, network, volume |
| 29 | System Call Investigation | strace observations |
| 30 | Virtual Network Lab | Packet path and routes |
| 31 | Virtualization Lab | VM lifecycle and resources |
| 32 | Cloud Linux Server | Secure deployment and teardown |
| 33 | Ansible Provisioning | Repeatable playbooks |
| 34 | Infrastructure Automation | Automated health/config checks |
| 35 | Enterprise Capstone | Integrated infrastructure evidence |

---

# Lab Difficulty Levels

- **Level 1 — Guided:** follow the setup steps and explain each command.
- **Level 2 — Semi-guided:** receive a goal, requirements, and verification criteria; choose the commands.
- **Level 3 — Independent:** receive only the problem and expected service behavior.
- **Level 4 — Incident:** investigate a deliberately broken, recoverable lab with no hint about the root cause.

# Required Lab Documentation Template

Every major lab should include a `README.md` with:

~~~text
1. Objective
2. Environment and prerequisites
3. Architecture / topology
4. Setup
5. Configuration
6. Commands and purpose
7. Expected behavior
8. Verification
9. Troubleshooting
10. Security considerations
11. Cleanup / rollback
12. Evidence and lessons learned
~~~

# Final Competency Checklist

## Fundamentals and system administration

- [ ] Identify Linux distribution, kernel, boot flow, and systemd
- [ ] Navigate the filesystem and inspect metadata, inodes, and mounts
- [ ] Use Bash, pipes, redirection, grep, find, sed, awk, and exit codes
- [ ] Manage users, groups, permissions, ACLs, and capabilities
- [ ] Inspect processes and manage systemd services
- [ ] Manage packages, updates, logs, and monitoring

## Networking and services

- [ ] Diagnose interface, IP, route, gateway, DNS, and application connectivity
- [ ] Configure Netplan safely and verify rollback
- [ ] Administer SSH with key authentication
- [ ] Understand TCP/UDP, sockets, packet capture, and network namespaces
- [ ] Deploy DNS/DHCP, web, application, and file services in a private lab
- [ ] Configure firewall rules without locking out administrative access

## Storage and reliability

- [ ] Partition and mount a test disk
- [ ] Configure and validate persistent mounts
- [ ] Create and extend LVM volumes
- [ ] Inspect memory, swap, disk, inode, and I/O pressure
- [ ] Create backups, verify integrity, and restore data
- [ ] Troubleshoot failures using evidence and verify recovery

## Security and modern infrastructure

- [ ] Apply least privilege and a Linux hardening baseline
- [ ] Inspect logs and authentication events
- [ ] Write safe, tested Bash automation
- [ ] Operate containers with appropriate isolation
- [ ] Understand Linux internals and advanced networking concepts
- [ ] Deploy a VM or cloud instance securely
- [ ] Configure multiple lab servers with Ansible
- [ ] Complete and document the enterprise capstone

# Final Learning Progression

~~~text
Lab Foundation
  → Linux Fundamentals
  → Installation and Boot
  → Filesystem and CLI
  → Users and Permissions
  → Processes and systemd
  → Packages
  → Networking and SSH
  → Storage and LVM
  → Memory, Monitoring, and Logging
  → Bash Automation
  → Security
  → DNS and DHCP
  → Web, Application, and File Services
  → Backup and Recovery
  → Troubleshooting and Performance
  → Containers and Linux Internals
  → Advanced Networking and Virtualization
  → Cloud Linux
  → Ansible and Infrastructure Automation
  → Enterprise Capstone
~~~

## Completion criteria

A topic is complete when you can:

**Explain it → Set it up → Configure it → Verify it → Troubleshoot it → Secure it → Automate it where appropriate → Document it.**

The target competency is to **deploy, configure, secure, monitor, troubleshoot, automate, recover, and document a Linux server independently**.
