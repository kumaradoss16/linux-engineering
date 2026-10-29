# systemd & Service Management: From Beginner to Server Administration

## Introduction

systemd is the init system and service manager used by modern Ubuntu, Debian, RHEL, Rocky Linux, AlmaLinux, and Fedora systems. It starts services, tracks dependencies, writes logs through `journald`, mounts filesystems, activates sockets, schedules timers, and coordinates shutdown.

Run every lab only on a disposable VM or container. Take a VM snapshot before starting. Do not stop, disable, mask, or edit services on a production server unless the change is approved and reversible.

Your service names, unit paths, PIDs, boot times, and log output will differ from the examples.

## Lab safety

Check the system and your privileges:

```bash
cat /etc/os-release
systemctl --version
whoami
id
sudo -v
```

Create a lab directory:

```bash
mkdir -p ~/systemd-lab
cd ~/systemd-lab
```

Safety rules:

- Never mask `systemd-journald`, `dbus`, `systemd-logind`, SSH, networking, or storage services without understanding the impact.
- Do not use `systemctl isolate` casually; it can stop unrelated services.
- Always run `systemctl daemon-reload` after editing or creating unit files.
- Validate unit files before restarting services.
- Prefer `systemctl edit` over editing vendor unit files directly.
- Do not disable or mask a service merely because it appears inactive.

***

# Part 1: systemd Fundamentals

## What is systemd?

systemd is both:

- **PID 1**: The first userspace process started by the kernel.
- **Service manager**: The component that starts, stops, restarts, and supervises services.

It replaced older init systems on most major distributions.

| Init system | Typical role |
|---|---|
| SysV init | Sequential shell-script-based boot and service control |
| Upstart | Event-based init system used by older Ubuntu releases |
| systemd | Dependency-aware parallel init, service supervision, logging, timers, sockets, mounts, and shutdown |

systemd responsibilities include:

- Starting and supervising services.
- Resolving dependencies.
- Logging through `journald`.
- Mounting filesystems.
- Scheduling timers.
- Activating services through sockets.
- Managing shutdown and reboot.

## PID 1

PID 1 is special because it is the first userspace process and the ancestor of most other userspace processes. On a modern systemd system, PID 1 is `systemd`.

Identify it:

```bash
ps -p 1 -o pid,comm,cmd
systemctl status
```

Example output; your values will differ:

```text
    PID COMM CMD
      1 systemd /sbin/init
```

If PID 1 fails or exits, the system cannot continue normal userspace operation. Kernel panics, corrupted initramfs, damaged systemd installation, or invalid critical boot configuration can prevent the system from booting normally.

## Units

A **unit** is a systemd configuration object that describes something systemd manages. Units are commonly defined in INI-style files ending with a type extension.

Examples:

```text
nginx.service
ssh.socket
logrotate.timer
tmp.mount
multi-user.target
```

### Unit-file locations

| Location | Purpose |
|---|---|
| `/usr/lib/systemd/system/` | Vendor/package-provided units on many distributions |
| `/lib/systemd/system/` | Common alias/vendor location on Debian/Ubuntu |
| `/etc/systemd/system/` | Administrator-created units and overrides; highest priority |
| `/run/systemd/system/` | Runtime-generated units; normally lost at reboot |

Configuration precedence generally favors `/etc/systemd/system/` over vendor locations. Create custom units and overrides under `/etc/systemd/system/`.

Do not edit package-owned files under `/usr/lib/systemd/system/` or `/lib/systemd/system/`. Package upgrades can overwrite them.

## Unit types

| Unit type | File extension | Purpose | Example |
|---|---|---|---|
| Service | `.service` | Manage daemons or background processes | `nginx.service`, `ssh.service` |
| Target | `.target` | Group units and define system states | `multi-user.target`, `graphical.target` |
| Socket | `.socket` | Activate a service when a socket receives traffic | `ssh.socket` |
| Timer | `.timer` | Schedule services; systemd alternative to cron | `logrotate.timer` |
| Mount | `.mount` | Manage filesystem mount points | `tmp.mount` |
| Automount | `.automount` | Mount filesystems on demand | `proc-fs-nfsd.automount` |
| Path | `.path` | Activate a unit when files or directories change | Configuration-change trigger |
| Device | `.device` | Represent hardware devices | Network or storage device units |
| Swap | `.swap` | Manage swap space | Swap unit |
| Slice | `.slice` | Group and limit resources for processes | `system.slice` |
| Scope | `.scope` | Manage externally created processes | User session scopes |

## Service states

| State | Meaning |
|---|---|
| `active (running)` | Service is running |
| `active (exited)` | Service completed successfully |
| `active (waiting)` | Service is waiting for an event |
| `inactive (dead)` | Service is not running |
| `failed` | Service failed to start or exited unexpectedly |
| `activating` | Service is starting |
| `deactivating` | Service is stopping |
| `masked` | Service is prevented from starting |

Inspect states:

```bash
systemctl is-active nginx
systemctl is-enabled nginx
systemctl is-failed nginx
systemctl status nginx --no-pager
```

## Startup and shutdown

A simplified boot sequence is:

```text
Firmware/BIOS/UEFI
  -> bootloader
    -> kernel + initramfs
      -> systemd as PID 1
        -> default.target
          -> dependencies and services
```

Common targets:

| Target | Typical use |
|---|---|
| `default.target` | The system’s normal boot target, usually linked to another target |
| `multi-user.target` | Server-like state: networking and multi-user services, no graphical login |
| `graphical.target` | Desktop system with graphical login |
| `rescue.target` | Single-user recovery environment |
| `emergency.target` | Minimal emergency shell |
| `shutdown.target` | Coordinates shutdown |

systemd starts units in parallel where dependencies allow. `Wants=` and `Requires=` determine whether a unit is pulled in; `After=` and `Before=` determine ordering.

During shutdown, systemd stops services in reverse dependency order where possible.

***

# Part 2: systemctl Commands

## `systemctl` basics

### Purpose

Controls and inspects systemd units.

### Syntax

```bash
systemctl [options] COMMAND [UNIT]
```

### Important global flags

| Flag | Meaning | Example |
|---|---|---|
| `--no-pager` | Disable pager output | `systemctl --no-pager` |
| `--all` | Include inactive units | `systemctl list-units --all` |
| `--type=TYPE` | Filter by unit type | `systemctl --type=service` |
| `--state=STATE` | Filter by state | `systemctl --type=service --state=running` |
| `--failed` | Show failed units | `systemctl --failed` |
| `--user` | Manage user-level units | `systemctl --user status app` |
| `--runtime` | Make change temporary until reboot | `sudo systemctl --runtime mask UNIT` |
| `-H USER@HOST` | Connect to a remote host over SSH | `systemctl -H admin@server status nginx` |
| `-M NAME` | Connect to a local container or VM | `systemctl -M container status nginx` |

### Beginner examples

```bash
systemctl
systemctl --no-pager
systemctl --type=service
systemctl --type=service --state=running
systemctl --failed
```

