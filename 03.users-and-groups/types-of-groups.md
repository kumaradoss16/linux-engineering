# Different Types of Groups in Linux

In Linux, a group is a collection of user accounts used to manage permissions and access to system resources. Instead of granting permissions to every user individually, an administrator can assign permissions to a group.

Linux groups are commonly understood in two ways:

1. By their relationship to a user: primary groups and supplementary groups.
2. By their purpose: user-created groups, system groups, administrative groups, and service groups.

## 1. Primary group

A primary group is the main group associated with a user's account. It is identified by a Group ID (GID).

For example:

```
id hytrel
```

Illustrative output:

```
uid=1000(hytrel) gid=1000(hytrel) groups=1000(hytrel),4(adm),27(sudo)
```

Here:

- `hytrel` - username.
- `gid=1000(hytrel)` - primary group.
- `adm` and `sudo` - supplementary groups.

Purpose: The primary group commonly determines the default group ownership of newly created files, subject to directory settings and other filesystem rules.

Useful commands:

```
id -gn hytrel
getent group hytrel
```

## 2. Supplementary group

A supplementary group is an additional group membership that gives a user access to resources permitted for that group.

For example, your Ubuntu account may belong to these groups:

```
hytrel adm cdrom sudo dip plugdev lxd
```

The exact permissions depend on system configuration.

| Group     | Typical purpose                                        |
| --------- | ------------------------------------------------------ |
| `adm`     | Access to selected system logs                         |
| `cdrom`   | Access to optical drive devices                        |
| `sudo`    | Potential authorization to run commands through `sudo` |
| `dip`     | Dial-up networking configuration                       |
| `plugdev` | Access to certain removable or plug-in devices         |
| `lxd`     | LXD management access; potentially highly privileged   |

Check supplementary groups:

```
id -nG hytrel
```

Add a user to a supplementary group:

```
sudo usermod -aG developers hytrel
```

The `-aG` options append the group membership rather than replacing the existing supplementary group list.

## 3. User-created groups

These are groups created by an administrator to organize users according to their roles or shared responsibilities.

Examples include `developers`, `testers`, `network-admins`, and `web-team`.

Create a group:

```
sudo groupadd developers
```

Add users:

```
sudo usermod -aG developers alice
sudo usermod -aG developers bob
```

Check the group:

```
getent group developers
```

Illustrative output:

```
developers:x:1002:alice,bob
```

A group does not automatically grant access to a directory. You must configure the directory's ownership and permissions too.

For example:

```
sudo mkdir -p /project
sudo chown root:developers /project
sudo chmod 2770 /project
```

This sets the group ownership to `developers` and grants the owner and group read, write, and directory-traversal permissions. The leading `2` sets the setgid bit so newly created items in that directory generally inherit its group.

## 4. System groups

System groups are generally associated with operating-system functions, system services, or restricted resource access.

Examples may include:

- `adm`
- `sys`
- `systemd-journal`
- `www-data`
- `mail`

The exact groups depend on the Linux distribution and installed software.

For example, a web server may run under the `www-data` account and group on Ubuntu and Debian systems.

Check a system group:

```
getent group www-data
```

List groups from the configured group database:

```
getent group
```

Security note: Do not assume every system group is safe to join. Some groups provide access to sensitive files, devices, or privileged services.

## 5. Service groups

Service groups are associated with applications and background services. They help isolate services from ordinary users and restrict access to application files.

Examples on Ubuntu may include:

| Group      | Typical association                  |
| ---------- | ------------------------------------ |
| `www-data` | Web server processes and files       |
| `mysql`    | MySQL or compatible database service |
| `postgres` | PostgreSQL service                   |
| `redis`    | Redis service                        |

For example:

```
getent group www-data
ps -eo user,group,comm | grep -E 'apache2|nginx'
```

The first command looks up the group. The second inspects running processes and their associated user and group names. Installed services and their configurations determine which accounts and groups are present.

## 6. Administrative and privileged groups

Some groups grant access to sensitive administrative functions.

Examples include:

- `sudo` - can authorize administrative commands according to sudo policy.
- `docker` - on many installations, membership can effectively provide root-level control of the host.
- `lxd` - membership can provide highly privileged access when LXD is configured.
- `disk` - may provide direct access to storage devices and potentially sensitive data.

Check whether a user belongs to privileged groups:

```
id hytrel
```

Check sudo permissions:

```
sudo -l
```

Avoid adding users to privileged groups unless they genuinely need that access.

## 7. Group types compared

| Type                 | Main purpose                                  | Example           |
| -------------------- | --------------------------------------------- | ----------------- |
| Primary group        | Default group association for a user          | `hytrel`          |
| Supplementary group  | Additional access through group membership    | `sudo`, `adm`     |
| User-created group   | Organize users for shared work                | `developers`      |
| System group         | Support operating-system functions            | `systemd-journal` |
| Service group        | Restrict access for applications and services | `www-data`        |
| Administrative group | Provide access to privileged capabilities     | `sudo`, `docker`  |

These categories can overlap. For example, `www-data` is both a system-created group and a service group; `sudo` is a supplementary group for some users and a privileged administrative group by purpose.

## 8. Practical Linux group administration commands

| Command                             | Purpose                                      |
| ----------------------------------- | -------------------------------------------- |
| `groups`                            | Show the current user's groups               |
| `id`                                | Show UID, primary GID, and group memberships |
| `getent group`                      | List configured group entries                |
| `getent group sudo`                 | Look up a specific group                     |
| `sudo groupadd developers`          | Create a group                               |
| `sudo groupdel developers`          | Delete a group                               |
| `sudo usermod -aG developers alice` | Add a user to a supplementary group          |
| `sudo gpasswd -d alice developers`  | Remove a user from a supplementary group     |
| `sudo chgrp developers file.txt`    | Change a file's group ownership              |
| `ls -l file.txt`                    | Inspect file permissions and ownership       |

Before deleting a group, check whether it is used by users, files, or services.

## 9. Practice on your Ubuntu Server

Run the following commands:

```
# 1. Display your group memberships
id

# 2. Find your primary group
id -gn

# 3. List your supplementary group names
id -nG

# 4. Inspect the sudo group
getent group sudo

# 5. Inspect the adm group
getent group adm

# 6. Count group entries returned by NSS
getent group | wc -l
```

Recommended learning path: Learn primary and supplementary groups first, then practise creating groups, assigning users, setting directory permissions, and using `chown`, `chgrp`, and `chmod`. These concepts are essential for Linux server administration and access control.
