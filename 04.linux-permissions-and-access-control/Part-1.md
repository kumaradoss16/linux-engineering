# Linux Permissions & Access Control: From Beginner to Server Administration

## Introduction

This tutorial continues the Linux Users & Groups material and teaches permissions, ownership, ACLs, capabilities, special permission bits, and least privilege through practical commands.

Run all labs only on a disposable Ubuntu 22.04/24.04, Debian, Fedora, Rocky, AlmaLinux, or RHEL VM/container. Take a VM snapshot first. Never run the destructive portions on a production server.

Commands that modify system state require `sudo`. Your usernames, UIDs, GIDs, timestamps, package versions, and output will differ from the examples.

## Lab safety

Before beginning:

```bash
whoami
id
pwd
sudo -v
```

Create a working directory:

```bash
mkdir -p ~/permissions-lab
cd ~/permissions-lab
```

Before using destructive commands, inspect the target:

```bash
pwd
ls -lah
```

Important warnings:

- Never use `chmod -R 777` on a production tree.
- Never use `chown -R` or `chmod -R` without verifying the exact path.
- Do not modify `/tmp`, `/var`, `/opt`, or `/srv` on production unless the change is authorized.
- Do not grant capabilities or SUID to untrusted binaries.
- Use `visudo` for sudoers changes.
- Passwords in this tutorial are lab-only examples.

***

# Part 1: Linux Permission Basics

## Users, groups, and ownership

A Linux process runs with a user identity and one or more group identities. Files and directories have an owning user and group. The kernel compares those identities with the permission bits attached to the object.

Inspect your identity:

```bash
whoami
id
groups
```

Example output; your values will differ:

```text
student
uid=1000(student) gid=1000(student) groups=1000(student),27(sudo)
student sudo
```

### Users

A user account identifies a person, service, or system component.

Examples:

```bash
getent passwd "$(whoami)"
getent passwd root
getent passwd www-data 2>/dev/null || true
```

Example output:

```text
student:x:1000:1000:Student User:/home/student:/bin/bash
root:x:0:0:root:/root:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

### Groups

A group is a collection of users used for access control.

```bash
getent group sudo 2>/dev/null || true
getent group wheel 2>/dev/null || true
```

Ubuntu commonly uses the `sudo` group for administrative access. RHEL-family systems commonly use `wheel`. Always verify the active policy on the system.

### Primary versus supplementary groups

Each user has:

- One primary group, stored in `/etc/passwd`.
- Zero or more supplementary groups, stored in `/etc/group` and directory-service databases.

Inspect both:

```bash
id "$(whoami)"
id -gn "$(whoami)"
id -Gn "$(whoami)"
```

The primary group is often used as the group owner for newly created files. Supplementary groups grant additional access to shared resources.

### File owner and group owner

Create a test file:

```bash
cd ~/permissions-lab
printf 'permission lab\n' > sample.txt
ls -l sample.txt
```

Example output:

```text
-rw-r--r-- 1 student student 15 Oct 21 18:10 sample.txt
```

This means:

- Owner: `student`
- Group owner: `student`
- Size: `15` bytes
- Permissions: `rw-r--r--`

Ownership matters because the same permission bits can have different effects depending on which user accesses the file.

***

## Read, write, and execute

### Files

For ordinary files:

| Permission | Meaning |
|---|---|
| Read `r` | Read file contents |
| Write `w` | Modify or truncate contents |
| Execute `x` | Execute the file as a program, if its contents are executable |

A file containing a shell script needs both readable content for the interpreter and execute permission for direct execution.

Create a script:

```bash
cat > hello.sh <<'EOF'
#!/usr/bin/env bash
printf 'Hello from the permission lab\n'
EOF
```

Initially inspect it:

```bash
ls -l hello.sh
```

Make it executable:

```bash
chmod u+x hello.sh
./hello.sh
```

Expected output:

```text
Hello from the permission lab
```

### Directories

Directory permissions have different meanings:

| Permission | Meaning on a directory |
|---|---|
| Read `r` | List directory entries |
| Write `w` | Create, delete, or rename entries, subject to directory execute permission |
| Execute `x` | Enter/traverse the directory and access known names inside it |

A user generally needs `x` on every parent directory in a path:

```text
/ -> /srv -> /srv/project -> /srv/project/file.txt
```

Without directory execute permission, a user may see a directory name but cannot enter it or access its contents.

Create a test directory:

```bash
mkdir directory-test
touch directory-test/file.txt
chmod 600 directory-test
ls -ld directory-test
```

Try to enter it:

```bash
cd directory-test
```

Expected result:

```text
bash: cd: directory-test: Permission denied
```

Return to the lab directory:

```bash
cd ..
```

Restore usable lab permissions:

```bash
chmod 700 directory-test
```

***

## Permission notation

A mode such as:

```text
rwxr-xr--
```

has three permission triplets:

```text
rwx r-x r--
 |   |   |
 |   |   +-- others
 |   +------ group
 +---------- owner
```

The equivalent numeric mode is `754`.

### Permission calculation

| Permission | Symbol | Numeric value |
|---|---:|---:|
| Read | `r` | 4 |
| Write | `w` | 2 |
| Execute | `x` | 1 |
| No permission | `-` | 0 |

Calculate each triplet by adding values:

| Triplet | Calculation | Numeric |
|---|---:|---:|
| `rwx` | 4 + 2 + 1 | 7 |
| `rw-` | 4 + 2 + 0 | 6 |
| `r-x` | 4 + 0 + 1 | 5 |
| `r--` | 4 + 0 + 0 | 4 |
| `---` | 0 + 0 + 0 | 0 |

Therefore:

```text
rwxr-xr-- = 754
```

### Common modes

| Mode | Owner | Group | Others | Typical use |
|---:|---|---|---|---|
| `644` | `rw-` | `r--` | `r--` | Publicly readable regular file |
| `600` | `rw-` | `---` | `---` | Private key or secret file |
| `700` | `rwx` | `---` | `---` | Private directory |
| `750` | `rwx` | `r-x` | `---` | Application or team directory |
| `755` | `rwx` | `r-x` | `r-x` | Public executable or directory |
| `770` | `rwx` | `rwx` | `---` | Private team directory |
| `775` | `rwx` | `rwx` | `r-x` | Shared group directory |
| `777` | `rwx` | `rwx` | `rwx` | Usually unsafe |

`777` allows every user to read, modify, and enter the object. On a directory, users may create, replace, rename, or delete entries. Use ownership, groups, ACLs, setgid, and sticky-bit controls instead.

***

## How Linux checks permissions

Linux generally evaluates:

1. If the process is root or has an applicable capability, privileged rules may apply.
2. If the effective UID matches the file owner, owner permissions are used.
3. Otherwise, if the process belongs to the file’s group, group permissions are used.
4. Otherwise, others permissions are used.

The owner, group, and others classes are not cumulatively added. If the user matches the owner, Linux does not also add group permissions.

Inspect identity and metadata together:

```bash
whoami
id
groups
ls -l sample.txt
ls -ld directory-test
stat sample.txt
```

***

# Part 2: Core Commands

## `ls`

### Purpose

Inspects file names, permissions, ownership, sizes, timestamps, and directory contents.

### Syntax

```bash
ls [options] [file_or_directory]
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-l` | Long listing | `ls -l file.txt` |
| `-a` | Include hidden files | `ls -la` |
| `-h` | Human-readable sizes | `ls -lh` |
| `-d` | Show directory itself | `ls -ld /srv/shared` |
| `-R` | Recursive listing | `ls -lR /opt/company` |
| `-i` | Show inode number | `ls -li file.txt` |
| `-n` | Show numeric UID/GID | `ls -ln file.txt` |

### Beginner example

```bash
cd ~/permissions-lab
ls -l
ls -la
ls -ld .
```

Expected output; your values will differ:

```text
-rw-r--r-- 1 student student 15 Oct 21 18:10 sample.txt
-rwxr-xr-x 1 student student 83 Oct 21 18:11 hello.sh
drwxr-xr-x 3 student student 42 Oct 21 18:12 .
```

### Reading a listing

```text
-rwxr-x--- 1 alice developers 1024 Oct 21 10:00 deploy.sh
```

| Part | Meaning |
|---|---|
| `-` | Regular file |
| `rwx` | Owner permissions |
| `r-x` | Group permissions |
| `---` | Others permissions |
| `1` | Hard-link count |
| `alice` | Owner |
| `developers` | Group owner |
| `1024` | Size in bytes |
| `Oct 21 10:00` | Modification time |
| `deploy.sh` | Name |

### Server-administration example

Inspect a shared directory without listing its contents:

```bash
sudo ls -ld /srv/shared
```

Inspect numeric ownership when an account was deleted:

```bash
sudo ls -ln /srv/shared
```

A numeric owner or group indicates that the current name-service configuration cannot map the UID or GID to a name.

### Common mistakes

- Using `ls -l DIRECTORY` when you meant `ls -ld DIRECTORY`.
- Assuming `ls -R` is safe and practical on a large filesystem.
- Parsing `ls` output in scripts. Use `find`, `stat`, or another structured interface instead.
- Forgetting hidden files such as `.ssh`.

### Verification

```bash
stat sample.txt
```

***

## `stat`

### Purpose

Displays detailed file metadata, including permissions, ownership, inode, timestamps, and allocated blocks.

### Syntax

```bash
stat [options] file
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-c FORMAT` | Use a custom output format | `stat -c "%A %a %U %G %n" file.txt` |
| `-f` | Show filesystem status | `stat -f /srv` |
| `-L` | Follow symbolic links | `stat -L link` |
| `-t` | Terse output | `stat -t file.txt` |

### Beginner example

```bash
stat sample.txt
```

Expected output; your values will differ:

```text
  File: sample.txt
  Size: 15         Blocks: 8          IO Block: 4096   regular file