### Server-administration examples

```bash
systemctl list-units --type=service
systemctl list-units --type=target
systemctl list-units --type=socket
systemctl list-units --type=timer
systemctl list-units --type=mount
```

### Common mistakes

- Confusing active with enabled.
- Assuming `systemctl` output is complete without `--all`.
- Using `isolate` without understanding which services it may stop.
- Editing unit files without running `daemon-reload`.

### Verification

```bash
systemctl --failed
systemctl list-units --type=service --state=running
```

***

## Service status

### Purpose

Inspects whether a service is loaded, active, enabled, failed, and how it is configured.

### Syntax

```bash
systemctl status UNIT
systemctl is-active UNIT
systemctl is-enabled UNIT
systemctl is-failed UNIT
systemctl show UNIT
systemctl cat UNIT
systemctl help UNIT
```

### Important commands

| Command | Meaning |
|---|---|
| `systemctl status UNIT` | Human-readable status and recent journal entries |
| `systemctl is-active UNIT` | Print `active`, `inactive`, `failed`, or another state |
| `systemctl is-enabled UNIT` | Print boot-enablement state |
| `systemctl is-failed UNIT` | Print whether the unit has failed |
| `systemctl show UNIT` | Machine-readable unit properties |
| `systemctl cat UNIT` | Show unit file and drop-in overrides |
| `systemctl help UNIT` | Show documentation references |

### Beginner example

```bash
systemctl status ssh --no-pager
```

Ubuntu commonly uses `ssh.service`; RHEL-family systems commonly use `sshd.service`.

```bash
systemctl status sshd --no-pager 2>/dev/null || systemctl status ssh --no-pager
```

Example output; your values will differ:

```text
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-10-26 09:30:00 IST; 2h ago
   Main PID: 812 (sshd)
      Tasks: 1 (limit: 4653)
     Memory: 4.2M
        CPU: 120ms
     CGroup: /system.slice/ssh.service
             └─812 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
```

### Server-administration examples

```bash
systemctl is-active nginx
systemctl is-enabled nginx
systemctl is-failed nginx
systemctl show nginx -p MainPID,ActiveState,SubState,ExecMainStartTimestamp
systemctl cat nginx
```

### Reading status fields

| Field | Meaning |
|---|---|
| `Loaded` | Unit file location and enablement state |
| `Active` | Runtime state |
| `Docs` | Documentation references |
| `Main PID` | Main service process |
| `Tasks` | Tasks or threads associated with the service |
| `Memory` | Current memory use |
| `CPU` | CPU time consumed |
| `CGroup` | Control-group membership |
| Recent journal entries | Latest relevant logs |

### Common mistakes

- Assuming `enabled` means currently running.
- Assuming `active` means enabled at boot.
- Reading only `systemctl status` without checking journal logs.
- Using the wrong service name across distributions.

### Verification

```bash
systemctl is-active nginx
systemctl is-enabled nginx
systemctl show nginx -p MainPID,ActiveState
```

***

## Start, stop, restart, reload

### Purpose

Controls the runtime lifecycle of a service.

### Syntax

```bash
sudo systemctl start UNIT
sudo systemctl stop UNIT
sudo systemctl restart UNIT
sudo systemctl reload UNIT
sudo systemctl reload-or-restart UNIT
sudo systemctl try-restart UNIT
sudo systemctl kill UNIT
```

### Command meanings

| Command | Meaning |
|---|---|
| `start` | Start an inactive service |
| `stop` | Stop a running service |
| `restart` | Stop and start again |
| `reload` | Reload configuration without restarting, if supported |
| `reload-or-restart` | Reload if possible, otherwise restart |
| `try-restart` | Restart only if currently running |
| `kill` | Send a signal to service processes |

### Beginner example

```bash
sudo systemctl start nginx
systemctl status nginx --no-pager
sudo systemctl reload nginx
sudo systemctl restart nginx
```

### Server-administration examples

Apply an Nginx configuration change safely:

```bash
sudo nginx -t
sudo systemctl reload nginx
systemctl status nginx --no-pager
```

Restart an application after deployment:

```bash
sudo systemctl restart wavecorp-webapp
```

Restart SSH safely:

```bash
sudo systemctl restart ssh
```

On RHEL-family systems:

```bash
sudo systemctl restart sshd
```

Reload a firewall service:

```bash
sudo systemctl reload firewalld
```

### Common mistakes

- Using `restart` when `reload` is sufficient.
- Assuming every service supports `reload`.
- Restarting SSH from a single SSH session without a fallback console.
- Using `kill` before checking service logs and status.

### Verification

```bash
systemctl is-active nginx
systemctl status nginx --no-pager
journalctl -u nginx -n 20 --no-pager
```

***

## Enable, disable, mask, unmask

### Purpose

Controls whether a unit starts at boot and whether it can start at all.

### Syntax

```bash
sudo systemctl enable UNIT
sudo systemctl disable UNIT
sudo systemctl enable --now UNIT
sudo systemctl disable --now UNIT
sudo systemctl mask UNIT
sudo systemctl unmask UNIT
```

### Command meanings

| Command | Meaning |
|---|---|
| `enable` | Create startup symlink according to `WantedBy` or `RequiredBy` |
| `disable` | Remove startup symlink |
| `enable --now` | Enable and start immediately |
| `disable --now` | Disable and stop immediately |
| `mask` | Prevent the unit from being started manually or by dependencies |
| `unmask` | Allow the unit to start again |

### Disabled versus masked

| State | Can start manually? | Can start at boot? | Can be pulled in by dependency? |
|---|---|---|---|
| Disabled | Yes | No | Yes |
| Masked | No | No | No |

Masking is useful for hardening, but it is powerful. A masked unit cannot start even if another unit requires it.

### Beginner example

```bash
sudo systemctl enable --now nginx
systemctl is-enabled nginx
systemctl is-active nginx

sudo systemctl disable --now nginx
systemctl is-enabled nginx
systemctl is-active nginx
```

### Mask example

```bash
sudo systemctl mask nginx
sudo systemctl start nginx
```

Expected output:

```text
Failed to start nginx.service: Unit nginx.service is masked.
```

Restore it:

```bash
sudo systemctl unmask nginx
sudo systemctl enable --now nginx
```

### Safety warning

Do not mask essential services such as `systemd-journald`, `dbus`, `systemd-logind`, SSH, networking, or storage services without understanding the impact. Masking can prevent a required dependency from starting and can make a system difficult to recover.

### Verification

```bash
systemctl is-enabled nginx
systemctl is-active nginx
systemctl status nginx --no-pager
```

***

## List and inspect units

### Purpose

Lists active units, installed unit files, dependencies, sockets, timers, and pending jobs.

### Syntax

```bash
systemctl list-units [options]
systemctl list-unit-files [options]
systemctl list-dependencies UNIT
systemctl list-sockets
systemctl list-timers
systemctl list-jobs
```

### Important commands

