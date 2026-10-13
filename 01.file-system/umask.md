# Linux `umask` Command Explained

`umask` controls the default permissions that are removed when new files and directories are created. As a Linux System Administrator, you use it to help prevent newly created files from being more accessible than intended.

For example, a `umask` of `022` typically results in new regular files having permissions `644` and new directories having permissions `755`.

## 1. What is `umask`?

`umask` stands for user file-creation mode mask.

When you create a file or directory, Linux applications usually request a set of initial permissions. The `umask` removes specified permission bits from those requested permissions.

Typical starting permission modes are:

- Regular files: `666` (`rw-rw-rw-`)
- Directories: `777` (`rwxrwxrwx`)

The operating system does not normally grant execute permission to newly created regular files automatically. The `umask` removes permissions from the requested mode; it does not add permissions.

## 2. Check your current `umask`

Run:

```
umask
```

Example output:

```
0022
```

This means the mask is `022` (the leading zero indicates octal notation).

To display it symbolically, run:

```
umask -S
```

Illustrative output:

```
u=rwx,g=rx,o=rx
```

This symbolic output represents the permissions that remain allowed by the mask for the three user classes. It is not a prediction of the exact permissions for every new file: regular files usually start with `666`, while directories usually start with `777`.

## 3. How does `umask` calculate permissions?

For the common basic permission bits, the effective permissions are calculated by removing the mask bits from the application's requested mode.

<img width="642" height="307" alt="image" src="https://github.com/user-attachments/assets/bc4e0817-33ad-4e6a-a47f-6cfe0bd4da3e" />

The displayed subtraction is a convenient shortcut for these common examples. Technically, the mask works bit by bit: permission bits set in the mask are removed from the requested mode.

## 4. Common `umask` values

| `umask` | Typical new file                | Typical new directory | General effect                            |
| ------- | ------------------------------- | --------------------- | ----------------------------------------- |
| `000`   | `666` (`644` is not the result) | `777`                 | No basic permissions masked               |
| `002`   | `664`                           | `775`                 | Group can write                           |
| `022`   | `644`                           | `755`                 | Group and others cannot write             |
| `027`   | `640`                           | `750`                 | Others receive no basic permissions       |
| `077`   | `600`                           | `700`                 | Only the owner receives basic permissions |

A `umask` of `000` means no basic permission bits are removed. It does not guarantee every program creates files with those modes; an application may request more restrictive permissions.

Practical recommendation: `022` is common for general-purpose user environments. `077` is useful for private files and directories, such as those containing credentials. Shared projects may need `002`, appropriate group membership, and possibly setgid directories or default ACLs.

## 5. Change `umask`

### Example 1: Set `umask` to `027`

```
umask 027
```

This affects subsequently created files and directories in the current shell and processes that inherit its mask.

Check it:

```
umask
```

Create test objects:

```
mkdir umask-lab
touch umask-lab/example.txt
mkdir umask-lab/example-dir
```

Inspect the resulting permissions:

```
ls -ld umask-lab umask-lab/example-dir
ls -l umask-lab/example.txt
```

With `umask 027`, typical results are:

```
drwxr-x--- ... umask-lab
drwxr-x--- ... example-dir
-rw-r----- ... example.txt
```

The permissions of `umask-lab` itself may differ because it was created before the mask was changed.

### Example 2: Set a private default

```
umask 077
```

Typical new files will have `600` permissions, and directories will have `700`.

This is useful when creating private working files or sensitive local data.

### Example 3: Restore the common default

```
umask 022
```

This changes the mask for the current shell; it does not retroactively change existing files.

## 6. `umask` vs `chmod`

| Feature                 | `umask`                               | `chmod`                                 |
| ----------------------- | ------------------------------------- | --------------------------------------- |
| Purpose                 | Restricts permissions on new objects  | Changes permissions on existing objects |
| When it acts            | During file/directory creation        | When you run the command                |
| Example                 | `umask 027`                           | `chmod 640 report.txt`                  |
| Affects existing files? | No                                    | Yes, on specified targets               |
| Typical scope           | Current shell and inherited processes | Selected file or directory              |

Remember: `umask` establishes a default restriction; `chmod` explicitly changes the permissions of an existing object.

## 7. Make a `umask` persistent

A `umask` set directly in your terminal usually lasts for that shell session. To apply it to future interactive Bash sessions for your user, you can add it to `~/.bashrc`.

For example:

```
nano ~/.bashrc
```

Add this line:

```
umask 027
```

Save the file, then reload the configuration:

```
source ~/.bashrc
umask
```

Server administration note: `~/.bashrc` is not a universal setting for all services, SSH sessions, cron jobs, or users. Login configuration, PAM settings, systemd service settings, and application behavior can affect the effective mask in other contexts. For system-wide policy, choose the appropriate configuration mechanism for the processes you need to control rather than assuming `.bashrc` applies everywhere.

## 8. Hands-on practice

Try this sequence in a test directory:

```
mkdir -p ~/fs-lab/umask-practice
cd ~/fs-lab/umask-practice

# Inspect your current mask
umask

# Set a restrictive mask
umask 077

# Create a file and directory
touch private.txt
mkdir private-dir

# Inspect their permissions
ls -l private.txt
ls -ld private-dir

# Restore the common mask for this shell
umask 022
```

Expected basic permissions after creation with `umask 077` are:

- `private.txt`: `600` (`rw-------`)
- `private-dir`: `700` (`rwx------`)

Exact results can differ if an application explicitly requests stricter permissions or changes permissions after creation.

### Key takeaway

- `umask` determines which permission bits are withheld from newly created files and directories.
- `umask 022` commonly yields files `644` and directories `755`.
- `umask 077` commonly yields files `600` and directories `700`.
- `chmod` changes permissions on existing objects; changing `umask` does not.