Device: 8,2   Inode: 524900      Links: 1
Access: (0644/-rw-r--r--)  Uid: ( 1000/ student)   Gid: ( 1000/ student)
Access: 2026-10-21 18:10:00.000000000 +0530
Modify: 2026-10-21 18:10:00.000000000 +0530
Change: 2026-10-21 18:10:00.000000000 +0530
```

### Custom format

```bash
stat -c "%A %a %U %G %n" sample.txt
```

Expected output:

```text
-rw-r--r-- 644 student student sample.txt
```

### Server-administration example

Check whether a sensitive configuration file is owned by the expected service account:

```bash
sudo stat -c "%A %a %U %G %n" /etc/ssh/sshd_config
```

### Common mistakes

- Confusing modification time with creation time.
- Assuming timestamps cannot be changed.
- Forgetting that symbolic links and their targets can produce different results.

### Verification

```bash
ls -l sample.txt
stat -c "%A %a %U %G %n" sample.txt
```

***

## `id`

### Purpose

Displays the current user’s UID, primary GID, and supplementary groups.

### Syntax

```bash
id [options] [username]
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-u` | Show user ID | `id -u alice` |
| `-g` | Show primary group ID | `id -g alice` |
| `-G` | Show all group IDs | `id -G alice` |
| `-n` | Show names instead of numbers | `id -Gn alice` |
| `-r` | Show real rather than effective ID | `id -ru` |

### Beginner example

```bash
id
id -Gn
```

Expected output; your values will differ:

```text
uid=1000(student) gid=1000(student) groups=1000(student),27(sudo)
student sudo
```

### Server-administration example

Verify access before changing a shared directory:

```bash
id deploy
id -Gn deploy
```

### Common mistakes

- Forgetting that group changes may not appear in an existing session.
- Confusing numeric IDs with names.
- Assuming `id` alone proves filesystem access; parent-directory permissions and ACLs also matter.

### Verification

```bash
getent passwd alice
getent group developers
id alice
```

***

## `whoami`

### Purpose

Prints the effective username of the current process.

### Syntax

```bash
whoami
```

### Beginner example

```bash
whoami
```

Expected output:

```text
student
```

### Server-administration example

Confirm the identity used for a privileged command:

```bash
sudo -u bob whoami
sudo whoami
```

Expected output:

```text
bob
root
```

### Common mistakes

- Assuming `whoami` shows the original login account after `sudo`.
- Using it as a replacement for `id`.

### Verification

```bash
whoami
id
```

***

## `groups`

### Purpose

Displays group membership for the current user or a specified user.

### Syntax

```bash
groups [username]
```

### Beginner example

```bash
groups
groups alice
```

Expected output; your values will differ:

```text
student sudo
alice developers
```

### Server-administration example

Check whether a user belongs to a sensitive group:

```bash
groups deploy
```

### Common mistakes

- Existing sessions may not have newly added memberships.
- `groups` does not show permission bits or ACLs.
- Use `getent group GROUP` to inspect the group database directly.

### Verification

```bash
id alice
getent group developers
```

***

## `chmod`

### Purpose

Changes the permission mode of files and directories.

### Syntax

```bash
chmod [options] mode file_or_directory
```

### Symbolic mode

| Symbol | Meaning |
|---|---|
| `u` | User/owner |
| `g` | Group |
| `o` | Others |
| `a` | All three classes |
| `+` | Add permissions |
| `-` | Remove permissions |
| `=` | Set exact permissions |
| `r` | Read |
| `w` | Write |
| `x` | Execute |

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-R` | Recursive | `chmod -R 750 /opt/project` |
| `-v` | Verbose | `chmod -v 640 file.txt` |
| `-c` | Report only changes | `chmod -c 640 file.txt` |
| `--reference=FILE` | Copy mode from another file | `chmod --reference=ref target` |

### Beginner examples

```bash
chmod u+x hello.sh
chmod g-w sample.txt
chmod o-rwx sample.txt
chmod 644 sample.txt
chmod 755 hello.sh
chmod 600 private.txt
```

Create the private file first:

```bash
touch private.txt
chmod 600 private.txt
ls -l private.txt
```

Expected output:

```text
-rw------- 1 student student 0 Oct 21 18:20 private.txt
```

Copy permissions from another file:

```bash
chmod --reference=private.txt sample.txt
stat -c "%A %a %n" sample.txt private.txt
```

### Recursive use

```bash
mkdir -p project
touch project/app.conf project/deploy.sh
chmod -R 750 project
```

Use recursive changes only after checking the tree:

```bash
find project -maxdepth 2 -print
```

### Server-administration examples

Protect an SSH private key:

```bash
chmod 600 ~/.ssh/id_rsa
```

Make a deployment script executable:

```bash
chmod 750 /opt/company/bin/deploy.sh
```

Allow a team to manage a project directory:

```bash
sudo chmod 2770 /srv/projects/dev
```

Remove access for others:

```bash
sudo chmod o-rwx /srv/projects/dev
```

### Security warning

Do not use:

```bash
sudo chmod -R 777 /srv
```

This can expose credentials, allow unauthorized modification, enable file replacement, and create privilege-escalation paths.

Safer pattern:

```bash
sudo chown -R root:developers /srv/projects/dev
sudo find /srv/projects/dev -type d -exec chmod 2770 {} +
sudo find /srv/projects/dev -type f -exec chmod 0660 {} +
```

Review the result before applying such commands to real data.

### Common mistakes

- Applying `chmod -R` to a tree containing executable and non-executable files.
- Using `777` to hide an ownership problem.
- Forgetting execute permission on directories.
- Giving scripts more permissions than their users require.
- Changing permissions on symlink targets unintentionally.

### Verification

```bash
ls -ld /srv/projects/dev
find /srv/projects/dev -maxdepth 2 -printf '%M %u %g %p\n'
stat -c "%A %a %U %G %n" /srv/projects/dev
```

***

## `chown`

### Purpose

Changes the owner and/or group owner of a file or directory.

### Syntax

```bash
chown [options] [owner][:group] file_or_directory
```

A dot may also be accepted as an owner/group separator on some systems, but the colon form is clearer and preferred:

```bash
owner:group
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-R` | Recursive | `chown -R alice:developers /srv/project` |
| `-v` | Verbose | `chown -v alice file.txt` |
| `-c` | Report only changes | `chown -c alice file.txt` |
| `--reference=FILE` | Copy ownership from another file | `chown --reference=ref target` |
| `--from=CURRENT_OWNER` | Change only if current owner matches | `chown --from=olduser alice file.txt` |
| `-h` | Change symlink itself where supported | `chown -h user link` |