| Command | Meaning |
|---|---|
| `systemctl list-units` | Show loaded and active units |
| `systemctl list-units --all` | Include inactive units |
| `systemctl list-units --type=service` | Show service units |
| `systemctl list-unit-files` | Show installed unit files |
| `systemctl list-unit-files --type=service` | Show installed service files |
| `systemctl list-dependencies UNIT` | Show dependencies |
| `systemctl list-dependencies --reverse UNIT` | Show reverse dependencies |
| `systemctl list-sockets` | Show listening sockets |
| `systemctl list-timers` | Show timers |
| `systemctl list-jobs` | Show pending systemd jobs |

### Beginner example

```bash
systemctl list-units --type=service
systemctl list-unit-files --type=service
systemctl list-timers --no-pager
```

### Server-administration examples

Inspect Nginx dependencies:

```bash
systemctl list-dependencies nginx
systemctl list-dependencies --reverse nginx
```

Check pending jobs:

```bash
systemctl list-jobs --no-pager
```

### Active versus enabled

| Concept | Meaning |
|---|---|
| Active | Currently running or otherwise active now |
| Enabled | Configured to start at boot |
| Installed unit file | Unit definition exists on disk |

A service can be active but disabled, enabled but inactive, or neither.

### Common mistakes

- Assuming `list-units` shows every installed unit.
- Confusing dependencies with ordering.
- Ignoring failed units.
- Treating a timer as inactive because its service is inactive.

### Verification

```bash
systemctl list-units --type=service --state=failed
systemctl list-dependencies nginx
```

***

## Default target and system state

### Purpose

Controls the default boot target and changes the running system state.

### Syntax

```bash
systemctl get-default
sudo systemctl set-default TARGET
sudo systemctl isolate TARGET
sudo systemctl rescue
sudo systemctl emergency
sudo systemctl reboot
sudo systemctl poweroff
```

### Important targets

| Target | Meaning |
|---|---|
| `default.target` | Symbolic link to the normal boot target |
| `multi-user.target` | Server-like state |
| `graphical.target` | Graphical desktop state |
| `rescue.target` | Recovery mode |
| `emergency.target` | Minimal emergency shell |
| `shutdown.target` | Shutdown coordination |

### Beginner example

```bash
systemctl get-default
```

Example output:

```text
graphical.target
```

Set a server-like default on a lab VM:

```bash
sudo systemctl set-default multi-user.target
systemctl get-default
```

### Isolate example

```bash
sudo systemctl isolate multi-user.target
```

**Warning:** `isolate` stops units not compatible with the new target. On a desktop VM, it may stop the graphical session. Do not run it remotely without an alternative access path.

### Recovery modes

```bash
sudo systemctl rescue
sudo systemctl emergency
```

Use these only from a console or VM console. Remote SSH access may become unavailable.

### Common mistakes

- Running `isolate rescue.target` over SSH without console access.
- Setting `multi-user.target` on a desktop and expecting the GUI to remain available.
- Confusing `reboot` with `poweroff`.

### Verification

```bash
systemctl get-default
systemctl list-units --type=target
```

***

# Part 3: journalctl Commands

## journald and journalctl

`systemd-journald` collects logs from the kernel, services, stdout/stderr, and syslog sources. `journalctl` queries those logs.

### Syntax

```bash
journalctl [options] [matches]
```

### Important flags

| Flag | Meaning | Example |
|---|---|---|
| `-u UNIT` | Filter by systemd unit | `journalctl -u nginx` |
| `-p PRIORITY` | Filter by priority | `journalctl -p err` |
| `-f` | Follow new entries | `journalctl -f` |
| `-n NUMBER` | Show last N entries | `journalctl -n 50` |
| `-b` | Current boot | `journalctl -b` |
| `-b -1` | Previous boot | `journalctl -b -1` |
| `--since` | Filter from a time | `journalctl --since today` |
| `--until` | Filter until a time | `journalctl --until "2026-10-28 10:00:00"` |
| `-x` | Add explanations where available | `journalctl -x` |
| `-o FORMAT` | Output format | `journalctl -o short-iso` |
| `--disk-usage` | Show journal disk usage | `journalctl --disk-usage` |
| `--vacuum-size` | Reduce journal size | `sudo journalctl --vacuum-size=200M` |
| `--vacuum-time` | Remove old entries | `sudo journalctl --vacuum-time=7d` |
| `-k` | Kernel messages | `journalctl -k` |
| `-g PATTERN` | Grep journal entries | `journalctl -g "failed"` |

### Beginner examples

```bash
journalctl
journalctl -n 50
journalctl -f
journalctl -b
journalctl -k
```

Press `q` to exit the pager.

### Server-administration examples

```bash
journalctl -u nginx
journalctl -u ssh
journalctl -u nginx -f
journalctl -u nginx --since today
journalctl -p err
journalctl -p warning
journalctl -g "failed"
```

Time-range query:

```bash
journalctl \
  --since "2026-10-28 09:00:00" \
  --until "2026-10-28 10:00:00"
```

Previous boot:

```bash
journalctl -b -1
```

Journal maintenance:

```bash
journalctl --disk-usage
sudo journalctl --vacuum-size=200M
sudo journalctl --vacuum-time=7d
```

## Log priorities

| Priority | Keyword | Meaning |
|---:|---|---|
| 0 | `emerg` | System is unusable |
| 1 | `alert` | Immediate action required |
| 2 | `crit` | Critical condition |
| 3 | `err` | Error |
| 4 | `warning` | Warning |
| 5 | `notice` | Normal but significant |
| 6 | `info` | Informational |
| 7 | `debug` | Debug message |

## Troubleshoot a failed service

```bash
systemctl status failed-service --no-pager
journalctl -u failed-service -n 100 --no-pager
journalctl -u failed-service -p err --since today --no-pager
systemctl cat failed-service
systemd-analyze verify /etc/systemd/system/failed-service.service
```

### Common mistakes

- Reading only the latest few lines.
- Forgetting to check the previous boot after a reboot.
- Assuming journald retains logs indefinitely.
- Editing unit files without `daemon-reload`.

### Verification

```bash
journalctl -u nginx -n 20 --no-pager
journalctl --disk-usage
```

***

# Part 4: systemd-analyze Commands

## Purpose

Analyzes boot time, unit dependencies, unit correctness, and security exposure.

### Syntax

```bash
systemd-analyze [options] COMMAND
```

### Important commands

| Command | Meaning |
|---|---|
| `systemd-analyze` | Show overall boot time |
| `systemd-analyze time` | Show boot time |
| `systemd-analyze blame` | Show time spent initializing units |
| `systemd-analyze critical-chain` | Show critical dependency chain |
| `systemd-analyze plot` | Generate a boot visualization |
| `systemd-analyze verify` | Validate unit files |
| `systemd-analyze security` | Show security exposure |
| `systemd-analyze dump` | Dump detailed systemd state |

### Important flags

