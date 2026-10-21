# Linux Users & Groups

> **Lab-only warning:** Take a VM snapshot before starting. Run these commands only on a disposable VM or container, never on production. User and group changes affect authentication, file ownership, sudo access, and service behavior.

Primary target: Ubuntu 22.04/24.04. RHEL/Rocky/AlmaLinux differences are shown where relevant. Red Hat systems commonly grant administrator access through the `wheel` group; sudo policy is defined in `/etc/sudoers` and drop-ins under `/etc/sudoers.d/`. [access.redhat](https://access.redhat.com/sites/products/files/2026-08/rhel9_configuring_basic_system_settings_guide.pdf)

# Introduction

Linux identifies processes through numeric **UIDs** and **GIDs**, then maps those numbers to human-friendly user and group names. Every file has an owning UID and GID; every running process has a real/effective identity and group set.

Start by recording your current identity:

```bash
whoami
id
groups
pwd
```

**Example output; your values will differ:**

```text
student
uid=1000(student) gid=1000(student) groups=1000(student),27(sudo)
student sudo
/home/student
```

Why it matters: access decisions are fundamentally numeric. A username is convenient, but the kernel checks UID, GID, supplementary group memberships, permissions, ACLs, and security policy.

***

# Concepts

## 1. Linux identity model

Linux commonly has three local-account categories:

| Account type | Typical use | Login shell | Example |
|---|---|---|---|
| Human account | A person administering or using the server | `/bin/bash` or another interactive shell | `developer01` |
| System account | Operating-system component | Usually no interactive shell | `systemd-network` |
| Service account | Application process identity | `nologin` or `/bin/false` | `www-data`, `postgres` |

Inspect a few accounts:

```bash
getent passwd root
getent passwd "$(whoami)"
getent passwd www-data 2>/dev/null || true
```

**Example output; your values will differ:**

```text
root:x:0:0:root:/root:/bin/bash
student:x:1000:1000:Student User,,,:/home/student:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

A service account limits the blast radius of a compromised service: the process should not run as `root`, should not have a useful password, and usually should not permit interactive login.

## 2. UID and GID

A **UID** identifies a user; a **GID** identifies a group. UID `0` is the root superuser and must be unique in a secure system.

Inspect configured allocation policy:

```bash
grep -E '^(UID_MIN|UID_MAX|SYS_UID_MIN|SYS_UID_MAX|GID_MIN|GID_MAX|SYS_GID_MIN|SYS_GID_MAX|CREATE_HOME)' /etc/login.defs
```

**Example output; your values will differ:**

```text
UID_MIN             1000
UID_MAX            60000
GID_MIN             1000
GID_MAX            60000
CREATE_HOME yes
```

Do not assume exact ranges. Ubuntu and RHEL defaults can differ by release and local policy. Verify `/etc/login.defs` and existing identities before assigning numeric IDs.

Inspect UID 0 accounts:

```bash
getent passwd | awk -F: '$3 == 0 {print $1 ":" $3 ":" $4 ":" $7}'
```

**Expected secure result:**

```text
root:0:0:/bin/bash
```

More than one UID-0 account requires immediate review.

## 3. Primary and supplementary groups

Every user has:

- **One primary group**: recorded as field 4 in `/etc/passwd`.
- **Zero or more supplementary groups**: recorded in `/etc/group` and `/etc/gshadow`.

Inspect your memberships:

```bash
id
id -gn
id -Gn
groups
```

**Example output; your values will differ:**

```text
uid=1000(student) gid=1000(student) groups=1000(student),27(sudo)
student
student sudo
student sudo
```

Why it matters: the primary group commonly owns newly created files, while supplementary groups grant collaborative access to shared resources.

## 4. `/etc/passwd`

Read it safely:

```bash
getent passwd root
getent passwd "$(whoami)"
```

A standard line has seven colon-separated fields:

```text
name:password-placeholder:UID:GID:GECOS:home-directory:login-shell
```

Example:

```text
student:x:1000:1000:Student User,,,:/home/student:/bin/bash
```

| Field | Example | Meaning |
|---:|---|---|
| 1 | `student` | Login name |
| 2 | `x` | Password data is stored in `/etc/shadow` |
| 3 | `1000` | UID |
| 4 | `1000` | Primary GID |
| 5 | `Student User,,,` | GECOS/comment field |
| 6 | `/home/student` | Home directory |
| 7 | `/bin/bash` | Login shell |

Inspect the fields directly:

```bash
getent passwd "$(whoami)" | awk -F: '{
  print "Name: " $1
  print "Password placeholder: " $2
  print "UID: " $3
  print "Primary GID: " $4
  print "GECOS: " $5
  print "Home: " $6
  print "Shell: " $7
}'
```

## 5. `/etc/shadow`

Only root normally reads `/etc/shadow`:

```bash
sudo getent shadow "$(whoami)"
```

A shadow entry has nine colon-separated fields:

```text
name:password-hash:last-change:min:max:warn:inactive:expire:reserved
```

Example with a deliberately abbreviated hash placeholder:

```text
student:$y$j9T$...:20000:0:99999:7::: 
```

| Field | Meaning |
|---:|---|
| 1 | Username |
| 2 | Password hash or account marker |
| 3 | Days since 1970-01-01 when password was last changed |
| 4 | Minimum days before password may change |
| 5 | Maximum password age in days |
| 6 | Warning days before expiry |
| 7 | Inactive days after password expiry before account disablement |
| 8 | Account-expiry date as days since 1970-01-01 |
| 9 | Reserved field |

Never display, copy, or fabricate real password hashes. A modular crypt-format entry commonly begins with an algorithm identifier such as `$y$` for yescrypt, followed by parameter/salt/hash data. The exact algorithm and configuration are system-specific.

Important password-field markers:

| Marker | General meaning |
|---|---|
| `$y$j9T$...` | Password hash placeholder in this guide |
| `!hash` | Password login locked; the original hash is prefixed with `!` |
| `*` | No usable password login; often used for system/service identities |
| Empty field | Potentially passwordless account; investigate immediately |

A password lock does **not** automatically prevent SSH public-key login, existing sessions, or other authentication paths. Account expiration and SSH policy may also be needed.

## 6. `/etc/group` and `/etc/gshadow`

Inspect your primary group:

```bash
getent group "$(id -gn)"
sudo getent gshadow "$(id -gn)"
```

`/etc/group` fields:

```text
group-name:password-placeholder:GID:member-list
```

Example:

```text
developers:x:3001:developer01,developer02
```

`/etc/gshadow` fields:

```text
group-name:encrypted-password:administrators:members
```

Example:

```text
developers:!::developer01,developer02
```

Group passwords and group administrators are rarely used in modern administration. Manage membership through `usermod`, `gpasswd`, configuration management, directory services, and audited sudo rules instead.

## 7. Root

`root` is UID 0 and bypasses most discretionary access-control checks. Direct root login is discouraged because it removes individual accountability and increases the chance of catastrophic mistakes.

Check root identity:

```bash
id root
sudo -l
```

Use `sudo` for individual privileged commands instead of an unrestricted root shell whenever possible.

## 8. `sudo` and `su`

- `sudo command`: execute one command under another identity, typically root.
- `sudo -i`: start a root login shell.
- `su user`: switch identity while retaining much of the current environment.
- `su - user`: switch identity and simulate a login shell.

Compare environment behavior:

```bash
echo "$HOME"
su nobody -s /bin/sh -c 'printf "su HOME=%s\n" "$HOME"' 2>/dev/null || true
sudo -u nobody -H sh -c 'printf "sudo HOME=%s\n" "$HOME"' 2>/dev/null || true
```

Use `visudo`, not a normal editor, for sudo policy. It validates syntax and uses safe locking. Red Hat documents `visudo -f /etc/sudoers.d/filename` for validating and editing sudo drop-ins. [docs.redhat](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_basic_system_settings/managing-sudo-access_configuring-basic-system-settings)

## 9. Login shells

Check valid login shells:

```bash
cat /etc/shells
```

**Example output; your values will differ:**

```text
/bin/sh
/usr/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/sbin/nologin
```

Common non-interactive shells:

| Shell | Typical purpose |
|---|---|
| `/usr/sbin/nologin` | Reject interactive logins with a message; location may be `/sbin/nologin` on some systems |
| `/sbin/nologin` | Alternate path used by some distributions |
| `/bin/false` | Immediately exits unsuccessfully; less informative |
| `/bin/bash` | Interactive Bash shell |

Verify available paths:

```bash
command -v nologin
ls -l /sbin/nologin /usr/sbin/nologin /bin/false 2>/dev/null
```

## 10. Home directories and `/etc/skel`

`/etc/skel` contains template files copied into newly created home directories when home creation is enabled.

```bash
sudo ls -la /etc/skel
```

**Example output:**

```text
.
..
.bash_logout
.bashrc
.profile
```

Why it matters: standard startup files, secure defaults, and user guidance can be centrally provisioned. Do not place secrets in `/etc/skel`.

## 11. User environment

Shell startup files differ by shell type:

| Context | Typical Bash files |
|---|---|
| Login shell | `/etc/profile`, then `~/.bash_profile`, `~/.bash_login`, or `~/.profile` |
| Interactive non-login shell | `~/.bashrc` |
| System-wide supplementary login configuration | `/etc/profile.d/*.sh` |

Inspect current shell and `PATH`:

```bash
echo "$SHELL"
ps -p $$ -o pid,ppid,args=
printf '%s\n' "$PATH"
umask
```

**Example output; your values will differ:**

```text
/bin/bash
    PID    PPID COMMAND
  12345   12340 -bash
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
0022
```

`umask 0022` commonly yields files that are not writable by other users. Shared collaboration areas often need a deliberately chosen group-oriented policy such as `0002`, but apply it only after understanding the impact.

## 12. Password management

Core tools:

```bash
sudo passwd developer01
sudo chage -l developer01
```

For labs only, a temporary password can be set non-interactively:

```bash
printf '%s\n' 'developer01:LabOnly-ChangeMe-2026!' | sudo chpasswd
```

Do not put passwords in shell history, scripts, CI logs, tickets, or production command lines. For real servers, use approved secret-management and onboarding processes.

Inspect policy defaults:

```bash
grep -E '^(PASS_MAX_DAYS|PASS_MIN_DAYS|PASS_WARN_AGE|ENCRYPT_METHOD)' /etc/login.defs
```

PAM modules enforce much of the actual password complexity and history policy. `/etc/login.defs` alone does not define the full password-policy posture.

## 13. Full user lifecycle

A well-managed account follows this sequence:

```text
Create -> Verify -> Modify -> Lock -> Expire -> Archive -> Delete -> Audit
```

Offboarding should first disable access, terminate sessions, preserve required evidence/data, remove authorizations, then delete or retain the account according to policy.

***

# Required Command Sections

## `id`

### Purpose

Displays a user’s UID, primary GID, and supplementary groups.

### Syntax

```bash
id [OPTION] [USER]
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-u` | Show UID | `id -u developer01` |
| `-g` | Show primary GID | `id -g developer01` |
| `-G` | Show all group IDs | `id -G developer01` |
| `-n` | Show names with `-u`, `-g`, or `-G` | `id -Gn developer01` |
| `-Z` | Show SELinux context where supported | `id -Z` |

### Basic example

```bash
id
```

**Example output; your values will differ:**

```text
uid=1000(student) gid=1000(student) groups=1000(student),27(sudo)
```

### Real-world example

Verify a developer received the intended access group before granting access to a shared project tree:

```bash
id developer01
```

### Common mistakes and gotchas

- Confusing a user’s primary GID with all group memberships.
- Forgetting that a user must start a new login session to receive new supplementary-group membership.
- Assuming group-name output is sufficient when auditing UID/GID collisions.

### Verify it worked

```bash
id developer01
getent passwd developer01
getent group developers
```

***

## `groups`

### Purpose

Shows the groups associated with the current user or a named user.

### Syntax

```bash
groups [USER...]
```

### Important options

`groups` has few commonly needed options; use `id -Gn` for more script-oriented group output.

| Option | Meaning | Example |
|---|---|---|
| No option | Print group names | `groups developer01` |
| N/A | Use `id -Gn` for names only | `id -Gn developer01` |

### Basic example

```bash
groups
```

**Example output; your values will differ:**

```text
student sudo
```

### Real-world example

Check whether a support engineer is in a sensitive administrative group:

```bash
groups support01
```

### Common mistakes and gotchas

- The output may not expose every directory-service detail in complex environments.
- New group membership may not appear inside an old login session.
- `groups` does not replace examining `getent group GROUP` for group membership records.

### Verify it worked

```bash
groups developer01
id -Gn developer01
getent group developers
```

***

## `whoami`

### Purpose

Prints the effective username of the current process.

### Syntax

```bash
whoami
```

### Basic example

```bash
whoami
sudo whoami
```

**Example output; your values will differ:**

```text
student
root
```

### Real-world example

Confirm whether an administrative shell is actually root before changing system configuration:

```bash
whoami
```

### Common mistakes and gotchas

- `whoami` does not show your original login identity after `sudo`.
- It does not show group memberships or authorization.

### Verify it worked

```bash
whoami
id
```

***

## `who`

### Purpose

Shows users currently logged into the system through tracked login sessions.

### Syntax

```bash
who [OPTION]...
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-a` | Show all available session information | `who -a` |
| `-H` | Print column headings | `who -H` |
| `-q` | Print names and count only | `who -q` |

### Basic example

```bash
who
```

**Example output; your values will differ:**

```text
student  pts/0  2026-10-20 13:10 (192.0.2.50)
```

### Real-world example

Check whether a departing employee still has active terminal sessions before offboarding:

```bash
who
```

### Common mistakes and gotchas

- Graphical, container, service, and SSH sessions may not all appear as expected in every environment.
- `who` does not prove every running process belongs to the shown users.

### Verify it worked

```bash
who -a
loginctl list-sessions 2>/dev/null || true
```

***

## `w`

### Purpose

Shows logged-in users and what they are currently doing.

### Syntax

```bash
w [OPTION] [USER]
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-h` | Suppress header | `w -h` |
| `-s` | Short format | `w -s` |
| `USER` | Show one user | `w developer01` |

### Basic example

```bash
w
```

**Example output; your values will differ:**

```text
 14:10:00 up 2:01, 1 user, load average: 0.02, 0.03, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
student  pts/0    192.0.2.50       13:10    0.00s  0.04s  0.00s w
```

### Real-world example

Before restarting a service, identify whether an administrator is already connected and performing maintenance:

```bash
w
```

### Common mistakes and gotchas

- Command information is a snapshot and can change immediately.
- Container environments may show limited or no session data.

### Verify it worked

```bash
w
who
```

***

## `useradd`

### Purpose

Creates a local user account.

### Syntax

```bash
sudo useradd [OPTIONS] LOGIN
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-m` | Create home directory | `sudo useradd -m analyst01` |
| `-d DIR` | Set home directory | `sudo useradd -m -d /srv/home/analyst01 analyst01` |
| `-s SHELL` | Set login shell | `sudo useradd -s /bin/bash analyst01` |
| `-c TEXT` | Set GECOS/comment | `sudo useradd -c "Asha Rao, Analyst" analyst01` |
| `-g GROUP` | Set primary group | `sudo useradd -g developers analyst01` |
| `-G GROUPS` | Set supplementary groups | `sudo useradd -G developers,support analyst01` |
| `-u UID` | Set explicit UID | `sudo useradd -u 2101 analyst01` |
| `-r` | Create system account | `sudo useradd -r svc-demo` |
| `-M` | Do not create home directory | `sudo useradd -M svc-demo` |
| `-e DATE` | Set account expiry date | `sudo useradd -e 2026-12-31 contractor01` |

### Basic example

```bash
sudo useradd -m -s /bin/bash -c "Lab Analyst" analyst01
id analyst01
getent passwd analyst01
```

**Example output; your values will differ:**

```text
uid=1001(analyst01) gid=1001(analyst01) groups=1001(analyst01)
analyst01:x:1001:1001:Lab Analyst:/home/analyst01:/bin/bash
```

### Real-world example

Create a non-interactive application identity with no home directory:

```bash
sudo useradd -r -M -s "$(command -v nologin)" -c "Demo backup service" backupsvc
```

### Common mistakes and gotchas

- On some distributions or local policies, `useradd` may not create a home directory unless `-m` is specified; use `-m` explicitly in automation.
- Ubuntu’s interactive `adduser` is a higher-level helper; `useradd` is the standard low-level command used across distributions.
- `-G` assigns supplementary groups at creation. Existing group membership is relevant when using `usermod`.
- Do not manually assign UIDs without checking for conflicts.

### Verify it worked

```bash
id analyst01
getent passwd analyst01
getent shadow analyst01 | sudo cut -d: -f1,2
sudo ls -ld /home/analyst01
```

***

## `usermod`

### Purpose

Modifies an existing local user account.

### Syntax

```bash
sudo usermod [OPTIONS] LOGIN
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-aG GROUPS` | Append supplementary groups safely | `sudo usermod -aG developers analyst01` |
| `-G GROUPS` | Replace supplementary groups | `sudo usermod -G support analyst01` |
| `-g GROUP` | Change primary group | `sudo usermod -g developers analyst01` |
| `-s SHELL` | Change login shell | `sudo usermod -s /bin/bash analyst01` |
| `-c TEXT` | Change GECOS/comment | `sudo usermod -c "Analyst, Finance" analyst01` |
| `-d DIR` | Change home path record | `sudo usermod -d /srv/home/analyst01 analyst01` |
| `-m` | Move home contents with `-d` | `sudo usermod -d /srv/home/analyst01 -m analyst01` |
| `-L` | Lock password | `sudo usermod -L analyst01` |
| `-U` | Unlock password | `sudo usermod -U analyst01` |
| `-e DATE` | Set account-expiry date | `sudo usermod -e 2026-12-31 analyst01` |
| `-f DAYS` | Set inactive days after password expiry | `sudo usermod -f 14 analyst01` |

### Basic example

```bash
sudo usermod -c "Lab Analyst, Operations" -s /bin/bash analyst01
getent passwd analyst01
```

**Example output; your values will differ:**

```text
analyst01:x:1001:1001:Lab Analyst, Operations:/home/analyst01:/bin/bash
```

### Real-world example

Add an employee to the `developers` group without removing current memberships:

```bash
sudo usermod -aG developers developer01
```

### Common mistakes and gotchas

- **Critical:** `usermod -G group user` replaces all supplementary groups. Use `usermod -aG group user` to append.
- New group membership requires a new login session, `su - USER`, or `newgrp GROUP` for the user to receive it.
- `-d` without `-m` changes the recorded home path but does not move files.
- Password locking with `-L` does not necessarily stop SSH-key authentication.

### Verify it worked

```bash
id analyst01
getent passwd analyst01
getent group developers
sudo chage -l analyst01
```

***

## `userdel`

### Purpose

Deletes a local user account.

### Syntax

```bash
sudo userdel [OPTIONS] LOGIN
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-r` | Remove home directory and mail spool | `sudo userdel -r analyst01` |
| `-f` | Force removal in limited circumstances | Avoid in normal administration |
| `-Z` | Remove SELinux user mapping where supported | Verify platform support |

### Basic example

```bash
sudo userdel analyst01
getent passwd analyst01 || echo "Account removed"
```

**Example output:**

```text
Account removed
```

### Real-world example

After archiving a former contractor’s home and removing access, delete the account and home:

```bash
sudo userdel -r contractor01
```

### Common mistakes and gotchas

- `userdel` without `-r` leaves the home directory and mail spool behind.
- Do not delete an account before preserving required business data and audit evidence.
- Deleting an account does not automatically remove every file it owns outside its home.
- Do not rapidly reuse a UID; stale files may become owned by the new account numerically.

### Verify it worked

```bash
getent passwd analyst01 || echo "No passwd entry"
getent shadow analyst01 || echo "No shadow entry"
sudo find /home -maxdepth 1 -user analyst01 -ls 2>/dev/null
```

***

## `passwd`

### Purpose

Sets, changes, locks, unlocks, or reports password status for local accounts.

### Syntax

```bash
sudo passwd [OPTIONS] [USER]
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| No option | Change current user password | `passwd` |
| `USER` | Change another user’s password as root | `sudo passwd developer01` |
| `-l` | Lock password | `sudo passwd -l developer01` |
| `-u` | Unlock password | `sudo passwd -u developer01` |
| `-S` | Show password status | `sudo passwd -S developer01` |
| `-e` | Expire password immediately | `sudo passwd -e developer01` |
| `-d` | Delete password hash; generally unsafe | Avoid except controlled recovery policy |

### Basic example

```bash
sudo passwd -S analyst01
```

**Example output; your values will differ:**

```text
analyst01 P 2026-10-20 0 99999 7 -1
```

The status letter varies: `P` commonly indicates a usable password, while `L` indicates a locked password.

### Real-world example

Force a user to change a temporary onboarding password at next password authentication:

```bash
sudo passwd -e developer01
```

### Common mistakes and gotchas

- `passwd -l` locks password authentication; it does not necessarily disable SSH keys or existing sessions.
- `passwd -d` can produce a dangerous empty-password state depending on PAM configuration. Avoid it.
- Never use non-interactive clear-text password handling outside controlled labs.

### Verify it worked

```bash
sudo passwd -S developer01
sudo getent shadow developer01 | cut -d: -f1,2
sudo chage -l developer01
```

***

## `su`

### Purpose

Switches to another user identity.

### Syntax

```bash
su [OPTIONS] [USER]
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-` or `-l` | Start a login shell | `su - developer01` |
| `-c COMMAND` | Run one command as target user | `su - developer01 -c 'id'` |
| `-s SHELL` | Use a specified shell | `su -s /bin/bash developer01` |

### Basic example

```bash
su - developer01 -c 'id; pwd; printf "HOME=%s\n" "$HOME"'
```

**Example output; your values will differ:**

```text
uid=1002(developer01) gid=3001(developers) groups=3001(developers)
 /home/developer01
HOME=/home/developer01
```

### Real-world example

Test whether a user can create files in a shared department directory:

```bash
sudo su - developer01 -c 'touch /srv/projects/dev/test-from-developer01'
```

### Common mistakes and gotchas

- `su user` does not establish a full login environment; prefer `su - user`.
- `su` may require the target user’s password unless run through `sudo`.
- Do not use `su` as a substitute for narrowly scoped sudo policy.

### Verify it worked

```bash
sudo su - developer01 -c 'whoami; id; pwd'
```

***

## `sudo`

### Purpose

Runs a command with privileges authorized by sudo policy.

### Syntax

```bash
sudo [OPTIONS] COMMAND
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-l` | List permitted commands | `sudo -l` |
| `-u USER` | Run as another user | `sudo -u developer01 id` |
| `-i` | Start target user login shell, default root | `sudo -i` |
| `-v` | Refresh authentication timestamp | `sudo -v` |
| `-k` | Invalidate authentication timestamp | `sudo -k` |
| `-n` | Non-interactive; fail instead of prompting | `sudo -n systemctl status ssh` |

### Basic example

```bash
sudo -l
```

**Example output; your values will differ:**

```text
Matching Defaults entries for student on labvm:
    env_reset, mail_badpass

User student may run the following commands on labvm:
    (ALL : ALL) ALL
```

### Real-world example

Run a read-only command as a service user:

```bash
sudo -u www-data id
```

### Common mistakes and gotchas

- `sudo` authorization is policy-based, not merely group-based.
- On Ubuntu, the `sudo` group commonly grants administrative rights; on RHEL-family systems, `wheel` commonly does. Verify `/etc/sudoers` using `visudo -c`.
- Never edit `/etc/sudoers` with a normal editor.
- Avoid broad `NOPASSWD: ALL` rules.

### Verify it worked

```bash
sudo -l
sudo -u root whoami
sudo visudo -c
```

***

## `groupadd`

### Purpose

Creates a local group.

### Syntax

```bash
sudo groupadd [OPTIONS] GROUP
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-g GID` | Set explicit GID | `sudo groupadd -g 3001 developers` |
| `-r` | Create system group | `sudo groupadd -r appsvc` |
| `-f` | Succeed if group already exists | Use carefully in idempotent automation |
| `-o` | Allow non-unique GID with `-g` | Avoid in normal administration |

### Basic example

```bash
sudo groupadd developers
getent group developers
```

**Example output; your values will differ:**

```text
developers:x:3001:
```

### Real-world example

Create a dedicated group for a shared application-data directory:

```bash
sudo groupadd appdata
```

### Common mistakes and gotchas

- A group name must be unique.
- Do not assign duplicate GIDs unless you understand the security and audit consequences.
- Avoid hard-coded GIDs unless your organization deliberately manages them across systems.

### Verify it worked

```bash
getent group developers
sudo getent gshadow developers
```

***

## `groupmod`

### Purpose

Modifies an existing local group.

### Syntax

```bash
sudo groupmod [OPTIONS] GROUP
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-n NEW_NAME` | Rename group | `sudo groupmod -n engineering developers` |
| `-g GID` | Change GID | `sudo groupmod -g 3101 developers` |
| `-o` | Allow non-unique GID with `-g` | Avoid in normal administration |

### Basic example

```bash
sudo groupadd tempteam
sudo groupmod -n projectteam tempteam
getent group projectteam
```

**Example output; your values will differ:**

```text
projectteam:x:3005:
```

### Real-world example

Rename a department group after an approved organizational change, then scan owned files before and after:

```bash
sudo groupmod -n platform developers
```

### Common mistakes and gotchas

- Changing a GID does not automatically change ownership of files elsewhere on the filesystem.
- Renaming a group can break scripts, ACLs, and service configuration that references the old name.
- Avoid changing group IDs casually.

### Verify it worked

```bash
getent group projectteam
getent group tempteam || echo "Old group name absent"
```

***

## `groupdel`

### Purpose

Deletes a local group.

### Syntax

```bash
sudo groupdel [OPTIONS] GROUP
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-f` | Force deletion in limited cases | Avoid unless you understand the consequences |

### Basic example

```bash
sudo groupdel projectteam
getent group projectteam || echo "Group removed"
```

**Example output:**

```text
Group removed
```

### Real-world example

Remove a project group only after removing memberships, ACL dependencies, group-owned shared data, and configuration references.

```bash
sudo groupdel retired-project
```

### Common mistakes and gotchas

- A primary group for an existing user cannot normally be deleted.
- Group deletion does not automatically change existing files with that numeric GID.
- Check `getent group`, `getent passwd`, and relevant filesystem ownership first.

### Verify it worked

```bash
getent group retired-project || echo "Group removed"
sudo find /srv -group retired-project -ls 2>/dev/null
```

***

# Lab 5.1: Company Users

## Task

Create these users:

```text
admin
developer01
developer02
network01
support01
backup01
```

Requirements:

- Each account needs a home directory.
- Each interactive account needs `/bin/bash`.
- Add a useful GECOS/comment field with full name and role.
- `backup01` must be a restricted service-style account: no interactive login.
- Verify with `id`, `getent passwd`, `ls -ld /home/*`, `/etc/passwd`, and `/etc/shadow`.

Attempt the task first. Use only the lab VM.

<details>
<summary><strong>Solution</strong></summary>

Create the accounts:

```bash
sudo useradd -m -s /bin/bash -c "Amina Patel, Systems Administrator" admin
sudo useradd -m -s /bin/bash -c "Devon Lee, Software Developer" developer01
sudo useradd -m -s /bin/bash -c "Morgan Kim, Software Developer" developer02
sudo useradd -m -s /bin/bash -c "Noah Singh, Network Engineer" network01
sudo useradd -m -s /bin/bash -c "Sofia Garcia, Support Engineer" support01
sudo useradd -m -s "$(command -v nologin)" -c "Backup Service Account" backup01
```

`backup01` has a home directory because the lab requires one, but its `nologin` shell blocks ordinary interactive shell access. In production, many service accounts also use `-M` to omit a home directory when the service does not require one.

Set **lab-only** throwaway passwords for the interactive accounts:

```bash
printf '%s\n' \
  'admin:LabOnly-Admin-2026!' \
  'developer01:LabOnly-Dev01-2026!' \
  'developer02:LabOnly-Dev02-2026!' \
  'network01:LabOnly-Net01-2026!' \
  'support01:LabOnly-Support01-2026!' | sudo chpasswd
```

Keep `backup01` without an interactive password:

```bash
sudo passwd -l backup01
```

Verify identities:

```bash
id admin
id developer01
id developer02
id network01
id support01
id backup01
```

**Example output; your values will differ:**

```text
uid=1001(admin) gid=1001(admin) groups=1001(admin)
uid=1002(developer01) gid=1002(developer01) groups=1002(developer01)
uid=1006(backup01) gid=1006(backup01) groups=1006(backup01)
```

Verify account records:

```bash
getent passwd admin developer01 developer02 network01 support01 backup01
```

**Example output; your values will differ:**

```text
admin:x:1001:1001:Amina Patel, Systems Administrator:/home/admin:/bin/bash
developer01:x:1002:1002:Devon Lee, Software Developer:/home/developer01:/bin/bash
developer02:x:1003:1003:Morgan Kim, Software Developer:/home/developer02:/bin/bash
network01:x:1004:1004:Noah Singh, Network Engineer:/home/network01:/bin/bash
support01:x:1005:1005:Sofia Garcia, Support Engineer:/home/support01:/bin/bash
backup01:x:1006:1006:Backup Service Account:/home/backup01:/usr/sbin/nologin
```

Verify homes:

```bash
sudo ls -ld /home/admin /home/developer01 /home/developer02 /home/network01 /home/support01 /home/backup01
```

Inspect `/etc/passwd` entries:

```bash
grep -E '^(admin|developer01|developer02|network01|support01|backup01):' /etc/passwd
```

Inspect only shadow usernames and password markers, not full hashes:

```bash
sudo awk -F: '/^(admin|developer01|developer02|network01|support01|backup01):/ {
  marker=$2
  if (marker ~ /^!/) status="locked"
  else if (marker == "*") status="no usable password"
  else status="hash present"
  print $1 ": " status
}' /etc/shadow
```

</details>

## Expected-results checklist

- [ ] Six accounts exist in `getent passwd`.
- [ ] Five interactive users have `/bin/bash`.
- [ ] `backup01` has a `nologin` path shown by `getent passwd backup01`.
- [ ] Every user has `/home/USERNAME`.
- [ ] `id USER` shows a unique UID and a primary group.
- [ ] Shadow inspection shows no real password hashes on screen.

***

# Lab 5.2: Department Groups

## Task

Create groups:

```text
developers
network
support
management
backup
```

Use this user-to-group assignment matrix:

| User | Primary group | Supplementary groups |
|---|---|---|
| `admin` | `admin` | `management`, distro admin group |
| `developer01` | `developers` | — |
| `developer02` | `developers` | — |
| `network01` | `network` | — |
| `support01` | `support` | — |
| `backup01` | `backup` | — |

Requirements:

1. Give `admin` sudo rights using the correct distro mechanism.
2. Demonstrate safe append behavior with `usermod -aG`.
3. Demonstrate the dangerous overwrite behavior of `usermod -G` without `-a` on a throwaway user.
4. Create `/srv/projects/dev`, group-owned by `developers`, with the setgid bit.
5. Prove `developer01` and `developer02` can collaborate, while `support01` cannot.

<details>
<summary><strong>Solution</strong></summary>

Create the department groups:

```bash
sudo groupadd developers
sudo groupadd network
sudo groupadd support
sudo groupadd management
sudo groupadd backup
```

Set each departmental primary group:

```bash
sudo usermod -g developers developer01
sudo usermod -g developers developer02
sudo usermod -g network network01
sudo usermod -g support support01
sudo usermod -g backup backup01
```

Add `admin` to management:

```bash
sudo usermod -aG management admin
```

Grant `admin` sudo access.

Ubuntu/Debian-family:

```bash
sudo usermod -aG sudo admin
```

RHEL/Rocky/AlmaLinux:

```bash
sudo usermod -aG wheel admin
```

The `wheel` group is the conventional RHEL administrative group when enabled in sudoers. Red Hat’s administration documentation demonstrates adding users with `usermod --append -G wheel USER`. [docs.redhat](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_basic_system_settings/managing-users-and-groups_configuring-basic-system-settings)

Verify the distro’s active administrator group policy:

```bash
sudo grep -E '^[[:space:]]*%((sudo)|(wheel))[[:space:]]+' /etc/sudoers
```

Verify membership:

```bash
id admin
id developer01
id developer02
id network01
id support01
id backup01
```

**Example output; your values will differ:**

```text
uid=1001(admin) gid=1001(admin) groups=1001(admin),27(sudo),3004(management)
uid=1002(developer01) gid=3001(developers) groups=3001(developers)
uid=1003(developer02) gid=3001(developers) groups=3001(developers)
```

### Demonstrate safe append versus overwrite

Create a throwaway user:

```bash
sudo useradd -m -s /bin/bash -c "Throwaway Group Test" groupdemo
sudo usermod -aG developers,support groupdemo
id groupdemo
```

**Example output:**

```text
uid=1007(groupdemo) gid=1007(groupdemo) groups=1007(groupdemo),3001(developers),3003(support)
```

Now demonstrate the mistake:

```bash
sudo usermod -G network groupdemo
id groupdemo
```

**Example output:**

```text
uid=1007(groupdemo) gid=1007(groupdemo) groups=1007(groupdemo),3002(network)
```

`developers` and `support` were removed because `-G` replaces the supplementary list.

Restore intended groups:

```bash
sudo usermod -aG developers,support groupdemo
id groupdemo
```

Remove the throwaway account:

```bash
sudo userdel -r groupdemo
```

### Create the shared project directory

```bash
sudo mkdir -p /srv/projects/dev
sudo chown root:developers /srv/projects/dev
sudo chmod 2770 /srv/projects/dev
sudo ls -ld /srv/projects/dev
```

**Example output:**

```text
drwxrws--- 2 root developers 4096 Oct 20 14:30 /srv/projects/dev
```

Interpretation:

- `root:developers`: root owns the directory; `developers` owns its group.
- `rwx` for owner and group: both can work inside it.
- `---` for others: nonmembers cannot access it.
- `s` in the group execute position: the **setgid bit** makes new files inherit the `developers` group.

Prove collaboration:

```bash
sudo -u developer01 touch /srv/projects/dev/from-developer01.txt
sudo -u developer02 sh -c 'printf "developer02 contribution\n" >> /srv/projects/dev/from-developer01.txt'
sudo ls -l /srv/projects/dev/from-developer01.txt
```

**Example output:**

```text
-rw-r--r-- 1 developer01 developers 24 Oct 20 14:31 /srv/projects/dev/from-developer01.txt
```

Test support denial:

```bash
sudo -u support01 touch /srv/projects/dev/should-not-create.txt
```

**Expected output:**

```text
touch: cannot touch '/srv/projects/dev/should-not-create.txt': Permission denied
```

For group collaboration, ensure a group-friendly umask or explicit modes when creating files. A default `umask 0022` can cause a developer’s new file to be group-readable but not group-writable. Test and adjust intentionally rather than globally weakening permissions.

Verify group records:

```bash
getent group developers network support management backup
sudo getent gshadow developers network support management backup
```

Inspect local records:

```bash
grep -E '^(developers|network|support|management|backup):' /etc/group
sudo grep -E '^(developers|network|support|management|backup):' /etc/gshadow
```

</details>

## Expected-results checklist

- [ ] Five department groups exist.
- [ ] Each departmental user has the intended primary group.
- [ ] `admin` belongs to `management` and `sudo` or `wheel`, as appropriate.
- [ ] The overwrite behavior of `usermod -G` was observed only on `groupdemo`.
- [ ] `/srv/projects/dev` displays group `developers` and the setgid `s` bit.
- [ ] Both developers can create or modify authorized files.
- [ ] `support01` receives permission denied.
- [ ] Group membership was verified with `id`, `groups`, `getent group`, `/etc/group`, and `/etc/gshadow`.

***

# Lab 5.3: User Lifecycle

## Task

Using a dedicated account named `lifecycle01`, practice each stage:

1. Create with explicit options.
2. Modify shell, comment, home, and groups.
3. Lock and inspect shadow behavior.
4. Unlock.
5. Expire the account and password separately.
6. Delete it safely, distinguish `userdel` from `userdel -r`, and locate orphaned files.

<details>
<summary><strong>Solution</strong></summary>

## 1. Create

```bash
sudo useradd \
  -m \
  -s /bin/bash \
  -c "Lifecycle Lab User, Temporary Account" \
  -G developers \
  lifecycle01
```

Set a lab-only password:

```bash
printf '%s\n' 'lifecycle01:LabOnly-Lifecycle-2026!' | sudo chpasswd
```

Verify:

```bash
id lifecycle01
getent passwd lifecycle01
sudo passwd -S lifecycle01
sudo chage -l lifecycle01
```

## 2. Modify

Create a new destination parent:

```bash
sudo mkdir -p /srv/home
```

Change shell, comment, home directory, and append another group:

```bash
sudo usermod \
  -s /bin/bash \
  -c "Lifecycle Lab User, Modified Account" \
  -d /srv/home/lifecycle01 \
  -m \
  -aG support \
  lifecycle01
```

Verify:

```bash
id lifecycle01
getent passwd lifecycle01
sudo ls -ld /srv/home/lifecycle01
```

## 3. Lock

Lock password authentication:

```bash
sudo passwd -l lifecycle01
sudo passwd -S lifecycle01
```

Inspect the shadow marker without showing the hash:

```bash
sudo awk -F: '$1=="lifecycle01" {
  if ($2 ~ /^!/) print "Password field begins with !: locked"
  else print "Password field is not visibly locked"
}' /etc/shadow
```

Alternative lock command:

```bash
sudo usermod -L lifecycle01
```

Important: this locks the password credential. It does **not** necessarily block:

- Existing sessions.
- Scheduled jobs already running as that user.
- SSH public-key login if SSH policy still allows it.
- `sudo -u lifecycle01` by a privileged administrator.

## 4. Unlock

```bash
sudo passwd -u lifecycle01
sudo passwd -S lifecycle01
```

Alternative:

```bash
sudo usermod -U lifecycle01
```

Verify:

```bash
sudo awk -F: '$1=="lifecycle01" {print $1 ": shadow field is present; inspect status via passwd -S"}' /etc/shadow
sudo passwd -S lifecycle01
```

## 5. Expire

### Account expiration

Set an account expiration date:

```bash
sudo usermod -e 2026-12-31 lifecycle01
sudo chage -l lifecycle01
```

Set the account to expire immediately for the lab:

```bash
sudo chage -E 0 lifecycle01
sudo chage -l lifecycle01
```

Restore it so later checks work:

```bash
sudo chage -E -1 lifecycle01
```

Account expiry blocks account use after the specified date.

### Password expiration

Force password change at next password authentication:

```bash
sudo chage -d 0 lifecycle01
sudo chage -l lifecycle01
```

Set a maximum password age:

```bash
sudo chage -M 30 -W 7 lifecycle01
sudo chage -l lifecycle01
```

Difference:

| Control | Effect |
|---|---|
| Account expiry (`usermod -e`, `chage -E`) | Account becomes unavailable after date |
| Password expiry (`chage -d 0`, `chage -M`) | Password must be changed or ages out; other auth mechanisms may differ |

## 6. Delete

Before deletion, search for files owned by the user in controlled locations:

```bash
sudo find /srv/home /srv/projects /home -xdev -user lifecycle01 -ls 2>/dev/null
```

Delete account **without** `-r`:

```bash
sudo userdel lifecycle01
getent passwd lifecycle01 || echo "Passwd entry removed"
sudo ls -ld /srv/home/lifecycle01
```

The home remains because `-r` was not used.

Find orphaned ownership by numeric identity. First record the former UID before deletion:

```bash
# Run this before deletion in a future real-world offboarding:
# id -u lifecycle01
```

For a broad audit, use carefully because searching `/` can be slow and cross filesystems:

```bash
sudo find / -xdev -nouser -ls 2>/dev/null
```

Do not immediately reuse the former UID. First remediate or archive files that retain it. To finish this lab, remove the preserved home only after inspection:

```bash
sudo rm -ri /srv/home/lifecycle01
```

</details>

## Expected-results checklist

- [ ] `lifecycle01` was created with explicit home, shell, comment, and group settings.
- [ ] `usermod` changed the account fields and moved the home with `-m`.
- [ ] Password lock state was verified using `passwd -S` and a safe shadow-marker check.
- [ ] Password locking was distinguished from full account disablement.
- [ ] Account expiration and password expiration were tested as separate controls.
- [ ] `userdel` left the home behind when used without `-r`.
- [ ] Orphaned files were discussed and searched with controlled `find` commands.

***

# Advanced Server Administration

## Limited sudo delegation

Grant `support01` permission to restart only one service. Replace `ssh` with a service that exists in your lab, and verify the exact systemctl path:

```bash
command -v systemctl
systemctl list-unit-files --type=service | grep -E '^ssh|^sshd'
```

Create a drop-in using `visudo`:

```bash
sudo visudo -f /etc/sudoers.d/support01-ssh
```

Add this line, adapting the service unit and `systemctl` path if required:

```text
support01 ALL=(root) /usr/bin/systemctl restart ssh.service
```

Validate:

```bash
sudo visudo -cf /etc/sudoers.d/support01-ssh
sudo -u support01 sudo -l
```

Test only the authorized command:

```bash
sudo -u support01 sudo /usr/bin/systemctl restart ssh.service
```

Avoid granting unrestricted shell commands, editors, package managers, wildcards, or `ALL` when the requirement is narrow. Sudoers rules can apply to individual users and groups and are normally managed in `/etc/sudoers` plus `/etc/sudoers.d/`. [docs.redhat](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_basic_system_settings/managing-sudo-access_configuring-basic-system-settings)

## Password-aging policy

For a server fleet:

- Define policy in configuration management, not manual one-off edits.
- Use `chage` to apply account-specific exceptions.
- Review `/etc/login.defs` for default aging values.
- Understand that PAM controls complexity, reuse history, lockout behavior, and much of real authentication policy.
- Prefer SSH keys for administrative remote access, protect keys with passphrases/hardware-backed mechanisms, and disable password SSH where policy permits.
- Do not depend on password expiry as the sole control for privileged access.

Example:

```bash
sudo chage -M 90 -m 1 -W 14 admin
sudo chage -l admin
```

## Service accounts

A service account should have only the permissions it needs:

```bash
sudo useradd -r -M -s "$(command -v nologin)" -c "Demo application service" appsvc
sudo passwd -l appsvc
id appsvc
getent passwd appsvc
```

Typical service-account characteristics:

- System UID selected by `useradd -r`.
- No interactive shell.
- No password authentication.
- No unnecessary home directory.
- Restricted file ownership.
- Narrow sudo rights only if unavoidable.

## Auditing identities and access

Check login history:

```bash
last -F | head -n 20
lastlog | head -n 20
who
w
```

Inspect authentication and sudo events on Ubuntu/Debian when rsyslog logs them:

```bash
sudo grep -E 'sudo:|sshd' /var/log/auth.log | tail -n 50
```

On RHEL-family systems:

```bash
sudo grep -E 'sudo:|sshd' /var/log/secure | tail -n 50
```

On systems using journald:

```bash
sudo journalctl --since "today" | grep -E 'sudo|sshd'
```

Use the appropriate log source for your system. Availability depends on journald, rsyslog, auditd, SSH configuration, and retention settings.

## Consistency checks and backups

Back up identity files before controlled maintenance:

```bash
sudo install -d -m 0700 /root/identity-backups
sudo cp -a /etc/passwd /etc/shadow /etc/group /etc/gshadow /root/identity-backups/
sudo ls -l /root/identity-backups
```

Run checks:

```bash
sudo pwck -r
sudo grpck -r
```

`-r` performs read-only checks rather than attempting repairs. Investigate findings; do not blindly “fix” identity databases on a production system.

## Security checklist

- [ ] Exactly one intended UID 0 account, normally `root`.
- [ ] No empty password fields in `/etc/shadow`.
- [ ] No shared human accounts; each person has an attributable identity.
- [ ] Privileged access is granted through groups and least-privilege sudo rules.
- [ ] Service accounts use `nologin` or equivalent and have no unnecessary passwords.
- [ ] Former employee accounts are locked, expired, and removed under policy.
- [ ] Orphaned UIDs/GIDs are investigated before reuse.
- [ ] `/etc/sudoers` and `/etc/sudoers.d` are edited only via `visudo`.
- [ ] Identity changes are logged through change management or configuration management.
- [ ] SSH authentication policy is reviewed regularly.

## Offboarding runbook

1. Record identity, UID, groups, home, sudo rules, SSH keys, running sessions, and file ownership.
2. Lock the password:
   ```bash
   sudo passwd -l USER
   ```
3. Expire the account immediately:
   ```bash
   sudo chage -E 0 USER
   ```
4. Remove privileged/supplementary access:
   ```bash
   sudo gpasswd -d USER sudo 2>/dev/null || true
   sudo gpasswd -d USER wheel 2>/dev/null || true
   ```
5. Remove or disable authorized SSH keys:
   ```bash
   sudo rm -f /home/USER/.ssh/authorized_keys
   ```
   Archive evidence first when policy requires it.
6. Identify and end sessions:
   ```bash
   who
   loginctl user-status USER 2>/dev/null || true
   sudo pkill -KILL -u USER
   ```
   Use process termination carefully; preserve required work and coordinate with HR/security policy.
7. Archive home and business data:
   ```bash
   sudo tar -C /home -czf /root/USER-home-archive-$(date +%F).tar.gz USER
   ```
8. Search for files outside the home:
   ```bash
   sudo find / -xdev -user USER -ls 2>/dev/null
   ```
9. Remove account according to retention policy:
   ```bash
   sudo userdel -r USER
   ```
10. Review audit logs and verify no remaining access paths.

***

# Real-World Scenarios

## Onboard a developer

**Brief:** A new developer needs a personal account and access to the development project area.

```bash
sudo useradd -m -s /bin/bash -c "Taylor Chen, Software Developer" developer03
printf '%s\n' 'developer03:LabOnly-Dev03-2026!' | sudo chpasswd
sudo usermod -g developers developer03
sudo chage -d 0 developer03
id developer03
sudo -u developer03 touch /srv/projects/dev/onboarding-test.txt
```

Why: individual accountability, group-based collaboration, and forced first-password change provide safer access than shared accounts.

## Create a 30-day contractor account

**Brief:** A contractor requires temporary access without indefinite persistence.

```bash
sudo useradd -m -s /bin/bash -c "Contractor, Ends in 30 days" -e "$(date -d '+30 days' +%F)" contractor01
printf '%s\n' 'contractor01:LabOnly-Contractor-2026!' | sudo chpasswd
sudo usermod -aG developers contractor01
sudo chage -l contractor01
```

Why: account expiry reduces the chance that temporary access becomes permanent. Confirm the `date` syntax on non-GNU systems if portability matters.

## Offboard an employee

**Brief:** A developer leaves immediately; preserve data and revoke access first.

```bash
sudo passwd -l developer02
sudo chage -E 0 developer02
sudo pkill -KILL -u developer02
sudo tar -C /home -czf /root/developer02-home-archive-$(date +%F).tar.gz developer02
sudo find / -xdev -user developer02 -ls 2>/dev/null
```

Why: deletion is last. Locking, expiring, session termination, archiving, and audit preservation come first.

## Create a backup service identity

**Brief:** A backup process needs file access but no human login.

```bash
sudo useradd -r -M -s "$(command -v nologin)" -c "Backup processing service" backupsvc
sudo passwd -l backupsvc
id backupsvc
getent passwd backupsvc
```

Why: a non-interactive identity prevents the service from becoming a normal remote-login account.

## Investigate unexpected sudo use

**Brief:** Determine who ran sudo today.

Ubuntu/Debian-style log:

```bash
sudo grep 'sudo:' /var/log/auth.log | tail -n 100
```

RHEL-style log:

```bash
sudo grep 'sudo:' /var/log/secure | tail -n 100
```

Journald:

```bash
sudo journalctl --since "today" | grep 'sudo'
```

Then correlate username, terminal, command, time, source address, and approved maintenance windows.

***

# Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| `user is not in the sudoers file` | No applicable sudo rule or wrong admin group | Check `id USER`, `sudo -l`, and `/etc/sudoers` via `visudo`; add to `sudo` on Ubuntu or `wheel` on RHEL only when policy allows |
| `useradd: group 'X' exists` | A requested primary-group name conflicts with local state | Inspect with `getent group X`; choose existing group via `-g` or a unique user/group name |
| `su: Authentication failure` | Wrong target password, locked account, PAM restriction, or root policy | Use `sudo su - USER` as authorized admin; inspect `passwd -S USER`, `chage -l USER`, and auth logs |
| User added to group but still denied | Existing session lacks refreshed group list | Log out/in, use `su - USER`, or start a new SSH session; verify `id USER` |
| `usermod -G` removed memberships | `-G` replaces supplementary groups | Restore intended groups with `usermod -aG group1,group2 USER`; document the incident |
| Home directory has wrong ownership | Manual copy/move or incorrect `usermod` usage | Check `ls -ld /home/USER`; correct cautiously with `sudo chown -R USER:PRIMARYGROUP /home/USER` only after verifying scope |
| `userdel` fails because user is logged in | Active processes or sessions | Investigate with `who`, `w`, `ps -u USER`; coordinate, then terminate sessions before retrying |
| Broken sudoers file | Direct editing introduced syntax error | Keep a root console open; use `visudo -c`; repair through `visudo` or a recovery console; use `/etc/sudoers.d` with validated files |
| `sudo: command not found` | `sudo` package absent, especially in minimal containers | Become root through approved console method, install `sudo` with the distro package manager, then configure policy |
| Shared directory blocks group writes | Missing setgid/group-write bit or restrictive umask | Inspect `ls -ld`; use appropriate group ownership and `chmod 2770 DIR`; consider a group-friendly umask or ACLs |
| `useradd` did not create a home | Distro/local defaults or omitted `-m` | Create explicitly with `-m`; inspect `CREATE_HOME` in `/etc/login.defs` |
| SSH key still works after `passwd -l` | Password lock affects password hash only | Expire account, remove `authorized_keys`, adjust SSH policy, and terminate active sessions as needed |

***

# One-Page Command Cheat Sheet

```bash
# Identity
whoami
id USER
id -u USER
id -Gn USER
groups USER
who
w

# Lookup local/NSS identity records
getent passwd USER
getent shadow USER          # usually via sudo
getent group GROUP
sudo getent gshadow GROUP

# Create groups
sudo groupadd GROUP
sudo groupadd -g GID GROUP
sudo groupadd -r SYSTEM_GROUP

# Modify/delete groups
sudo groupmod -n NEW OLD
sudo groupmod -g NEW_GID GROUP
sudo groupdel GROUP

# Create users
sudo useradd -m -s /bin/bash -c "Name, Role" USER
sudo useradd -m -g PRIMARY -G supplementary1,supplementary2 USER
sudo useradd -r -M -s "$(command -v nologin)" SERVICE_USER

# Modify users
sudo usermod -aG GROUP USER       # append safely
sudo usermod -G GROUP1,GROUP2 USER # replaces supplementary groups
sudo usermod -g PRIMARY USER
sudo usermod -s /bin/bash USER
sudo usermod -c "Name, Role" USER
sudo usermod -d /new/home -m USER
sudo usermod -L USER
sudo usermod -U USER
sudo usermod -e YYYY-MM-DD USER

# Passwords and aging
sudo passwd USER
sudo passwd -S USER
sudo passwd -l USER
sudo passwd -u USER
sudo passwd -e USER
sudo chage -l USER
sudo chage -d 0 USER
sudo chage -M 90 -W 14 USER
sudo chage -E YYYY-MM-DD USER

# Delete users
sudo userdel USER
sudo userdel -r USER

# Privilege and identity switching
sudo -l
sudo -u USER COMMAND
sudo -i
su - USER
sudo su - USER

# Sudo policy
sudo visudo
sudo visudo -f /etc/sudoers.d/NAME
sudo visudo -c

# Audit and validation
last -F
lastlog
sudo journalctl --since "today" | grep -E 'sudo|sshd'
sudo pwck -r
sudo grpck -r
```

***

# Quiz

1. What numeric UID identifies root?
2. What is the difference between a primary group and a supplementary group?
3. Which `/etc/passwd` field specifies the login shell?
4. What does a leading `!` in a shadow password field generally indicate?
5. What dangerous effect can `usermod -G developers USER` have?
6. Which command safely appends `USER` to the `developers` group?
7. What is the difference between `su USER` and `su - USER`?
8. Does `passwd -l USER` necessarily block SSH public-key login?
9. Which mode enables setgid inheritance on a group-shared directory: `0770` or `2770`?
10. Why should `/etc/sudoers` be edited through `visudo`?

## Answers

1. UID `0`.
2. A user has one primary group recorded in `/etc/passwd`; supplementary groups provide additional memberships.
3. Field 7.
4. The password credential is locked by prefixing the existing hash with `!`.
5. It replaces the supplementary-group list, potentially removing existing memberships.
6. `sudo usermod -aG developers USER`.
7. `su - USER` starts a login-style environment; `su USER` retains more of the current environment.
8. No. It locks password authentication but may not block SSH keys, existing sessions, or other mechanisms.
9. `2770`; the leading `2` sets setgid on the directory.
10. It locks and validates sudoers syntax, reducing the chance of losing administrative access.

***

# Cleanup

> Read the script before running it. It removes all accounts, groups, lab homes, the shared lab directory, and the sudo drop-in created in this tutorial. Run it only in the disposable lab VM.

Create the cleanup file:

```bash
cat > ~/cleanup-linux-users-groups-lab.sh <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

users=(
  admin
  developer01
  developer02
  network01
  support01
  backup01
  lifecycle01
  developer03
  contractor01
  backupsvc
  appsvc
  analyst01
  groupdemo
)

groups=(
  developers
  network
  support
  management
  backup
  appdata
  tempteam
  projectteam
  retired-project
)

echo "This removes Linux Users & Groups lab artifacts only."
echo "Reviewing target accounts:"
for user in "${users[@]}"; do
  getent passwd "$user" || true
done

echo
echo "Reviewing target groups:"
for group in "${groups[@]}"; do
  getent group "$group" || true
done

echo
read -r -p "Type REMOVE-LAB to continue: " confirmation
if [[ "$confirmation" != "REMOVE-LAB" ]]; then
  echo "Cleanup cancelled."
  exit 1
fi

sudo rm -f /etc/sudoers.d/support01-ssh
sudo visudo -c

for user in "${users[@]}"; do
  if getent passwd "$user" >/dev/null; then
    echo "Removing user: $user"
    sudo userdel -r "$user" || echo "Could not fully remove $user; inspect active processes or retained files."
  fi
done

sudo rm -rf /srv/projects/dev
sudo rmdir /srv/projects 2>/dev/null || true
sudo rmdir /srv/home 2>/dev/null || true

for group in "${groups[@]}"; do
  if getent group "$group" >/dev/null; then
    echo "Removing group: $group"
    sudo groupdel "$group" || echo "Could not remove $group; it may still be a primary group or in use."
  fi
done

echo "Remaining matching identities:"
getent passwd | awk -F: '$1 ~ /^(admin|developer01|developer02|network01|support01|backup01|lifecycle01|developer03|contractor01|backupsvc|appsvc|analyst01|groupdemo)$/ {print}'
getent group | awk -F: '$1 ~ /^(developers|network|support|management|backup|appdata|tempteam|projectteam|retired-project)$/ {print}'

echo "Cleanup complete. Review any warnings above."
EOF
```

Inspect it:

```bash
less ~/cleanup-linux-users-groups-lab.sh
```

Make it executable and run it only after review:

```bash
chmod 700 ~/cleanup-linux-users-groups-lab.sh
~/cleanup-linux-users-groups-lab.sh
```

# Next Topics

- Linux permissions: ownership, `chmod`, symbolic and numeric modes.
- ACLs: `getfacl`, `setfacl`, default ACLs, and shared-directory patterns.
- PAM: authentication modules, password policy, lockouts, and MFA integration.
- Centralized identity: LDAP, Active Directory integration, SSSD, Kerberos, and enterprise access control.

What part of the lifecycle lab would you explain first to a teammate who believes that `passwd -l` completely disables an account?