### Beginner examples

```bash
sudo chown alice sample.txt
sudo chown alice:developers sample.txt
sudo chown :developers sample.txt
```

### Recursive example

```bash
sudo chown -R alice:developers /srv/project
```

Always verify the path first:

```bash
sudo ls -ld /srv/project
sudo find /srv/project -maxdepth 2 -print
```

### Server-administration examples

Assign a web application tree to a service account:

```bash
sudo chown -R www-data:www-data /var/www/example
```

On RHEL-family systems, the service account may be `apache` instead:

```bash
sudo chown -R apache:apache /var/www/example
```

Recover ownership after copying files as root:

```bash
sudo chown -R deploy:developers /srv/releases/current
```

### Common mistakes

- Running `chown -R` on the wrong directory.
- Changing ownership of a symlink target unintentionally.
- Assigning application files to a service account without understanding the service’s write requirements.
- Assuming `chown` changes permissions; it changes ownership only.

### Verification

```bash
ls -ld /srv/project
stat -c "%U %G %n" /srv/project
```

***

## `chgrp`

### Purpose

Changes only the group ownership of a file or directory.

### Syntax

```bash
chgrp [options] group file_or_directory
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-R` | Recursive | `chgrp -R developers /srv/project` |
| `-v` | Verbose | `chgrp -v developers file.txt` |
| `-c` | Report only changes | `chgrp -c developers file.txt` |
| `--reference=FILE` | Copy group ownership from another file | `chgrp --reference=ref target` |

### Beginner examples

```bash
sudo chgrp developers sample.txt
sudo chgrp --reference=sample.txt private.txt
```

### Server-administration example

Change a shared directory’s group while preserving its current owner:

```bash
sudo chgrp -R developers /srv/project
```

Use `chgrp` when the owner should remain unchanged and only group access needs adjustment. Use `chown owner:group` when both need to change.

### Common mistakes

- Changing a group without giving that group useful permission bits.
- Forgetting that users need membership in the group.
- Applying recursively without checking the target tree.

### Verification

```bash
stat -c "%U %G %n" sample.txt
getent group developers
```

***

## `useradd`, `groupadd`, and `usermod`

These commands are covered in detail in the Users & Groups tutorial. They are included here because permissions depend on identities.

### Create a group

```bash
sudo groupadd developers
```

Verify:

```bash
getent group developers
```

### Create a user

```bash
sudo useradd -m -s /bin/bash developer1
```

Important options:

| Option | Meaning |
|---|---|
| `-m` | Create the home directory |
| `-s SHELL` | Set login shell |
| `-c COMMENT` | Set GECOS/comment |
| `-g GROUP` | Set primary group |
| `-G GROUPS` | Set supplementary groups |

### Add a user to a group

```bash
sudo usermod -aG developers developer1
```

`-aG` is important. Without `-a`, `-G` replaces the supplementary group list:

```bash
sudo usermod -G developers developer1
```

Use the replacement form only when you intentionally want to reset all supplementary memberships.

Verify:

```bash
id developer1
groups developer1
```

***

## `su` and `sudo`

### `su`

Switches to another user.

```bash
su - alice
```

The `-` starts a login shell and loads the target user’s normal login environment.

Run one command:

```bash
su - alice -c 'id; pwd; umask'
```

### `sudo`

Runs a command under an authorized identity:

```bash
sudo -u alice id
sudo -u bob cat /srv/project/file.txt
```

Start a root login shell:

```bash
sudo -i
```

Prefer individual commands or narrowly scoped sudo rules over unrestricted root shells.

### Difference

| Command | Typical use |
|---|---|
| `su alice` | Switch user, retain much of current environment |
| `su - alice` | Switch user with login environment |
| `sudo COMMAND` | Run one authorized command as root |
| `sudo -u alice COMMAND` | Run one command as another user |

Verify:

```bash
whoami
sudo -u alice whoami
```

***

# Part 3: Umask

## What umask does

`umask` removes permissions from newly created files and directories.

Base modes are commonly:

```text
Files:       666
Directories: 777
```

The execute bit is not normally included in the base mode for regular files. This prevents ordinary files from becoming executable automatically.

Inspect the current mask:

```bash
umask
umask -S
```

Example output:

```text
0022
u=rwx,g=rx,o=rx
```

### Common results

| Umask | New files | New directories |
|---:|---:|---:|
| `022` | `644` | `755` |
| `027` | `640` | `750` |
| `077` | `600` | `700` |

The conceptual calculation is:

```text
file mode      666 minus umask 022 -> 644
directory mode 777 minus umask 022 -> 755
```

The exact behavior is bitwise masking rather than ordinary arithmetic subtraction, so use a test file to verify unusual masks.

## Test umask values

```bash
cd ~/permissions-lab
umask 022
touch umask-022-file
mkdir umask-022-dir
stat -c "%A %a %n" umask-022-file umask-022-dir
```

Expected output:

```text
-rw-r--r-- 644 umask-022-file
drwxr-xr-x 755 umask-022-dir
```

Test a stricter mask:

```bash
umask 077
touch umask-077-file
mkdir umask-077-dir
stat -c "%A %a %n" umask-077-file umask-077-dir
```

Expected output:

```text
-rw------- 600 umask-077-file
drwx------ 700 umask-077-dir
```

Set a temporary mask:

```bash
umask 027
```

Set a persistent user mask in `~/.bashrc`:

```bash
printf '\numask 027\n' >> ~/.bashrc
```

For login shells, `~/.profile` or `~/.bash_profile` may also be relevant.

System-wide locations may include:

```text
/etc/profile
/etc/profile.d/custom-umask.sh
```

Create a system-wide configuration only on the lab VM:

```bash
sudo tee /etc/profile.d/custom-umask.sh >/dev/null <<'EOF'
# Lab example: restrictive default permissions
umask 027
EOF
sudo chmod 0644 /etc/profile.d/custom-umask.sh
```

### Real-world use cases

- `022`: ordinary multi-user systems where files may be world-readable.
- `027`: files readable by the owner and group, but not others.
- `077`: sensitive administrative accounts, credentials, and private workspaces.

Be careful with global umask changes: applications may depend on existing defaults, and group collaboration may require `0002` or explicit ACLs.

### Common mistakes

- Expecting `umask` to change existing files.
- Assuming every process reads shell startup files.
- Setting `077` globally without testing services and deployment workflows.
- Treating umask as a replacement for correct ownership and ACLs.

Verification:

```bash
umask
stat -c "%A %a %n" umask-077-file
```

***

# Part 4: POSIX ACLs

## Why ACLs exist

Traditional permissions provide only three classes:

```text
owner | group | others
```

ACLs allow named users and groups to receive additional permissions without changing the primary owner or group.

A directory listing may show a `+`:

```text
-rw-rw-r--+ 1 alice developers 100 Oct 21 18:30 report.txt
```

The `+` means an extended ACL exists.

## Install ACL tools

Ubuntu/Debian:

```bash
sudo apt update
sudo apt install acl
```

RHEL/Rocky/Alma/Fedora:

```bash
sudo dnf install acl
```

Check commands:

```bash
command -v getfacl
command -v setfacl
```

The filesystem must support POSIX ACLs. Most modern Linux filesystems do, but mount options and filesystem type can affect behavior.

## ACL concepts

| ACL entry | Meaning |
|---|---|
| `user::rwx` | Owner permissions |
| `user:bob:rw-` | Named-user permissions |
| `group::r-x` | Owning-group permissions |
| `group:developers:r-x` | Named-group permissions |
| `mask::rwx` | Maximum effective permissions for named users, named groups, and owning group |
| `other::---` | Permissions for everyone else |
| Default ACL | ACL inherited by newly created children of a directory |

The ACL mask can reduce effective permissions even when a named entry appears broader.

***

## `getfacl`

### Purpose

Displays access and default ACL entries.

### Syntax

```bash
getfacl [options] file_or_directory
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-R` | Recursive | `getfacl -R /srv/project` |
| `-d` | Show default ACL only | `getfacl -d /srv/project` |
| `-a` | Show access ACL | `getfacl -a file.txt` |
| `-c` | Omit header comments | `getfacl -c file.txt` |
| `-p` | Preserve full path format | `getfacl -p file.txt` |