| Flag | Meaning | Example |
|---|---|---|
| `--no-pager` | Disable pager | `systemd-analyze blame --no-pager` |
| `--no-legend` | Omit legend output | `systemd-analyze blame --no-legend` |
| `--root=PATH` | Analyze an alternate root | `systemd-analyze verify --root=/mnt/root UNIT` |
| `--offline` | Analyze without booting | `systemd-analyze verify --offline UNIT` |

### Beginner examples

```bash
systemd-analyze
systemd-analyze time
systemd-analyze blame | head -n 20
```

Example output; your values will differ:

```text
Startup finished in 2.145s (kernel) + 4.321s (initrd) + 9.876s (userspace) = 16.342s
multi-user.target reached after 9.654s in userspace.
```

### Server-administration examples

```bash
systemd-analyze critical-chain multi-user.target
systemd-analyze critical-chain nginx.service
systemd-analyze verify /etc/systemd/system/wavecorp-webapp.service
systemd-analyze security nginx.service
systemd-analyze plot > boot-analysis.svg
```

### Why verify matters

Run `systemd-analyze verify` after editing a unit file. It can detect:

- Invalid directives.
- Missing executable paths in some cases.
- Malformed dependencies.
- Incorrect unit syntax.

It does not prove that the application will work; it validates much of the unit configuration.

### Common mistakes

- Treating `blame` as a definitive cause of slow boot.
- Assuming a high security exposure score means an immediate vulnerability.
- Editing a unit and restarting without `daemon-reload`.
- Ignoring warnings from `verify`.

### Verification

```bash
systemd-analyze verify /etc/systemd/system/wavecorp-webapp.service
systemd-analyze critical-chain wavecorp-webapp.service
```

***

# Part 5: systemd Unit Files

## Service unit example

```ini
[Unit]
Description=WaveCorp Web Application
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=webapp
Group=webapp
WorkingDirectory=/opt/wavecorp/webapp
ExecStart=/usr/bin/python3 /opt/wavecorp/webapp/app.py
Restart=on-failure
RestartSec=5
ExecReload=/bin/kill -HUP $MAINPID
Environment=ENV=production
EnvironmentFile=/etc/wavecorp/webapp.env

[Install]
WantedBy=multi-user.target
```

## `[Unit]` section

| Directive | Meaning |
|---|---|
| `Description=` | Human-readable unit description |
| `Documentation=` | Documentation URL or manual page |
| `After=` | Start after listed units |
| `Before=` | Start before listed units |
| `Wants=` | Weak dependency; start if possible |
| `Requires=` | Strong dependency; required unit must succeed |
| `Conflicts=` | Starting this unit stops conflicting units |
| `ConditionPathExists=` | Start only if a path exists |

## `[Service]` section

| Directive | Meaning |
|---|---|
| `Type=` | How systemd determines service startup |
| `User=` | Run service as this user |
| `Group=` | Run service as this group |
| `WorkingDirectory=` | Working directory for the service |
| `ExecStart=` | Command to start the service |
| `ExecStartPre=` | Command before the main process |
| `ExecStartPost=` | Command after the main process starts |
| `ExecReload=` | Command used by `systemctl reload` |
| `ExecStop=` | Command used to stop the service |
| `Restart=` | Restart policy |
| `RestartSec=` | Delay before restart |
| `Environment=` | Set an environment variable |
| `EnvironmentFile=` | Read environment variables from a file |
| `StandardOutput=` | Where stdout goes |
| `StandardError=` | Where stderr goes |
| `TimeoutStartSec=` | Maximum startup time |
| `TimeoutStopSec=` | Maximum shutdown time |
| `KillMode=` | Which processes systemd kills on stop |
| `ProtectSystem=` | Restrict filesystem write access |
| `ProtectHome=` | Restrict access to home directories |
| `PrivateTmp=` | Give the service a private `/tmp` |
| `NoNewPrivileges=` | Prevent gaining new privileges |

## `[Install]` section

| Directive | Meaning |
|---|---|
| `WantedBy=` | Target that weakly wants this unit |
| `RequiredBy=` | Target that strongly requires this unit |
| `Alias=` | Additional unit name |
| `Also=` | Additional units enabled or disabled together |

## Service types

| Type | Meaning | Example use |
|---|---|---|
| `simple` | `ExecStart` starts the main process directly | Most scripts and applications |
| `exec` | Like `simple`, but systemd waits until the binary is executed | Modern services |
| `forking` | Parent exits and child continues | Traditional daemon |
| `oneshot` | Process performs one task and exits | Setup scripts, cleanup jobs |
| `notify` | Service signals readiness through `sd_notify` | Advanced services |
| `dbus` | Service acquires a D-Bus name | D-Bus services |

## Dependencies and ordering

| Directive | Meaning | Example |
|---|---|---|
| `Wants=` | Weak dependency; unit is started if possible, but failure does not stop this unit | `Wants=network-online.target` |
| `Requires=` | Strong dependency; if required unit fails, this unit is not started | `Requires=database.service` |
| `After=` | Ordering only; start after listed units | `After=network.target` |
| `Before=` | Ordering only; start before listed units | `Before=nginx.service` |
| `Conflicts=` | Starting this unit stops conflicting units | `Conflicts=shutdown.target` |
| `WantedBy=` | Used in `[Install]`; target weakly wants this unit | `WantedBy=multi-user.target` |
| `RequiredBy=` | Used in `[Install]`; target strongly requires this unit | `RequiredBy=multi-user.target` |

Dependency and ordering are different:

- `Wants=` and `Requires=` decide whether another unit is pulled in.
- `After=` and `Before=` decide startup order.
- A service may need both:

```ini
Wants=database.service
After=database.service
```

## Unit-file locations and overrides

Do not edit vendor files under:

```text
/usr/lib/systemd/system/
/lib/systemd/system/
```

Use:

```text
/etc/systemd/system/
```

Create a full custom unit:

```bash
sudo systemctl edit --full nginx
```

Create an override:

```bash
sudo systemctl edit nginx
```

This creates:

```text
/etc/systemd/system/nginx.service.d/override.conf
```

After editing any unit or drop-in:

```bash
sudo systemctl daemon-reload
sudo systemctl restart nginx
```

***

# Part 6: Timers, Sockets, and Mount Units

## Timer units

A timer activates another unit, usually a `.service`, according to a schedule.

### Backup service

```bash
sudo tee /usr/local/bin/wavecorp-backup.sh >/dev/null <<'EOF'
#!/usr/bin/env bash
set -euo pipefail
printf '%s\n' "WaveCorp backup started at $(date --iso-8601=seconds)"
# Replace with a real, tested backup command.
sleep 5
printf '%s\n' "WaveCorp backup completed"
EOF

sudo chmod 0755 /usr/local/bin/wavecorp-backup.sh
```

Create `/etc/systemd/system/wavecorp-backup.service`:

```ini
[Unit]
Description=WaveCorp Daily Backup

[Service]
Type=oneshot
ExecStart=/usr/local/bin/wavecorp-backup.sh
```

Create `/etc/systemd/system/wavecorp-backup.timer`:

```ini
[Unit]
Description=Run WaveCorp Backup Daily

[Timer]
OnCalendar=*-*-* 02:30:00
Persistent=true
Unit=wavecorp-backup.service

[Install]
WantedBy=timers.target
```

Enable it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now wavecorp-backup.timer
systemctl list-timers --no-pager
systemctl status wavecorp-backup.timer --no-pager
journalctl -u wavecorp-backup.service --no-pager
```

### Timer directives

| Directive | Meaning |
|---|---|
| `OnCalendar=` | Calendar-based schedule |
| `OnBootSec=` | Run a specified time after boot |
| `OnUnitActiveSec=` | Run a specified time after the unit last activated |
| `Persistent=` | Run a missed timer after the system was down |
| `RandomizedDelaySec=` | Add randomized delay to reduce simultaneous load |
| `Unit=` | Unit activated by the timer |

`timers.target` is a common target for enabling system timers.

### Timers versus cron

| Feature | systemd timer | cron |
|---|---|---|
| Integration | Native systemd unit and journal integration | Separate scheduler |
| Missed runs | `Persistent=true` can catch missed runs | Depends on implementation/configuration |
| Dependencies | Can use systemd dependencies | Limited |
| Resource controls | Can use cgroup/slice settings | Limited |
| Familiar syntax | Calendar syntax | crontab syntax |

## Socket units

Socket activation starts a service when traffic arrives on a configured socket.

Create `/etc/systemd/system/wavecorp-api.socket`:

```ini
[Unit]
Description=WaveCorp API Socket

[Socket]
ListenStream=8080
Accept=false

[Install]
WantedBy=sockets.target
```

Create a corresponding `/etc/systemd/system/wavecorp-api.service`:

```ini
[Unit]
Description=WaveCorp API Service
Requires=wavecorp-api.socket
After=wavecorp-api.socket

[Service]
ExecStart=/usr/local/bin/wavecorp-api
```

Enable:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now wavecorp-api.socket
systemctl list-sockets --no-pager
```

### Socket directives

| Directive | Meaning |
|---|---|
| `ListenStream=` | TCP listening socket |
| `Accept=` | Whether each connection receives its own service instance |
| `sockets.target` | Common target for socket units |

## Mount units

Create `/etc/systemd/system/data-backup.mount`:

```ini
[Unit]
Description=Backup Data Mount

[Mount]
What=/dev/sdb1
Where=/mnt/backup
Type=ext4
Options=defaults

[Install]
WantedBy=multi-user.target
```

### Mount directives

| Directive | Meaning |
|---|---|
| `What=` | Device, UUID, or source path |
| `Where=` | Mount point |
| `Type=` | Filesystem type |
| `Options=` | Mount options |

The mount unit name must match the mount path. For `/mnt/backup`, the unit is:

```text
data-backup.mount
```

For `/mnt/backup/data`, the unit is:

```text
data-backup-data.mount
```

For persistent mounts, prefer a properly tested `/etc/fstab` entry. systemd can generate mount units from `/etc/fstab` at boot.

***

# Hands-On Labs

# Lab 8.1 — Service Lifecycle

## Objective

Practice start, stop, restart, enable, disable, mask, and unmask.

Use a safe service such as `nginx`, `ssh`, `cron`, or a custom test service.

## Task

Complete the full lifecycle for a noncritical lab service and verify every state change.

## Solution

Install Nginx if it is not installed.

Ubuntu/Debian:

```bash
sudo apt update
sudo apt install nginx
```

RHEL/Rocky/Alma/Fedora:

```bash
sudo dnf install nginx
```

Check status:

```bash
systemctl status nginx --no-pager
systemctl is-active nginx
systemctl is-enabled nginx
```

Stop it:

```bash
sudo systemctl stop nginx
systemctl is-active nginx
```

Expected output:

```text
inactive
```

Start it:

```bash
sudo systemctl start nginx
systemctl is-active nginx
```

Expected output:

```text
active
```

Restart it:

```bash
sudo systemctl restart nginx
systemctl is-active nginx
```

Reload configuration if supported:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

Enable at boot:

```bash
sudo systemctl enable nginx
systemctl is-enabled nginx
```

Disable at boot:

```bash
sudo systemctl disable nginx
systemctl is-enabled nginx
```

Mask it:

```bash
sudo systemctl mask nginx
sudo systemctl start nginx
```

Expected output:

```text
Failed to start nginx.service: Unit nginx.service is masked.
```

Unmask and restore:

```bash
sudo systemctl unmask nginx
sudo systemctl enable --now nginx
systemctl is-enabled nginx
systemctl is-active nginx
```

## Expected-results checklist

- [ ] `start` makes the service active.
- [ ] `stop` makes it inactive.
- [ ] `restart` restarts the process.
- [ ] `enable` creates boot startup configuration.
- [ ] `disable` removes boot startup configuration.
- [ ] `mask` prevents startup completely.
- [ ] `unmask` restores normal behavior.
- [ ] Final state is enabled and active.

***

# Lab 8.2 — Create Custom Service

## Objective

Create a systemd service for a custom Python script.

## Task

Create a dedicated service user, application directory, Python service, log file, systemd unit, and automatic-restart test.

## Solution

Create the application directory and service user:

```bash
sudo mkdir -p /opt/wavecorp/webapp
sudo useradd -r -s /usr/sbin/nologin webapp
sudo chown -R webapp:webapp /opt/wavecorp/webapp
```

Create the script:

```bash
sudo tee /opt/wavecorp/webapp/app.py >/dev/null <<'EOF'
import time
import logging

logging.basicConfig(
    filename="/var/log/wavecorp-webapp.log",
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(message)s",
)

while True:
    logging.info("WaveCorp web application is running")
    time.sleep(10)
EOF
```

Create the log file:

```bash
sudo touch /var/log/wavecorp-webapp.log
sudo chown webapp:webapp /var/log/wavecorp-webapp.log
sudo chmod 0640 /var/log/wavecorp-webapp.log
```

Create the service:

```bash
sudo tee /etc/systemd/system/wavecorp-webapp.service >/dev/null <<'EOF'
[Unit]
Description=WaveCorp Web Application
After=network.target

[Service]
Type=simple
User=webapp
Group=webapp
WorkingDirectory=/opt/wavecorp/webapp
ExecStart=/usr/bin/python3 /opt/wavecorp/webapp/app.py
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

Validate and activate:

```bash
systemd-analyze verify /etc/systemd/system/wavecorp-webapp.service
sudo systemctl daemon-reload
sudo systemctl enable --now wavecorp-webapp
systemctl status wavecorp-webapp --no-pager
journalctl -u wavecorp-webapp -f
```

Press `q` to exit the journal pager.

### Test recovery

Find the main PID:

```bash
pid=$(systemctl show wavecorp-webapp -p MainPID --value)
echo "$pid"
```

Kill the process:

```bash
sudo kill "$pid"
```

Wait and verify:

```bash
sleep 8
systemctl status wavecorp-webapp --no-pager
journalctl -u wavecorp-webapp -n 20 --no-pager
```

## Expected-results checklist

- [ ] The custom service starts automatically.
- [ ] It runs as the `webapp` user.
- [ ] It restarts after failure because `Restart=on-failure`.
- [ ] Logs appear through `journalctl`.
- [ ] `systemctl show` reports a new main PID after recovery.

***

# Lab 8.3 — Service Failure and Troubleshooting

## Objective

Break a service configuration safely, diagnose the failure, and repair it.

Use the custom service from Lab 8.2.

## Failure 1: Incorrect `ExecStart` path

Edit the full unit:

```bash
sudo systemctl edit --full wavecorp-webapp
```

Change:

```ini
ExecStart=/usr/bin/python3 /opt/wavecorp/webapp/app-wrong.py
```

Reload and restart:

```bash
sudo systemctl daemon-reload
sudo systemctl restart wavecorp-webapp
```

Check failure:

```bash
systemctl status wavecorp-webapp --no-pager
journalctl -u wavecorp-webapp -n 50 --no-pager
systemd-analyze verify /etc/systemd/system/wavecorp-webapp.service
```

Expected journal-style error:

```text
python3: can't open file '/opt/wavecorp/webapp/app-wrong.py': [Errno 2] No such file or directory
```

Repair the path:

```bash
sudo systemctl edit --full wavecorp-webapp
```

Restore:

```ini
ExecStart=/usr/bin/python3 /opt/wavecorp/webapp/app.py
```

Reload and restart:

```bash
sudo systemctl daemon-reload
sudo systemctl restart wavecorp-webapp
systemctl is-active wavecorp-webapp
```

## Failure 2: Missing working directory

Temporarily rename the directory:

```bash
sudo mv /opt/wavecorp/webapp /opt/wavecorp/webapp.bak
sudo systemctl restart wavecorp-webapp
```

Observe failure:

```bash
systemctl status wavecorp-webapp --no-pager
journalctl -u wavecorp-webapp -n 50 --no-pager
```

Restore:

```bash
sudo mv /opt/wavecorp/webapp.bak /opt/wavecorp/webapp
sudo systemctl restart wavecorp-webapp
systemctl is-active wavecorp-webapp
```

## Failure 3: Permission problem

Change ownership incorrectly:

```bash
sudo chown -R root:root /opt/wavecorp/webapp
sudo systemctl restart wavecorp-webapp
```

Check logs:

```bash
journalctl -u wavecorp-webapp -n 50 --no-pager
```

Expected style of error:

```text
PermissionError: [Errno 13] Permission denied: '/var/log/wavecorp-webapp.log'
```

Restore correct ownership:

```bash
sudo chown -R webapp:webapp /opt/wavecorp/webapp
sudo chown webapp:webapp /var/log/wavecorp-webapp.log
sudo systemctl restart wavecorp-webapp
systemctl is-active wavecorp-webapp
```

## Failure 4: Syntax error in unit file

Add an invalid directive:

```bash
sudo systemctl edit --full wavecorp-webapp
```

Add under `[Service]`:

```ini
InvalidDirective=yes
```

Validate:

```bash
sudo systemctl daemon-reload
systemd-analyze verify /etc/systemd/system/wavecorp-webapp.service
```

Expected style of warning:

```text
Unknown key name 'InvalidDirective' in section 'Service'
```

Remove the invalid directive, then reload and verify:

```bash
sudo systemctl edit --full wavecorp-webapp
sudo systemctl daemon-reload
systemd-analyze verify /etc/systemd/system/wavecorp-webapp.service
sudo systemctl restart wavecorp-webapp
systemctl is-active wavecorp-webapp
```

## Expected-results checklist

- [ ] Bad `ExecStart` path was diagnosed from journal logs.
- [ ] Missing working directory was diagnosed.
- [ ] Permission-denied failure was diagnosed.
- [ ] Invalid unit directive was detected by `systemd-analyze verify`.
- [ ] Service was restored to active state after each failure.

***

# Part 7: Real-World Server Use Cases

## Web server management

Apply an Nginx configuration safely:

```bash
sudo nginx -t
sudo systemctl reload nginx
systemctl status nginx --no-pager
journalctl -u nginx -n 50 --no-pager
```

If reload is unavailable or the service is unhealthy:

```bash
sudo systemctl restart nginx
```

## Application deployment

Create a dedicated non-login user:

```bash
sudo useradd -r -M -s /usr/sbin/nologin appsvc
```

Create a service:

```ini
[Unit]
Description=WaveCorp Application
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=appsvc
Group=appsvc
WorkingDirectory=/opt/wavecorp/app
ExecStart=/usr/bin/python3 /opt/wavecorp/app/app.py
Restart=on-failure
RestartSec=5
NoNewPrivileges=yes
ProtectSystem=strict
ProtectHome=yes
PrivateTmp=yes
ReadWritePaths=/var/lib/wavecorp

[Install]
WantedBy=multi-user.target
```

Enable and inspect:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now wavecorp-app
systemctl status wavecorp-app --no-pager
journalctl -u wavecorp-app -f
```

## Scheduled backups

Use a `Type=oneshot` service and a timer:

```ini
[Unit]
Description=WaveCorp Backup

[Service]
Type=oneshot
ExecStart=/usr/local/bin/wavecorp-backup.sh
```

```ini
[Unit]
Description=WaveCorp Backup Timer

[Timer]
OnCalendar=*-*-* 02:30:00
Persistent=true
RandomizedDelaySec=10m
Unit=wavecorp-backup.service

[Install]
WantedBy=timers.target
```

Enable:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now wavecorp-backup.timer
systemctl list-timers --no-pager
```

## Socket activation

Use a socket unit to start an API only when traffic arrives:

```ini
[Unit]
Description=WaveCorp API Socket

[Socket]
ListenStream=8080
Accept=false

[Install]
WantedBy=sockets.target
```

Enable:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now wavecorp-api.socket
systemctl list-sockets --no-pager
```

## Security hardening

Review services:

```bash
systemctl list-unit-files --type=service --state=enabled
systemd-analyze security nginx.service
```

Apply sandboxing only after testing:

```ini
NoNewPrivileges=yes
ProtectSystem=strict
ProtectHome=yes
PrivateTmp=yes
ProtectKernelTunables=yes
ProtectControlGroups=yes
RestrictSUIDSGID=yes
```

Mask an unused service only after confirming nothing depends on it:

```bash
systemctl list-dependencies --reverse unused-service
sudo systemctl mask unused-service
```

## Boot performance troubleshooting

```bash
systemd-analyze
systemd-analyze blame | head -n 20
systemd-analyze critical-chain multi-user.target
systemd-analyze verify /etc/systemd/system/wavecorp-webapp.service
```

Disable an unnecessary service only after checking reverse dependencies:

```bash
systemctl list-dependencies --reverse service-name
sudo systemctl disable --now service-name
```

***

