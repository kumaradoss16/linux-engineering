# Linux `chmod` and `chown` Commands

The key difference: `chmod` changes a file's or directory's permissions, while `chown` changes its owner and/or group.

As a Linux System Administrator, you use these commands to control who can read, modify, execute, or manage files and directories.

## 1. Understand Linux file permissions

Before using `chmod` and `chown`, inspect a file:

```
ls -l script.sh
```

Illustrative output:

<img width="642" height="232" alt="image" src="https://github.com/user-attachments/assets/1575ca14-1045-4eb0-b147-3f957a5b2e49" />


The permission string `-rwxr-xr--` is divided into four parts:

| Characters | Meaning   | Permissions          |
| ---------- | --------- | -------------------- |
| `-`        | File type | Regular file         |
| `rwx`      | Owner     | Read, write, execute |
| `r-x`      | Group     | Read, execute        |
| `r--`      | Others    | Read only            |

The three permission categories are:

- Owner (`u`) - the user who owns the file.
- Group (`g`) - users who belong to the file's associated group.
- Others (`o`) - everyone else, subject to other access controls.

Permission letters:

| Permission    | Letter | Numeric value |
| ------------- | ------ | ------------- |
| Read          | `r`    | 4             |
| Write         | `w`    | 2             |
| Execute       | `x`    | 1             |
| No permission | `-`    | 0             |

For directories, `r` allows listing entries, `w` allows creating/removing entries when combined with appropriate execute permission, and `x` allows traversing the directory.

## 2. `chmod` - Change file permissions

`chmod` stands for change mode. It changes which read, write, and execute permissions apply to the owner, group, and others.

### A. Numeric (octal) permissions

Each permission category gets a number calculated by adding the values of its permissions.

For example:

- `7` = `4 + 2 + 1` = `rwx`
- `6` = `4 + 2` = `rw-`
- `5` = `4 + 1` = `r-x`
- `4` = `4` = `r--`

The three digits represent owner, group, and others, respectively.

| Command                 | Result      | Typical use                                                       |
| ----------------------- | ----------- | ----------------------------------------------------------------- |
| `chmod 755 script.sh`   | `rwxr-xr-x` | Executable script                                                 |
| `chmod 644 notes.txt`   | `rw-r--r--` | Ordinary text/config file                                         |
| `chmod 600 secrets.txt` | `rw-------` | Private file for its owner                                        |
| `chmod 700 private-dir` | `rwx------` | Private directory                                                 |
| `chmod 750 deploy.sh`   | `rwxr-x---` | Owner and group can access                                        |
| `chmod 777 file.txt`    | `rwxrwxrwx` | Grants all three categories all basic permissions; usually unsafe |

Examples:

```
# Make a script executable for its owner and readable/executable by others
chmod 755 script.sh

# Make a file writable only by its owner
chmod 600 secrets.txt

# Give a directory private access to its owner
chmod 700 private-dir
```

### B. Symbolic permissions

Symbolic mode uses letters instead of numeric values.

- `u` = owner/user
- `g` = group
- `o` = others
- `a` = all three categories
- `+` = add permissions
- `-` = remove permissions
- `=` = set the specified permissions exactly for those categories

| Command                      | Explanation                                                      |
| ---------------------------- | ---------------------------------------------------------------- |
| `chmod u+x script.sh`        | Add execute permission for the owner                             |
| `chmod g+w report.txt`       | Add group write permission                                       |
| `chmod o-r secret.txt`       | Remove others' read permission                                   |
| `chmod a+r document.txt`     | Add read permission for everyone                                 |
| `chmod u=rw,g=r,o= file.txt` | Set owner to read/write, group to read, others to no permissions |

For example, if `script.sh` is initially `-rw-r--r--`, then:

```
chmod u+x script.sh
ls -l script.sh
```

The resulting permissions should be:

```
-rwxr--r-- ... script.sh
```

Only the owner's execute permission was added.

### C. Apply permissions recursively

Use `-R` to apply changes to a directory tree.

```
# Give the owner full permissions and others read/execute
chmod -R 755 ~/fs-lab/project

# Remove write permission from group and others
chmod -R go-w ~/fs-lab/project
```

Important: Recursive `chmod 755` gives regular files execute permission too. That is often undesirable. For mixed directories containing scripts, documents, and configuration files, use `find` to handle each type separately:

```
# Directories: owner full access, others read/traverse
find ~/fs-lab/project -type d -exec chmod 755 {} +

# Regular files: owner read/write, others read
find ~/fs-lab/project -type f -exec chmod 644 {} +
```

These are example permissions, not universal defaults. Private data and executable files may need different settings.

## 3. `chown` - Change file ownership

`chown` stands for change owner. It changes the user who owns a file or directory and can also change its associated group.

### A. Basic syntax

```
chown [OPTIONS] OWNER[:GROUP] FILE
```

For example, suppose you have a file named `report.txt`.

```
# Change the owner
sudo chown alice report.txt

# Change the owner and group
sudo chown alice:developers report.txt

# Change only the group
sudo chown :developers report.txt
```

Replace `alice` and `developers` with actual users and groups on your system.

On most Linux systems, changing ownership requires root privileges or the appropriate capabilities, so administrators commonly use `sudo`.

### B. Practical examples

Example 1: Inspect the current owner and group

```
ls -l report.txt
stat report.txt
```

Example 2: Change the owner

```
sudo chown alice report.txt
```

The file is now owned by `alice`; its group is unchanged.

Example 3: Change the owner and group

```
sudo chown alice:developers report.txt
```

The owner becomes `alice` and the group becomes `developers`.

Example 4: Change only the group

```
sudo chown :developers report.txt
```

This changes the group without specifying a new owner. You can also use `chgrp developers report.txt` for the group-only operation.

Example 5: Change ownership recursively

```
sudo chown -R alice:developers /srv/project
```

This applies the ownership change to the directory and its contents.

Use recursive ownership changes carefully, particularly on system directories such as `/etc`, `/usr`, and `/var`. Changing ownership there indiscriminately can break services or weaken security.

Example 6: Verify the change

```
ls -ld /srv/project
ls -l /srv/project
```

- `ls -ld` shows the directory itself.
- `ls -l` shows its immediate contents.

## 4. `chmod` vs `chown`

| Feature                 | `chmod`                                          | `chown`                                |
| ----------------------- | ------------------------------------------------ | -------------------------------------- |
| Full meaning            | Change mode                                      | Change owner                           |
| Main purpose            | Change access permissions                        | Change user/group ownership            |
| Changes                 | Read, write, execute and special permission bits | Owning user and/or group               |
| Common syntax           | `chmod 640 file.txt`                             | `sudo chown alice:developers file.txt` |
| Recursive option        | `-R`                                             | `-R`                                   |
| Typical need for `sudo` | Depends on who owns the file and permissions     | Usually required to change the owner   |
| Verification            | `ls -l`, `stat`                                  | `ls -l`, `stat`                        |

### Remember the distinction

Imagine a file owned by `hytrel`, with group `developers` and permissions `640`.

```
-rw-r----- 1 hytrel developers ... report.txt
```

If you run:

```
chmod 600 report.txt
```

The ownership remains `hytrel:developers`, but permissions become `rw-------`.

If you run:

```
sudo chown root:root report.txt
```

The ownership becomes `root:root`, but the existing permission bits remain `600`.

In short: `chmod` controls the file's permission bits; `chown` controls who owns the file and its associated group.

## 5. Hands-on Linux administration lab

Try this in your home directory on Ubuntu Server. It creates test files and avoids changing system files.

```
# 1. Create a practice directory
mkdir -p ~/fs-lab/permissions-lab
cd ~/fs-lab/permissions-lab

# 2. Create a file
echo "Linux permissions practice" > report.txt

# 3. Inspect current permissions and ownership
ls -l report.txt
stat report.txt

# 4. Change permissions
chmod 640 report.txt
ls -l report.txt

# 5. Add execute permission for the owner
chmod u+x report.txt
ls -l report.txt

# 6. Inspect the final metadata
stat report.txt
```

To practise `chown`, first check which users and groups exist:

```
id
getent passwd nobody
getent group nogroup
```

On Ubuntu, `nobody` and `nogroup` commonly exist, but verify them before use. If they are present, you can test ownership changes on the practice file:

```
sudo chown nobody:nogroup report.txt
ls -l report.txt
```

Restore ownership to your own user and primary group afterward:

```
sudo chown "$(id -un):$(id -gn)" report.txt
```

This lab demonstrates both commands without requiring changes to your server's system configuration.

Recommended learning order: Start with `ls -l` and `stat`, learn permission values such as `644`, `755`, and `600`, practise symbolic modes like `u+x`, and then learn `chown` and recursive ownership management.