### Beginner example

```bash
getfacl sample.txt
```

Example output:

```text
# file: sample.txt
# owner: student
# group: student
user::rw-
group::r--
other::r--
```

### Server-administration example

```bash
sudo getfacl -R /srv/project
sudo getfacl -d /srv/project
```

### Common mistakes

- Ignoring the ACL mask.
- Setting a default ACL but forgetting to set an access ACL on an existing directory.
- Assuming ACLs replace ownership and group design.
- Copying files with a tool that does not preserve ACLs.

### Verification

```bash
getfacl -c sample.txt
ls -l sample.txt
```

***

## `setfacl`

### Purpose

Creates, modifies, removes, or restores POSIX ACL entries.

### Syntax

```bash
setfacl [options] acl_specification file_or_directory
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-m` | Modify or add ACL entries | `setfacl -m u:bob:rw file.txt` |
| `-x` | Remove a specific ACL entry | `setfacl -x u:carol file.txt` |
| `-b` | Remove all extended ACL entries | `setfacl -b file.txt` |
| `-R` | Recursive | `setfacl -R -m g:developers:rx /srv/project` |
| `-d` | Modify default ACL | `setfacl -d -m g:developers:rwx /srv/project` |
| `--restore=FILE` | Restore from `getfacl` backup | `setfacl --restore=acl.backup` |

### Beginner examples

```bash
setfacl -m u:bob:rw sample.txt
setfacl -m u:carol:r sample.txt
getfacl sample.txt
```

Remove one entry:

```bash
setfacl -x u:carol sample.txt
```

Remove extended ACL entries:

```bash
setfacl -b sample.txt
```

### Server-administration examples

Give an auditor read-only access to logs:

```bash
sudo setfacl -m u:auditor:r /var/log/application.log
```

Give a second team read and execute access to a directory:

```bash
sudo setfacl -m g:security:rx /srv/application
```

Restrict others:

```bash
sudo setfacl -m o::--- /srv/application
```

Set default inheritance:

```bash
sudo setfacl -d -m u:bob:rw /srv/project
sudo setfacl -d -m u:carol:r /srv/project
```

### ACL mask example

```bash
setfacl -m u:bob:rwx,m::r-x sample.txt
getfacl sample.txt
```

The `user:bob:rwx` entry may display effective permissions limited by `mask::r-x`.

### Common mistakes

- Using `-d` without understanding default ACL inheritance.
- Forgetting that a named user still needs execute permission on parent directories.
- Setting a named ACL but leaving the mask too restrictive.
- Applying `setfacl -R` to an entire filesystem.
- Assuming ACLs are preserved by every backup or copy tool.

### Verification

```bash
getfacl sample.txt
ls -l sample.txt
```

***

# Part 5: Linux Capabilities

## Why capabilities exist

Traditional privilege treats root as broadly privileged. Linux capabilities divide parts of that power into narrower privileges.

Common capabilities include:

| Capability | Typical meaning |
|---|---|
| `cap_net_bind_service` | Bind to ports below 1024 |
| `cap_net_raw` | Use raw and packet sockets |
| `cap_dac_read_search` | Bypass file read and directory-search permission checks in limited ways |
| `cap_dac_override` | Bypass many discretionary access checks |
| `cap_sys_admin` | Very broad administrative power; often considered dangerous |

Capability sets include:

- **Permitted**: capabilities the process may use.
- **Effective**: capabilities currently active.
- **Inheritable**: capabilities available for inheritance across execution boundaries.

Install tools:

Ubuntu/Debian:

```bash
sudo apt install libcap2-bin
```

RHEL/Rocky/Alma/Fedora:

```bash
sudo dnf install libcap
```

## `getcap`

### Purpose

Displays file capabilities.

### Syntax

```bash
getcap [options] file
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-r` | Recursive search | `getcap -r /usr/bin` |
| `-v` | Verbose output | `getcap -v /usr/bin/ping` |

### Beginner example

```bash
getcap /usr/bin/ping 2>/dev/null || true
```

Possible output:

```text
/usr/bin/ping cap_net_raw=ep
```

Some systems use other mechanisms or no file capability for `ping`.

### Server-administration example

```bash
sudo getcap -r /usr/local/bin /opt 2>/dev/null
```

Audit all configured application binaries in a controlled scope.

### Common mistakes

- Assuming no output means no privilege path exists.
- Searching `/` without considering performance and mounted filesystems.
- Treating capabilities as harmless because they are not SUID.

### Verification

```bash
getcap /usr/local/bin/myserver
```

***

## `setcap`

### Purpose

Assigns or removes file capabilities.

### Syntax

```bash
setcap [options] capability file
```

### Important options

| Option | Meaning | Example |
|---|---|---|
| `-r` | Remove all capabilities | `sudo setcap -r /usr/local/bin/myserver` |
| `-v` | Verify | `sudo setcap -v cap_net_bind_service=+ep file` |
| `-q` | Quiet mode | `sudo setcap -q ...` |
| `-n UID` | Set root UID for namespaced file capabilities where supported | Verify platform support |

### Capability syntax

```text
cap_net_bind_service=+ep
```

Means add the capability to the effective and permitted sets.

```text
cap_net_bind_service=ep
```

Sets the effective and permitted flags exactly for that capability.

### Example

Create a lab binary:

```bash
cat > ~/permissions-lab/bind-test.c <<'EOF'
#include <stdio.h>
int main(void) {
    puts("Capability test binary");
    return 0;
}
EOF
```

Compile if a compiler is available:

```bash
cc ~/permissions-lab/bind-test.c -o ~/permissions-lab/bind-test
```

Granting a capability to this harmless binary is only a syntax demonstration; it does not bind to a privileged port:

```bash
sudo setcap cap_net_bind_service=+ep ~/permissions-lab/bind-test
getcap ~/permissions-lab/bind-test
```

Expected output:

```text
/home/student/permissions-lab/bind-test cap_net_bind_service=ep
```

Remove it:

```bash
sudo setcap -r ~/permissions-lab/bind-test
getcap ~/permissions-lab/bind-test
```

### Server-administration example

A reviewed, dedicated network daemon may bind to port 80 without running as root:

```bash
sudo setcap cap_net_bind_service=+ep /usr/local/bin/myserver
getcap /usr/local/bin/myserver
```

Grant this only after verifying the binary, ownership, update process, service account, and threat model.

### Security warnings

- Do not grant capabilities to untrusted or user-writable binaries.
- `cap_dac_override` and `cap_sys_admin` can be extremely powerful.
- Package updates or binary replacement may remove file capabilities.
- Capabilities can be hidden privilege paths during audits.
- Prefer service-manager socket activation or a reverse proxy when practical.

### Verification

```bash
getcap /usr/local/bin/myserver
sudo getcap -r /usr/local/bin /opt 2>/dev/null
```

***

# Part 6: Special Permissions

## SUID

### Meaning

SUID has numeric value `4` and symbolic form `u+s`.

On an executable file, SUID causes the process to run with the file owner’s effective UID. The classic example is a password-changing utility that needs controlled access to protected account databases.

Inspect:

```bash
ls -l /usr/bin/passwd
stat /usr/bin/passwd
```

Example output; your values will differ:

```text
-rwsr-xr-x 1 root root 64152 Oct 21 18:40 /usr/bin/passwd
```

The `s` in the owner execute position indicates SUID.

### Test a lab file

```bash
cd ~/permissions-lab
printf '#!/usr/bin/env bash\nid\n' > testfile
chmod 755 testfile
chmod u+s testfile
ls -l testfile
```

A regular shell script generally does not provide a useful or safe SUID privilege boundary. Modern kernels ignore or restrict SUID behavior on scripts because of race and interpreter-security problems.

Remove it:

```bash
chmod u-s testfile
```

### Find SUID files

Limit the search to a controlled filesystem:

```bash
sudo find /opt/permission-lab -xdev -perm -4000 -type f -ls 2>/dev/null
```

A broader audit:

```bash
sudo find / -xdev -perm -4000 -type f -ls 2>/dev/null
```

### Security warning