# Cheat Sheets

## systemctl command cheat sheet

```bash
systemctl status UNIT
systemctl is-active UNIT
systemctl is-enabled UNIT
systemctl is-failed UNIT
systemctl show UNIT
systemctl cat UNIT
systemctl help UNIT

sudo systemctl start UNIT
sudo systemctl stop UNIT
sudo systemctl restart UNIT
sudo systemctl reload UNIT
sudo systemctl reload-or-restart UNIT
sudo systemctl try-restart UNIT
sudo systemctl kill UNIT

sudo systemctl enable UNIT
sudo systemctl disable UNIT
sudo systemctl enable --now UNIT
sudo systemctl disable --now UNIT
sudo systemctl mask UNIT
sudo systemctl unmask UNIT

systemctl list-units
systemctl list-units --all
systemctl list-units --type=service
systemctl list-unit-files
systemctl list-unit-files --type=service
systemctl list-dependencies UNIT
systemctl list-dependencies --reverse UNIT
systemctl list-sockets
systemctl list-timers
systemctl list-jobs
systemctl --failed

systemctl get-default
sudo systemctl set-default multi-user.target
sudo systemctl isolate multi-user.target
sudo systemctl rescue
sudo systemctl emergency
sudo systemctl reboot
sudo systemctl poweroff

sudo systemctl daemon-reload
sudo systemctl edit UNIT
sudo systemctl edit --full UNIT
```

## journalctl command cheat sheet

```bash
journalctl
journalctl -n 50
journalctl -f
journalctl -u UNIT
journalctl -u UNIT -f
journalctl -u UNIT -n 100
journalctl -p err
journalctl -p warning
journalctl -b
journalctl -b -1
journalctl --since today
journalctl --since "YYYY-MM-DD HH:MM:SS" --until "YYYY-MM-DD HH:MM:SS"
journalctl -k
journalctl -g PATTERN
journalctl -x
journalctl -o short-iso
journalctl --disk-usage
sudo journalctl --vacuum-size=200M
sudo journalctl --vacuum-time=7d
```

## systemd-analyze cheat sheet

```bash
systemd-analyze
systemd-analyze time
systemd-analyze blame | head -n 20
systemd-analyze critical-chain
systemd-analyze critical-chain UNIT
systemd-analyze plot > boot-analysis.svg
systemd-analyze verify /etc/systemd/system/UNIT.service
systemd-analyze security UNIT
systemd-analyze dump
```

## Unit-type cheat sheet

| Type | Extension | Purpose |
|---|---|---|
| Service | `.service` | Daemon or background process |
| Target | `.target` | Group of units/system state |
| Socket | `.socket` | Socket-based activation |
| Timer | `.timer` | Scheduled activation |
| Mount | `.mount` | Filesystem mount |
| Automount | `.automount` | On-demand mount |
| Path | `.path` | File/directory change trigger |
| Device | `.device` | Hardware device |
| Swap | `.swap` | Swap space |
| Slice | `.slice` | Resource grouping |
| Scope | `.scope` | Externally created process group |

## Service-state cheat sheet

| State | Meaning |
|---|---|
| `active (running)` | Running |
| `active (exited)` | Completed successfully |
| `active (waiting)` | Waiting for an event |
| `inactive (dead)` | Not running |
| `failed` | Failed or exited unexpectedly |
| `activating` | Starting |
| `deactivating` | Stopping |
| `masked` | Prevented from starting |

## Dependency directive cheat sheet

| Directive | Meaning |
|---|---|
| `Wants=` | Weak dependency |
| `Requires=` | Strong dependency |
| `After=` | Start after |
| `Before=` | Start before |
| `Conflicts=` | Stop conflicting unit |
| `WantedBy=` | Install-time weak wanted-by relationship |
| `RequiredBy=` | Install-time strong required-by relationship |

***

# Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| Service fails to start | Bad `ExecStart`, missing dependency, permission issue, port conflict | Check `systemctl status`, `journalctl -u`, `systemctl cat`, and `systemd-analyze verify` |
| Service exits immediately | Application error, missing file, bad environment, wrong working directory | Read journal logs and run the command manually as the service user where safe |
| Service is active but application is not working | Wrong port, firewall, application error, bad backend dependency | Check listening sockets, application logs, firewall, and upstream dependencies |
| Service cannot bind to a port | Port already used, insufficient privilege, wrong address | Check `sudo ss -ltnp`, service logs, and capability/port configuration |
| Permission denied in service logs | Wrong `User`, `Group`, file ownership, SELinux/AppArmor policy | Correct ownership/permissions; inspect SELinux or AppArmor denials |
| Unit-file changes do not take effect | Missing `daemon-reload` or service not restarted | Run `sudo systemctl daemon-reload`, then restart or reload the unit |
| Service is masked accidentally | `systemctl mask` was run | Run `sudo systemctl unmask UNIT`, then enable/start as appropriate |
| Timer does not run | Timer disabled, wrong calendar syntax, service failed, system was asleep | Check `list-timers`, `systemctl status TIMER`, and service journal |
| Socket-activated service does not start | Socket disabled, port conflict, wrong socket/service linkage | Check `list-sockets`, socket status, service unit, and port availability |
| Boot is slow | Slow unit, dependency wait, network wait, storage issue | Use `systemd-analyze`, `blame`, and `critical-chain`; disable only unnecessary services |

***

# Practice Exercises

1. Find PID 1 and identify whether it is systemd.
2. List all failed units on the lab VM.
3. Check whether Nginx is active, enabled, and failed.
4. Stop, start, restart, and reload a safe service while verifying each state.
5. Create a custom Python service running as a dedicated non-login user.
6. Kill the custom service process and verify systemd restarts it.
7. Create a daily timer and inspect it with `systemctl list-timers`.
8. Use `journalctl` to view only error-level messages for a service.
9. Use `systemd-analyze verify` to detect an invalid unit directive.
10. Use `systemd-analyze blame` and `critical-chain` to identify boot-time contributors.

***

# Interview Questions

## 1. What is the difference between active and enabled?

Active means the unit is running or otherwise active now. Enabled means it is configured to start at boot.

## 2. What does `systemctl mask` do?

It prevents a unit from being started manually, at boot, or as a dependency. It is stronger than disabling.

## 3. What is the difference between `Wants=` and `Requires=`?

`Wants=` is a weak dependency; the wanted unit is started if possible. `Requires=` is stronger; if the required unit fails, this unit is not started.

## 4. Why must you run `systemctl daemon-reload` after editing a unit file?

systemd caches unit definitions. `daemon-reload` makes it reload unit files and drop-ins from disk.

## 5. How do you investigate a failed service?

Use `systemctl status UNIT`, `journalctl -u UNIT`, `systemctl cat UNIT`, and `systemd-analyze verify UNIT`. Check both current and previous boot logs when relevant.

***

# Final Capstone Project

## Objective

Create a secure custom Python application service, add a backup timer and socket activation, apply hardening, break and repair the service, and audit the final configuration.

