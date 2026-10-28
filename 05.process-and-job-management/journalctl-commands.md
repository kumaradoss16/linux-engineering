`journalctl` is a Linux command used to view and analyze system logs collected by systemd's journal. It helps troubleshoot service failures, boot issues, SSH problems, and other system events.

## 1. Basic `journalctl` commands

| Command                           | Purpose                                         |
| --------------------------------- | ----------------------------------------------- |
| `journalctl`                      | Display available journal logs.                 |
| `journalctl -n 20`                | Show the latest 20 log entries.                 |
| `journalctl -f`                   | Follow logs live as new entries arrive.         |
| `journalctl -r`                   | Display logs in reverse order (newest first).   |
| `journalctl -b`                   | Show logs from the current boot.                |
| `journalctl -b -1`                | Show logs from the previous boot, if retained.  |
| `journalctl --since today`        | Show logs since the beginning of today.         |
| `journalctl --since "1 hour ago"` | Show logs from the past hour.                   |
| `journalctl -p err`               | Show error-level logs and more severe messages. |
| `journalctl -k`                   | Show kernel messages.                           |
| `journalctl --no-pager`           | Print logs directly without opening a pager.    |

## 2. View logs for a specific service

For example, to inspect SSH service logs:

```
journalctl -u ssh
```

Show the latest 50 SSH log entries:

```
journalctl -u ssh -n 50
```

Follow SSH logs live:

```
journalctl -u ssh -f
```

On some distributions, the service is named `sshd` instead of `ssh`.

Other examples:

```
journalctl -u apache2
journalctl -u NetworkManager
journalctl -u cron
```

The service name depends on your Linux distribution and installed software.

## 3. Filter logs by time and priority

```
journalctl --since "2026-10-28 09:00:00"
```

```
journalctl --since "30 minutes ago" --until "10 minutes ago"
```

Show only errors from the current boot:

```
journalctl -b -p err
```

Common priority levels:

| Priority | Meaning       |
| -------- | ------------- |
| `0`      | Emergency     |
| `1`      | Alert         |
| `2`      | Critical      |
| `3`      | Error         |
| `4`      | Warning       |
| `5`      | Notice        |
| `6`      | Informational |
| `7`      | Debug         |

For example, `-p warning` includes warnings and more severe messages.

## 4. Practical Linux server lab

Run these commands on your Ubuntu Server.

Step 1 — Check recent logs

```
journalctl -n 20
```

Step 2 — Check the current boot

```
journalctl -b
```

Step 3 — Find errors

```
journalctl -p err -b
```

Step 4 — Investigate SSH

```
journalctl -u ssh -n 30
```

Step 5 — Monitor logs live

```
journalctl -f
```

Press `Ctrl+C` to stop live monitoring.

## 5. Useful troubleshooting pattern

If a service fails to start, inspect its status and logs:

```
systemctl status ssh
journalctl -u ssh -b -n 50 --no-pager
```

The first command gives the service's current state; the second shows its recent logs from the current boot.

Remember: `systemctl` manages services, while `journalctl` examines their logs. Some logs may require `sudo` depending on your user's permissions and system configuration.