Unexpected SUID binaries can provide privilege-escalation paths. Review owner, package source, integrity, purpose, and change history before removing anything.

***

## SGID

### Meaning

SGID has numeric value `2` and symbolic form `g+s`.

On an executable, SGID can make a process run with the file’s group identity.

On a directory, SGID causes new files and directories to inherit the directory’s group ownership.

Create a shared directory:

```bash
sudo mkdir -p /srv/permission-lab/shared
sudo groupadd permissionteam 2>/dev/null || true
sudo chown root:permissionteam /srv/permission-lab/shared
sudo chmod 2770 /srv/permission-lab/shared
sudo ls -ld /srv/permission-lab/shared
```

Expected output:

```text
drwxrws--- 2 root permissionteam 4096 Oct 21 18:45 /srv/permission-lab/shared
```

The `s` in the group execute position indicates SGID.

Test inheritance:

```bash
sudo -u developer1 touch /srv/permission-lab/shared/developer-file.txt
sudo ls -l /srv/permission-lab/shared/developer-file.txt
```

The file should show group `permissionteam` if `developer1` is a member and access is configured correctly.

Find SGID directories:

```bash
sudo find /opt/permission-lab -xdev -perm -2000 -type d -ls 2>/dev/null
```

***

## Sticky bit

### Meaning

The sticky bit has numeric value `1` and symbolic form `+t`.

On a directory, users may create files according to directory permissions, but normally only:

- The file owner,
- The directory owner,
- Or root

can delete or rename an entry.

Inspect `/tmp`:

```bash
ls -ld /tmp
```

Typical output:

```text
drwxrwxrwt 20 root root 4096 Oct 21 18:45 /tmp
```

The final `t` indicates the sticky bit.

Create a lab directory:

```bash
sudo mkdir -p /srv/permission-lab/dropbox
sudo chown root:permissionteam /srv/permission-lab/dropbox
sudo chmod 1770 /srv/permission-lab/dropbox
sudo ls -ld /srv/permission-lab/dropbox
```

For a world-writable temporary-style directory:

```bash
sudo chmod 1777 /srv/permission-lab/dropbox
```

Use `1777` only for a deliberately world-writable temporary area. Group-specific shared areas should normally use a narrower mode such as `2770`, `1770`, or an ACL design.

Find sticky directories:

```bash
sudo find /opt/permission-lab -xdev -perm -1000 -type d -ls 2>/dev/null
```

***

## Lowercase and uppercase special bits

| Display | Meaning |
|---|---|
| `s` | Special bit set and execute bit set |
| `S` | Special bit set but execute bit not set |
| `t` | Sticky bit set and execute bit set |
| `T` | Sticky bit set but execute bit not set |

Examples:

```bash
chmod 4755 testfile
chmod 4644 testfile
ls -l testfile
```

A capital `S` may indicate a suspicious or ineffective SUID/SGID configuration because the corresponding execute bit is absent.

***

# Part 7: Least Privilege

Least privilege means granting the smallest access needed for a user, service, process, or task.

Apply it by:

- Using groups instead of broad “others” permissions.
- Using ACLs for carefully documented exceptions.
- Using capabilities instead of full root where appropriate.
- Avoiding unnecessary SUID binaries.
- Avoiding `777`.
- Restricting private keys and credentials.
- Giving service accounts no interactive shell unless required.
- Auditing ownership, ACLs, capabilities, and special bits.

## Secure permission baseline

| Resource | Recommended permission | Reason |
|---|---:|---|
| SSH private key | `600` | Only owner should read it |
| Public web file | `644` | Owner writes, others read |
| Executable script | `755` or `750` | Controlled execution |
| Team project directory | `770` or `2770` | Team-only access |
| Shared team directory | `2770`, optionally sticky policy | Group inheritance and controlled deletion |
| Configuration with secrets | `600` or `640` | Restrict sensitive access |
| Log directory | `750` or `2750` | Limit access to authorized operators |

***

# Hands-On Labs

# Lab 6.1 — Permission Matrix

## Task

Create groups:

```text
management
developers
network
support
shared
```

Create users:

```text
manager1
developer1
network1
support1
shareduser
```

Create directories:

```text
/opt/company/management
/opt/company/developers
/opt/company/network
/opt/company/support
/opt/company/shared
```

Apply:

- `management`: only the management group can read, write, and enter.
- `developers`: developers can read, write, and enter; others have no access.
- `network`: network can read and enter, but cannot write.
- `support`: support can read and enter; manager can also write with an ACL.
- `shared`: all teams can read and enter, but only `shared` can write.

Test with identity switching and verify every result.

## Solution

Create groups:

```bash
for group in management developers network support shared; do
  sudo groupadd "$group" 2>/dev/null || true
done
```

Create users with primary groups:

```bash
sudo useradd -m -s /bin/bash -g management manager1
sudo useradd -m -s /bin/bash -g developers developer1
sudo useradd -m -s /bin/bash -g network network1
sudo useradd -m -s /bin/bash -g support support1
sudo useradd -m -s /bin/bash -g shared shareduser
```

Add cross-team memberships needed for the shared directory:

```bash
sudo usermod -aG management manager1
sudo usermod -aG developers developer1
sudo usermod -aG network network1
sudo usermod -aG support support1
sudo usermod -aG shared shareduser

sudo usermod -aG shared manager1
sudo usermod -aG shared developer1
sudo usermod -aG shared network1
sudo usermod -aG shared support1
```

Set lab-only passwords if interactive `su` testing is required:

```bash
printf '%s\n' \
  'manager1:LabOnly-Manager-2026!' \
  'developer1:LabOnly-Developer-2026!' \
  'network1:LabOnly-Network-2026!' \
  'support1:LabOnly-Support-2026!' \
  'shareduser:LabOnly-Shared-2026!' | sudo chpasswd
```

Create directories:

```bash
sudo mkdir -p \
  /opt/company/management \
  /opt/company/developers \
  /opt/company/network \
  /opt/company/support \
  /opt/company/shared
```

Set group ownership:

```bash
sudo chown root:management /opt/company/management
sudo chown root:developers /opt/company/developers
sudo chown root:network /opt/company/network
sudo chown root:support /opt/company/support
sudo chown root:shared /opt/company/shared
```

Apply base permissions:

```bash
sudo chmod 2770 /opt/company/management
sudo chmod 2770 /opt/company/developers
sudo chmod 2750 /opt/company/network
sudo chmod 2750 /opt/company/support
sudo chmod 2770 /opt/company/shared
```

The `management` and `developers` directories are group-writable. The `network` and `support` directories are group-readable and traversable but not group-writable.

Allow `manager1` to write to support using an ACL:

```bash
sudo setfacl -m u:manager1:rwx /opt/company/support
```

Configure the shared directory so department members can read and enter, but only `shareduser` and the `shared` group can write:

```bash
sudo setfacl -m \
  g:management:rx,g:developers:rx,g:network:rx,g:support:rx,g:shared:rwx \
  /opt/company/shared

sudo setfacl -m o::--- /opt/company/shared
```

Inspect:

```bash
sudo ls -ld /opt/company/*
sudo getfacl /opt/company/support
sudo getfacl /opt/company/shared
```

### Test management access

```bash
sudo -u manager1 sh -c 'touch /opt/company/management/manager-file && printf "manager data\n" > /opt/company/management/manager-file'
sudo -u developer1 ls /opt/company/management
```

The developer listing should fail:

```text
ls: cannot open directory '/opt/company/management': Permission denied
```

### Test developer access

```bash
sudo -u developer1 touch /opt/company/developers/dev-file
sudo -u developer1 cat /opt/company/developers/dev-file
sudo -u support1 ls /opt/company/developers
```

The support access should fail.

### Test network read-only access

```bash
printf 'network data\n' | sudo tee /opt/company/network/network-file >/dev/null
sudo chown root:network /opt/company/network/network-file
sudo chmod 0640 /opt/company/network/network-file

sudo -u network1 cat /opt/company/network/network-file
sudo -u network1 sh -c 'printf "blocked\n" >> /opt/company/network/network-file'
```

The append should fail because the file is not group-writable.

### Test manager ACL on support