## Project components

Create:

```text
/opt/wavecorp/webapp
/etc/wavecorp/webapp.env
/etc/systemd/system/wavecorp-webapp.service
/etc/systemd/system/wavecorp-webapp.timer
/etc/systemd/system/wavecorp-webapp.socket
/usr/local/bin/wavecorp-backup.sh
```

## 1. Create the application identity

```bash
sudo useradd -r -M -s /usr/sbin/nologin webapp
sudo mkdir -p /opt/wavecorp/webapp /var/lib/wavecorp
sudo chown -R webapp:webapp /opt/wavecorp/webapp /var/lib/wavecorp
```

## 2. Create the application

```bash
sudo tee /opt/wavecorp/webapp/app.py >/dev/null <<'EOF'
import logging
import os
import time
from http.server import BaseHTTPRequestHandler, HTTPServer

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(message)s",
)

PORT = int(os.environ.get("APP_PORT", "8080"))


class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header("Content-Type", "text/plain")
        self.end_headers()
        self.wfile.write(b"WaveCorp application is healthy\n")

    def log_message(self, fmt, *args):
        logging.info("%s - %s", self.address_string(), fmt % args)


if __name__ == "__main__":
    server = HTTPServer(("127.0.0.1", PORT), Handler)
    logging.info("WaveCorp application listening on port %s", PORT)
    server.serve_forever()
EOF

sudo chown webapp:webapp /opt/wavecorp/webapp/app.py
sudo chmod 0644 /opt/wavecorp/webapp/app.py
```

## 3. Create the environment file

```bash
sudo install -d -m 0750 /etc/wavecorp
sudo tee /etc/wavecorp/webapp.env >/dev/null <<'EOF'
APP_PORT=8080
ENV=production
EOF

sudo chown root:webapp /etc/wavecorp/webapp.env
sudo chmod 0640 /etc/wavecorp/webapp.env
```

## 4. Create the hardened service

Create `/etc/systemd/system/wavecorp-webapp.service`:

```ini
[Unit]
Description=WaveCorp Web Application
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=webapp
Group=webapp
WorkingDirectory=/opt/wavecorp/webapp
EnvironmentFile=/etc/wavecorp/webapp.env
ExecStart=/usr/bin/python3 /opt/wavecorp/webapp/app.py
Restart=on-failure
RestartSec=5
NoNewPrivileges=yes
ProtectSystem=strict
ProtectHome=yes
PrivateTmp=yes
ProtectKernelTunables=yes
ProtectControlGroups=yes
RestrictSUIDSGID=yes
ReadWritePaths=/var/lib/wavecorp

[Install]
WantedBy=multi-user.target
```

Validate and enable:

```bash
systemd-analyze verify /etc/systemd/system/wavecorp-webapp.service
sudo systemctl daemon-reload
sudo systemctl enable --now wavecorp-webapp
systemctl status wavecorp-webapp --no-pager
```

Test the application:

```bash
curl http://127.0.0.1:8080/
journalctl -u wavecorp-webapp -n 20 --no-pager
```

## 5. Add a cleanup timer

Create `/usr/local/bin/wavecorp-cleanup.sh`:

```bash
sudo tee /usr/local/bin/wavecorp-cleanup.sh >/dev/null <<'EOF'
#!/usr/bin/env bash
set -euo pipefail
find /var/lib/wavecorp -type f -mtime +7 -print -delete
EOF

sudo chmod 0750 /usr/local/bin/wavecorp-cleanup.sh
sudo chown root:webapp /usr/local/bin/wavecorp-cleanup.sh
```

Create `/etc/systemd/system/wavecorp-cleanup.service`:

```ini
[Unit]
Description=WaveCorp Temporary Data Cleanup

[Service]
Type=oneshot
User=webapp
Group=webapp
ExecStart=/usr/local/bin/wavecorp-cleanup.sh
```

Create `/etc/systemd/system/wavecorp-cleanup.timer`:

```ini
[Unit]
Description=Run WaveCorp Cleanup Daily

[Timer]
OnCalendar=*-*-* 03:15:00
Persistent=true
RandomizedDelaySec=10m
Unit=wavecorp-cleanup.service

[Install]
WantedBy=timers.target
```

Enable:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now wavecorp-cleanup.timer
systemctl list-timers --no-pager
```

## 6. Add socket activation

Create `/etc/systemd/system/wavecorp-webapp.socket`:

```ini
[Unit]
Description=WaveCorp Web Application Socket

[Socket]
ListenStream=8080
Accept=false

[Install]
WantedBy=sockets.target
```

Because the service binds to the port itself in this lab, enable either the service or the socket demonstration—not both simultaneously on the same port. For a true socket-activated service, the application should accept the socket passed by systemd.

For this capstone, demonstrate socket configuration without enabling it:

```bash
sudo systemctl daemon-reload
systemctl cat wavecorp-webapp.socket
systemd-analyze verify /etc/systemd/system/wavecorp-webapp.socket
```

## 7. Break and repair the service

Break the application path:

```bash
sudo systemctl edit --full wavecorp-webapp
```

Change:

```ini
ExecStart=/usr/bin/python3 /opt/wavecorp/webapp/app-missing.py
```

Reload and test:

```bash
sudo systemctl daemon-reload
sudo systemctl restart wavecorp-webapp
systemctl status wavecorp-webapp --no-pager
journalctl -u wavecorp-webapp -n 50 --no-pager
```

Repair:

```bash
sudo systemctl edit --full wavecorp-webapp
```

Restore:

```ini
ExecStart=/usr/bin/python3 /opt/wavecorp/webapp/app.py
```

Recover:

```bash
sudo systemctl daemon-reload
sudo systemctl restart wavecorp-webapp
systemctl is-active wavecorp-webapp
curl http://127.0.0.1:8080/
```

## 8. Audit the final configuration

```bash
systemctl status wavecorp-webapp --no-pager
systemctl is-enabled wavecorp-webapp
systemctl is-active wavecorp-webapp

systemctl list-timers --no-pager
systemctl list-sockets --no-pager

journalctl -u wavecorp-webapp -n 100 --no-pager
journalctl -u wavecorp-cleanup.service -n 50 --no-pager

systemd-analyze verify \
  /etc/systemd/system/wavecorp-webapp.service \
  /etc/systemd/system/wavecorp-cleanup.service \
  /etc/systemd/system/wavecorp-webapp.socket

systemd-analyze security wavecorp-webapp.service
systemd-analyze critical-chain wavecorp-webapp.service
```

## Expected capstone results

- [ ] `wavecorp-webapp` runs as `webapp`.
- [ ] The service restarts after a controlled process failure.
- [ ] Application logs are visible through `journalctl`.
- [ ] Cleanup timer is enabled and visible in `systemctl list-timers`.
- [ ] Socket unit validates successfully.
- [ ] Hardening options are present in the unit file.
- [ ] The controlled failure was diagnosed and repaired.
- [ ] Final audit reports no failed units and no invalid unit files.