```bash
sudo -u support1 touch /opt/company/support/support-file
sudo -u manager1 sh -c 'printf "manager review\n" >> /opt/company/support/support-file'
```

### Test shared access

```bash
sudo -u developer1 touch /opt/company/shared/developer-shared-file
sudo -u support1 cat /opt/company/shared/developer-shared-file
sudo -u support1 sh -c 'printf "support update\n" >> /opt/company/shared/developer-shared-file'
```

The second operation may depend on the created file’s mode and the effective ACL/umask. Shared write behavior should be designed deliberately with default ACLs or a group-friendly umask.

Verify identities and metadata:

```bash
for user in manager1 developer1 network1 support1 shareduser; do
  id "$user"
  groups "$user"
done

sudo stat -c "%A %a %U %G %n" /opt/company/*
sudo getfacl -p /opt/company/support /opt/company/shared
```

## Expected-results checklist

- [ ] All five groups exist.
- [ ] All five users exist.
- [ ] Every directory has the intended group owner.
- [ ] Management and developer members can work in their directories.
- [ ] Network and support groups cannot write to their base directories.
- [ ] `manager1` has ACL-based write access to support.
- [ ] Department users can read the shared directory.
- [ ] Only the `shared` group can write to the shared directory.
- [ ] `id`, `groups`, `ls -ld`, `stat`, and `getfacl` verify the configuration.

***

# Lab 6.2 — Shared Directory

## Task

Create `/srv/shared` with:

- Department users able to read and write.
- Other users denied.
- New files inheriting the department group.
- Users unable to delete files owned by other users.

Use:

- A dedicated group.
- Group ownership.
- `chmod`.
- SGID.
- Sticky bit.
- Optional ACLs.

## Solution

Create the group and users:

```bash
sudo groupadd department 2>/dev/null || true
sudo useradd -m -s /bin/bash -G department deptuser1
sudo useradd -m -s /bin/bash -G department deptuser2
```

Create a nonmember test user:

```bash
sudo useradd -m -s /bin/bash outsider1
```

Create the directory:

```bash
sudo mkdir -p /srv/shared
sudo chown root:department /srv/shared
sudo chmod 3770 /srv/shared
```

`3770` combines:

- `2`: SGID.
- `1`: Sticky bit.
- `770`: owner/group full access, others no access.

Inspect:

```bash
sudo ls -ld /srv/shared
```

Expected output:

```text
drwxrws--T 2 root department 4096 Oct 21 19:00 /srv/shared
```

The uppercase `T` means sticky bit is set but others execute permission is absent. For group members to enter, group execute permission is present; the display may instead show a lowercase `t` depending on the other execute bit. A practical mode for a group-only directory is often `3770`.

Set lab-only passwords if using `su`:

```bash
printf '%s\n' \
  'deptuser1:LabOnly-Dept1-2026!' \
  'deptuser2:LabOnly-Dept2-2026!' \
  'outsider1:LabOnly-Outsider-2026!' | sudo chpasswd
```

Test creation and inherited group:

```bash
sudo -u deptuser1 touch /srv/shared/file-from-deptuser1
sudo ls -l /srv/shared/file-from-deptuser1
```

Expected group owner:

```text
-rw-r--r-- 1 deptuser1 department 0 Oct 21 19:01 file-from-deptuser1
```

Test reading and writing:

```bash
sudo -u deptuser2 cat /srv/shared/file-from-deptuser1
sudo -u deptuser2 sh -c 'printf "deptuser2 update\n" >> /srv/shared/file-from-deptuser1'
```

Whether `deptuser2` can write depends on the file’s group-write bit. Use a group-friendly umask or default ACLs for reliable collaboration:

```bash
sudo setfacl -d -m g:department:rwx /srv/shared
sudo setfacl -d -m o::--- /srv/shared
```

Create another file:

```bash
sudo -u deptuser1 touch /srv/shared/collaborative-file
sudo getfacl /srv/shared/collaborative-file
```

Test outsider denial:

```bash
sudo -u outsider1 ls /srv/shared
```

Expected:

```text
ls: cannot open directory '/srv/shared': Permission denied
```

Test sticky-bit deletion protection:

```bash
sudo -u deptuser1 touch /srv/shared/dept1-owned
sudo -u deptuser2 rm /srv/shared/dept1-owned
```

Expected:

```text
rm: cannot remove '/srv/shared/dept1-owned': Operation not permitted
```

Verify:

```bash
sudo ls -ld /srv/shared
sudo stat -c "%A %a %U %G %n" /srv/shared
sudo getfacl /srv/shared
```

## Expected-results checklist

- [ ] `/srv/shared` is group-owned by `department`.
- [ ] SGID is set.
- [ ] Sticky bit is set.
- [ ] Department users can enter the directory.
- [ ] An outsider cannot enter it.
- [ ] New files inherit group `department`.
- [ ] A department user cannot delete another department user’s file.
- [ ] Default ACLs, if used, are visible with `getfacl`.

***

# Lab 6.3 — ACL

## Task

Create users:

```text
alice
bob
carol
```

Create `/srv/project` with:

- `alice` as owner.
- `developers` as group.
- Developers with read and execute access.
- `bob` with individual read/write access through ACL.
- `carol` with individual read-only access through ACL.
- Others denied.
- New files inheriting the intended ACL permissions.

## Solution

Ensure users and group exist:

```bash
sudo groupadd developers 2>/dev/null || true
sudo useradd -m -s /bin/bash -G developers alice
sudo useradd -m -s /bin/bash -G developers bob
sudo useradd -m -s /bin/bash carol
```

Create the project:

```bash
sudo mkdir -p /srv/project
sudo chown alice:developers /srv/project
sudo chmod 2770 /srv/project
```

Set access ACLs:

```bash
sudo setfacl -m u:bob:rwx /srv/project
sudo setfacl -m u:carol:r-x /srv/project
sudo setfacl -m g:developers:r-x /srv/project
sudo setfacl -m o::--- /srv/project
```

The requirement says Bob has read/write access. Directory write access also requires execute access, so `rwx` is needed on the directory.

Set default ACLs for new children:

```bash
sudo setfacl -d -m u::rwx /srv/project
sudo setfacl -d -m u:bob:rwX /srv/project
sudo setfacl -d -m u:carol:r-X /srv/project
sudo setfacl -d -m g::r-X /srv/project
sudo setfacl -d -m g:developers:r-X /srv/project
sudo setfacl -d -m m::rwx /srv/project
sudo setfacl -d -m o::--- /srv/project
```

Inspect:

```bash
getfacl /srv/project
getfacl -d /srv/project
```

Create a file as Alice:

```bash
sudo -u alice sh -c 'printf "project data\n" > /srv/project/project.txt'
sudo getfacl /srv/project/project.txt
```

Test Alice:

```bash
sudo -u alice cat /srv/project/project.txt
sudo -u alice sh -c 'printf "alice update\n" >> /srv/project/project.txt'
```

Test Bob:

```bash
sudo -u bob cat /srv/project/project.txt
sudo -u bob sh -c 'printf "bob update\n" >> /srv/project/project.txt'
```

Test Carol:

```bash
sudo -u carol cat /srv/project/project.txt
sudo -u carol sh -c 'printf "carol attempt\n" >> /srv/project/project.txt'
```

Expected result:

```text
bash: /srv/project/project.txt: Permission denied
```

Create an unrelated user:

```bash
sudo useradd -m -s /bin/bash unrelated1
sudo -u unrelated1 ls /srv/project
```

Expected:

```text
ls: cannot open directory '/srv/project': Permission denied
```

Verify:

```bash
sudo ls -ld /srv/project
sudo getfacl -p /srv/project
sudo getfacl -p /srv/project/project.txt
```

## Expected-results checklist

- [ ] Alice owns `/srv/project`.
- [ ] Developers have read/execute access.
- [ ] Bob has ACL-based write access.
- [ ] Carol can read but cannot write.
- [ ] Unrelated users are denied.
- [ ] New files receive inherited ACL entries.
- [ ] The `+` marker appears in `ls -l` where extended ACLs exist.

***

# Lab 6.4 — SUID, SGID, and Sticky Bit

## Task

Create `/opt/permission-lab`.

Investigate:

- SUID on `/usr/bin/passwd`.
- SGID on a shared directory.
- Sticky bit on a temporary directory.
- SUID, SGID, and sticky-bit searches.

## Solution

Create the isolated tree:

```bash
sudo mkdir -p /opt/permission-lab
sudo chmod 0755 /opt/permission-lab
```

Inspect `passwd`:

```bash
ls -l /usr/bin/passwd
stat /usr/bin/passwd
```

Demonstrate normal password change behavior without changing a production password:

```bash
passwd --help | head
```

Do not actually change a password unless using a disposable lab user.

Create a test executable:

```bash
cat > ~/permissions-lab/testfile.sh <<'EOF'
#!/usr/bin/env bash
id
EOF
chmod 755 ~/permissions-lab/testfile.sh
```

Apply and inspect SUID:

```bash
chmod u+s ~/permissions-lab/testfile.sh
ls -l ~/permissions-lab/testfile.sh
```

Remove it:

```bash
chmod u-s ~/permissions-lab/testfile.sh
```

Do not rely on SUID shell scripts for privilege control.

Create an SGID shared directory:

```bash
sudo groupadd permissionteam 2>/dev/null || true
sudo usermod -aG permissionteam developer1 2>/dev/null || true
sudo mkdir -p /opt/permission-lab/shared-development
sudo chown root:permissionteam /opt/permission-lab/shared-development
sudo chmod 2775 /opt/permission-lab/shared-development
sudo ls -ld /opt/permission-lab/shared-development
```

Create a file as a group member:

```bash
sudo -u developer1 touch /opt/permission-lab/shared-development/developer-file
sudo ls -l /opt/permission-lab/shared-development/developer-file
```

The group owner should be `permissionteam`.

Create a sticky directory:

```bash
sudo mkdir -p /opt/permission-lab/temporary
sudo chmod 1777 /opt/permission-lab/temporary
sudo ls -ld /opt/permission-lab/temporary
```

Use a less permissive group variant where appropriate:

```bash
sudo chown root:permissionteam /opt/permission-lab/temporary
sudo chmod 1770 /opt/permission-lab/temporary
```

Find special files in the lab:

```bash
sudo find /opt/permission-lab -perm -4000 -type f -ls
sudo find /opt/permission-lab -perm -2000 -type f -o -perm -2000 -type d -ls
sudo find /opt/permission-lab -perm -1000 -type d -ls
```

Use grouped expressions for clarity:

```bash
sudo find /opt/permission-lab -type f -perm -4000 -ls
sudo find /opt/permission-lab -type d -perm -2000 -ls
sudo find /opt/permission-lab -type d -perm -1000 -ls
```

## Expected-results checklist

- [ ] `/usr/bin/passwd` was inspected.
- [ ] SUID display behavior was observed.
- [ ] SUID was removed from the test script.
- [ ] SGID caused new files to inherit the shared directory’s group.
- [ ] Sticky bit was applied to a temporary-style directory.
- [ ] Special-bit searches were limited to the lab path.

***

# Command Cheat Sheet

## Inspection

```bash
ls -l FILE
ls -ld DIRECTORY
ls -la
ls -lh
ls -li
stat FILE
stat -c "%A %a %U %G %n" FILE
id
id USER
whoami
groups USER
```

## Permissions

```bash
chmod 644 FILE
chmod 600 FILE
chmod 755 SCRIPT
chmod 750 DIRECTORY
chmod u+x SCRIPT
chmod g-w FILE
chmod o-rwx FILE
chmod -R 750 DIRECTORY
chmod --reference=REFERENCE TARGET
```

## Ownership

```bash
sudo chown USER FILE
sudo chown USER:GROUP FILE
sudo chown :GROUP FILE
sudo chown -R USER:GROUP DIRECTORY
sudo chgrp GROUP FILE
sudo chgrp -R GROUP DIRECTORY
```

## Users and groups

```bash
sudo groupadd GROUP
sudo useradd -m -s /bin/bash USER
sudo usermod -aG GROUP USER
sudo usermod -g GROUP USER
sudo userdel -r USER
```

## Identity switching

```bash
su - USER
sudo -u USER COMMAND
sudo -i
```

## Umask

```bash
umask
umask -S
umask 022
umask 027
umask 077
```

## ACLs

```bash
getfacl FILE
getfacl -R DIRECTORY
getfacl -d DIRECTORY
setfacl -m u:USER:rw FILE
setfacl -m g:GROUP:rx DIRECTORY
setfacl -x u:USER FILE
setfacl -b FILE
setfacl -d -m g:GROUP:rwx DIRECTORY
setfacl --restore=BACKUP
```

## Capabilities

```bash
getcap FILE
getcap -r DIRECTORY
sudo setcap cap_net_bind_service=+ep FILE
sudo setcap -r FILE
```

## Special bits

```bash
chmod 4755 FILE
chmod u+s FILE
chmod 2770 DIRECTORY
chmod g+s DIRECTORY
chmod 1777 DIRECTORY
chmod +t DIRECTORY
find DIRECTORY -perm -4000 -type f
find DIRECTORY -perm -2000 -type d
find DIRECTORY -perm -1000 -type d
```

***

# Permission Mode Cheat Sheet

| Mode | Meaning |
|---:|---|
| `000` | No access |
| `400` | Owner read |
| `600` | Owner read/write |
| `640` | Owner read/write, group read |
| `644` | Owner read/write, everyone else read |
| `700` | Owner full access |
| `750` | Owner full, group read/execute |
| `755` | Owner full, everyone else read/execute |
| `770` | Owner/group full |
| `775` | Owner/group full, others read/execute |
| `777` | Everyone full; normally unsafe |
| `1755` | Sticky bit plus `755` |
| `2770` | SGID plus owner/group full |
| `3770` | Sticky plus SGID plus owner/group full |
| `4755` | SUID plus `755` |

***

# ACL Cheat Sheet

```bash
getfacl FILE
getfacl -d DIRECTORY
setfacl -m u:alice:rw FILE
setfacl -m g:developers:r-x DIRECTORY
setfacl -m o::--- DIRECTORY
setfacl -x u:alice FILE
setfacl -b FILE
setfacl -R -m g:developers:r-x DIRECTORY
setfacl -d -m g:developers:rwx DIRECTORY
getfacl -R DIRECTORY > acl-backup.txt
setfacl --restore=acl-backup.txt
```

Remember the ACL mask:

```bash
getfacl FILE
```

Look for:

```text
mask::rwx
```

The mask limits the effective permissions of named users, named groups, and the owning group.

***

# Capability Cheat Sheet

```bash
getcap FILE
getcap -r /usr/local/bin
sudo setcap cap_net_bind_service=+ep FILE
sudo setcap cap_net_raw=+ep FILE
sudo setcap -r FILE
```

Audit capabilities after:

- Package updates.
- Binary replacement.
- Deployment changes.
- Service-account changes.
- Security incidents.

***

# Troubleshooting

## Permission denied

Inspect every path component:

```bash
namei -l /srv/project/file.txt
```

Then inspect:

```bash
id USER
getfacl /srv/project/file.txt
ls -ld /srv /srv/project
```

Common causes:

- Missing execute permission on a parent directory.
- Wrong owner or group.
- User not in the expected group.
- ACL mask restriction.
- SELinux denial on RHEL-family systems.
- AppArmor denial on Ubuntu systems.

## User cannot enter a directory

A directory requires execute permission:

```bash
ls -ld /path/to/directory
namei -l /path/to/directory
```

Grant only the needed access:

```bash
sudo chmod g+x /path/to/directory
```

Do not immediately use `777`.

## User can read but cannot write

Inspect the file and parent directory:

```bash
ls -l FILE
ls -ld "$(dirname FILE)"
getfacl FILE
getfacl "$(dirname FILE)"
```

A user needs write permission on the file to modify it, but needs write and execute permission on the directory to create, rename, or delete entries.

## Script will not execute

Check:

```bash
ls -l script.sh
file script.sh
head -n 1 script.sh
```

Fix common issues:

```bash
chmod u+x script.sh
sed -i 's/\r$//' script.sh
```

The second command removes carriage returns from Windows-style line endings. Review the file before using it.

## Web server cannot read application files

Identify the service account:

```bash
ps -eo user,group,comm,args | grep -E 'nginx|apache|httpd' | grep -v grep
```

Inspect the path:

```bash
namei -l /var/www/example/index.html
getfacl /var/www/example/index.html
```

Typical fixes involve correct ownership and read/execute permissions:

Ubuntu/Debian example:

```bash
sudo chown -R www-data:www-data /var/www/example
sudo find /var/www/example -type d -exec chmod 0750 {} +
sudo find /var/www/example -type f -exec chmod 0640 {} +
```

RHEL-family example:

```bash
sudo chown -R apache:apache /var/www/example
```

Do not grant the web server write access unless the application explicitly requires it.

## ACL is not inherited

Check default ACLs:

```bash
getfacl -d DIRECTORY
```

Set them:

```bash
sudo setfacl -d -m g:developers:rwx DIRECTORY
```

Then create a new file and inspect:

```bash
touch DIRECTORY/new-file
getfacl DIRECTORY/new-file
```

Existing files are not automatically changed by a default ACL. Apply an access ACL recursively when appropriate:

```bash
sudo setfacl -R -m g:developers:rwX DIRECTORY
```

## `setcap` disappears after replacing a binary

File capabilities are metadata on a specific inode. Replacing the file can remove them.

Check:

```bash
getcap /usr/local/bin/myserver
```

Reapply through the deployment process:

```bash
sudo setcap cap_net_bind_service=+ep /usr/local/bin/myserver
```

Then verify and document it.

## SUID does not work on a shell script

This is expected on modern Linux systems. SUID scripts are generally ignored or restricted because of interpreter and race-condition risks.

Use instead:

- A carefully designed compiled helper.
- A narrowly scoped sudo rule.
- A system service.
- A capability where appropriate.
- A privileged broker with strict validation.

## Permission changes appear ineffective

Check whether the user’s session has refreshed groups:

```bash
id USER
```

Start a new login session:

```bash
su - USER
```

Also check ACLs, mount options, SELinux, AppArmor, and service-specific security policies.

***

# Practice Exercises

1. Create a file with mode `640`, then explain who can read and write it.
2. Create a directory with mode `750` and test access as owner, group member, and outsider.
3. Use `namei -l` to diagnose a missing execute permission on a parent directory.
4. Create a group-owned directory with SGID and verify group inheritance.
5. Create a sticky directory and test cross-user deletion.
6. Set a named-user ACL for read-only access and verify it with `getfacl`.
7. Add a default ACL and prove that a newly created file inherits it.
8. Assign `cap_net_bind_service` to a harmless test binary, inspect it, and remove it.
9. Compare files created under umasks `022`, `027`, and `077`.
10. Audit `/usr/bin`, `/usr/sbin`, and `/usr/local/bin` for SUID files and capabilities, then document whether each result is expected.

***

# Interview Questions

## 1. Why does a user need execute permission on a directory?

Execute permission allows traversal and access to known entries. Read permission allows listing names, but without execute permission the user generally cannot enter or access the directory contents.

## 2. What is the difference between `chmod 777` and an ACL?

`chmod 777` grants broad access to everyone through the ordinary permission classes. An ACL can grant a narrowly scoped exception to a named user or group while retaining a safer baseline.

## 3. What does the SGID bit do on a directory?

New files and subdirectories inherit the directory’s group ownership, which supports team collaboration.

## 4. Why is `usermod -aG developers USER` safer than `usermod -G developers USER`?

`-aG` appends the group. `-G` replaces the user’s supplementary-group list and may remove existing access.

## 5. Why should a service bind to a privileged port without running as root?

A capability such as `cap_net_bind_service` can grant only the required port-binding privilege, reducing the consequences of a service compromise compared with full root access.

***

# Final Mini-Project: Secure a Multi-Team Linux Server

## Objective

Build and audit a multi-team server with:

- Management, development, network, and support teams.
- Shared storage.
- Safe umask.
- ACL exceptions.
- SGID and sticky-bit directories.
- A narrowly scoped capability.
- Final permission and identity audit.

## Project task

Create groups:

```text
project-management
project-developers
project-network
project-support
project-shared
```

Create users:

```text
pm01
dev01
net01
support01
```

Create:

```text
/srv/company/dev
/srv/company/shared
/srv/company/dropbox
```

Requirements:

- Developers collaborate in `/srv/company/dev`.
- All approved departments can read `/srv/company/shared`.
- Only the shared group writes to `/srv/company/shared`.
- `/srv/company/dropbox` allows controlled file submission but prevents users deleting one another’s files.
- A named ACL gives `support01` read-only access to developer reports.
- New developer files inherit the developer group.
- A harmless test binary receives `cap_net_bind_service`, then the capability is audited and removed.
- The final system is checked for world-writable directories, SUID files, ACLs, and ownership errors.

## Reference solution

Create groups:

```bash
for group in project-management project-developers project-network project-support project-shared; do
  sudo groupadd "$group" 2>/dev/null || true
done
```

Create users:

```bash
sudo useradd -m -s /bin/bash -g project-management pm01
sudo useradd -m -s /bin/bash -g project-developers dev01
sudo useradd -m -s /bin/bash -g project-network net01
sudo useradd -m -s /bin/bash -g project-support support01
```

Add shared membership:

```bash
sudo usermod -aG project-shared pm01
sudo usermod -aG project-shared dev01
sudo usermod -aG project-shared net01
sudo usermod -aG project-shared support01
```

Set a restrictive lab umask for the project users through their shell startup files:

```bash
for user in pm01 dev01 net01 support01; do
  printf '\numask 027\n' | sudo tee -a "/home/$user/.bashrc" >/dev/null
  sudo chown "$user:$(id -gn "$user")" "/home/$user/.bashrc"
done
```

Create directories:

```bash
sudo mkdir -p /srv/company/dev /srv/company/shared /srv/company/dropbox
```

Configure development:

```bash
sudo chown root:project-developers /srv/company/dev
sudo chmod 2770 /srv/company/dev
sudo setfacl -d -m g:project-developers:rwx /srv/company/dev
sudo setfacl -d -m o::--- /srv/company/dev
```

Configure shared read access with only the shared group writing:

```bash
sudo chown root:project-shared /srv/company/shared
sudo chmod 2770 /srv/company/shared
sudo setfacl -m g:project-management:rx,g:project-developers:rx,g:project-network:rx,g:project-support:rx /srv/company/shared
sudo setfacl -m o::--- /srv/company/shared
```

Configure the dropbox:

```bash
sudo chown root:project-shared /srv/company/dropbox
sudo chmod 3770 /srv/company/dropbox
```

Grant support read-only access to developer reports:

```bash
sudo mkdir -p /srv/company/dev/reports
sudo setfacl -m u:support01:rx /srv/company/dev/reports
sudo setfacl -d -m u:support01:r-X /srv/company/dev/reports
```

Create a harmless binary for capability demonstration:

```bash
printf '#include <stdio.h>\nint main(void){puts("mini-project test");return 0;}\n' > /tmp/mini-project.c
cc /tmp/mini-project.c -o /tmp/mini-project
sudo install -o root -g root -m 0755 /tmp/mini-project /usr/local/bin/mini-project
sudo setcap cap_net_bind_service=+ep /usr/local/bin/mini-project
getcap /usr/local/bin/mini-project
```

Remove the capability after the demonstration:

```bash
sudo setcap -r /usr/local/bin/mini-project
getcap /usr/local/bin/mini-project
```

Audit the final configuration:

```bash
for user in pm01 dev01 net01 support01; do
  id "$user"
done

sudo find /srv/company -printf '%M %u %g %p\n'
sudo getfacl -R /srv/company
sudo getcap -r /usr/local/bin /opt 2>/dev/null
sudo find /srv/company -type d -perm -0002 -ls
sudo find /srv/company -type f -perm -0002 -ls
sudo find /srv/company -type f -perm -4000 -ls
sudo pwck -r
sudo grpck -r
```

Expected audit goals:

- No unintended world-writable files or directories.
- Development files inherit `project-developers`.
- Support has only the intended report access.
- The dropbox has sticky-bit deletion protection.
- The test capability is removed after testing.
- User and group databases pass consistency checks.
- Every access exception is documented and justified.
