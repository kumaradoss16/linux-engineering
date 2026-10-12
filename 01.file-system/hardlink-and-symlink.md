# Linux Symlink vs Hard Link - Practical Tutorial

The key difference: A hard link is another directory entry that refers to the same inode as the original file. A symbolic link (symlink) is a separate filesystem object that stores a path pointing to another file or directory.

Both are useful for Linux system administration, application deployments, configuration management, and log handling.

## 1. What is a hard link?

A hard link is an additional name for the same underlying file data. The original filename and the hard-link filename point to the same inode.

HARD LINK STRUCTURE

Directory entries

`report.txt`

`report-backup.txt`

Same inode

Same file data and metadata

Deleting one filename does not remove the data while another hard link still exists.

### Example: Create a hard link

First, create a practice directory and a file.

```
mkdir -p ~/linux-filesystem-labs/links-lab
cd ~/linux-filesystem-labs/links-lab

echo "Linux hard link demonstration" > report.txt
```

Create a hard link named `report-backup.txt`:

```
ln report.txt report-backup.txt
```

The general syntax is:

```
ln SOURCE LINK_NAME
```

Here:

- `ln` creates a hard link by default.
- `report.txt` is the existing file.
- `report-backup.txt` is the new directory entry.

### Verify the inode numbers

```
ls -li report.txt report-backup.txt
```

Example output:

```
123456 -rw-rw-r-- 2 user user 30 Oct 12 13:00 report.txt
123456 -rw-rw-r-- 2 user user 30 Oct 12 13:00 report-backup.txt
```

The inode number, permissions, size, and link count shown here are illustrative.

Notice:

- Both filenames have the same inode number (`123456`).
- The link count is `2`, because two directory entries refer to that inode.
- Both names refer to the same underlying file content.

You can also check the inode directly:

```
stat report.txt
stat report-backup.txt
```

Look for the `Inode` and `Links` fields.

### Modify a hard-linked file

```
echo "New line added through hard link" >> report-backup.txt
cat report.txt
```

The new line appears when you read `report.txt`, because both names refer to the same file data.

### Delete the original filename

```
rm report.txt
cat report-backup.txt
```

The content is still available through `report-backup.txt`. Removing `report.txt` removes only that directory entry; the file's data remains accessible through the remaining hard link.

Real-world relevance: Hard links can provide multiple names for the same file without duplicating its contents. However, they are not independent backups: changes made through one name affect the same data.

## 2. What is a symbolic link (symlink)?

A symbolic link is a separate filesystem object that stores a path to another file or directory. It behaves similarly to a shortcut, but it is implemented as a filesystem link.

SYMLINK STRUCTURE

`report-link.txt`

Symlink: stores a target path

`report.txt`

Target file and its own inode

A symlink has its own inode. Its target may be a file or directory, and the target may not exist.

### Example: Create a symbolic link

Create a file:

```
cd ~/linux-filesystem-labs/links-lab

echo "Linux symbolic link demonstration" > report.txt
```

Create a symlink:

```
ln -s report.txt report-link.txt
```

The syntax is:

```
ln -s TARGET LINK_NAME
```

- `ln` creates a link.
- `-s` requests a symbolic link.
- `report.txt` is the target path.
- `report-link.txt` is the new symlink.

### Verify the symlink

```
ls -l report-link.txt
```

Example output:

```
lrwxrwxrwx 1 user user 10 Oct 12 13:00 report-link.txt -> report.txt
```

The leading `l` indicates a symbolic link, and `-> report.txt` shows its stored target path.

Check the target path:

```
readlink report-link.txt
```

Example:

```
report.txt
```

Read the target's content through the symlink:

```
cat report-link.txt
```

This works because the relative target `report.txt` is resolved relative to the directory containing `report-link.txt`.

### Modify the target through the symlink

```
echo "Added through symlink" >> report-link.txt
cat report.txt
```

The added line appears in `report.txt`. The symlink redirects access to the target file; it does not create an independent copy.

### What happens if the target is deleted?

```
rm report.txt
ls -l report-link.txt
cat report-link.txt
```

The symlink still exists, but `cat` reports that the target cannot be found. This is called a broken symlink or dangling symlink.

Find symlinks whose targets cannot be resolved under your lab directory:

```
find ~/linux-filesystem-labs/links-lab -xtype l
```

This is useful for locating broken links in application directories or configuration trees.

## 3. Hard link vs symlink: Key differences

| Feature            | Hard link                                                 | Symbolic link                                         |
| ------------------ | --------------------------------------------------------- | ----------------------------------------------------- |
| What it references | The same inode as the original file                       | A stored target path                                  |
| Inode              | Same inode as the original                                | Its own inode                                         |
| Target deleted     | Data remains if another hard link exists                  | Link becomes dangling                                 |
| Directories        | Normally not permitted for ordinary users                 | Can link to directories                               |
| Across filesystems | Not permitted across filesystem boundaries                | Can cross filesystem boundaries                       |
| Relative paths     | Not applicable to the link relationship                   | Relative targets resolve from the symlink's directory |
| Link count         | Increases the target inode's hard-link count              | Does not increase the target's hard-link count        |
| Storage            | Additional directory entry; no separate copy of file data | Separate link object storing the path                 |
| Best suited for    | Multiple names for the same file data                     | Shortcuts, directory aliases, and deployment paths    |

Important: A hard link does not depend on the original filename. It refers to the same inode, provided the filesystem supports hard links. A symlink depends on its target path resolving correctly.

## 4. Practical lab: Compare both link types

Run this complete example in your lab directory:

```
cd ~/linux-filesystem-labs/links-lab

printf "Initial content\n" > original.txt

ln original.txt hardlink.txt
ln -s original.txt symlink.txt
```

Inspect the objects:

```
ls -li original.txt hardlink.txt symlink.txt
```

You should observe that `original.txt` and `hardlink.txt` share an inode number, while `symlink.txt` has a different inode number.

Inspect their types:

```
stat -c '%n | inode=%i | type=%F | links=%h' \
  original.txt hardlink.txt symlink.txt
```

The hard link and original share the inode and hard-link count. The symlink has its own inode and usually has a link count of `1`.

Compare reading and writing:

```
echo "Change through hard link" >> hardlink.txt
echo "Change through symlink" >> symlink.txt

cat original.txt
```

Both added lines appear in `original.txt` because both links ultimately access the same underlying file data.

Now remove the original filename:

```
rm original.txt

cat hardlink.txt
cat symlink.txt
```

The hard link still works. The symlink normally fails because its stored target, `original.txt`, no longer exists.

To reset the lab, recreate the target:

```
printf "Restored target\n" > original.txt
cat symlink.txt
```

The symlink works again because the path it stores now resolves to a file.

## 5. Real-world system administration uses

[Using ‘symlinks’ to create embedded CLI applications in C/C++ | by Saket Upadhyay | Medium](https://images.openai.com/static-rsc-4/9-5cxxnk9mCFlYhVxUftzkEnOvelciNk9EpSJjZlGe4tjDWXgy6WHRd5GuuItnICebojPvylR4G0-v5eG7ztL19HGZZW3yBxDo6L-UK1spELCmYoGi7SSFtHpiYnuTv22GdXOi-n4Ph7MpzzmV6BwBFgnJEj0skTC8D_LT0x8kM?purpose=inline)

Symlinks: application deployments

An application may use a stable path such as `/var/www/current` that points to a versioned release directory. An administrator can change the symlink to select a different release without moving every application file.

[Linux Administration: Ubuntu Linux Tutorial für Einsteiger – HowtoForge](https://images.openai.com/static-rsc-4/U5QUhcqdCrz-H-gheAFiQxjrsjKab-Oj_UasX1ehOn77xpjpnKsqM5T_4w-9a6ZEhW_IiX6PAG-9TzbPW324TbtnPnqSuQ9Votd3YtBcYbBb5COKExZjQ6HY-rftBz333M6wzFkcHC9nOvMCX-8J1fEAMDin0dBDAPrG_E3HuWs?purpose=inline)

Hard links: multiple names for shared data

Hard links can expose the same file under multiple filenames within one filesystem. They are less commonly used for routine application deployment and are not a substitute for backups.

[Pracownia Systemów Operacyjnych](https://images.openai.com/static-rsc-4/R8btEyV1e-4KBiPUKSWhJYOqkYq6B4HNMc7yF6_nXOR3SUz27N2AlTI0FlvtkpcYLncxqft0zfOLjWFqH7QpZpog5eS6THVqBzhmVsgCKKp4CaZMZAnig8gJyF-j2exa8WrmoNlyhGMWkDbQrbwVpEv5nq-lNy4yPg5PxQoWwsw?purpose=inline)

Symlinks: configuration and directory organization

Symlinks can provide convenient paths to shared configuration files or directories. Before changing one on a production server, inspect the target and understand which services depend on it.

## 6. Useful commands to remember

| Command                | Purpose                                                     |
| ---------------------- | ----------------------------------------------------------- |
| `ln file hardlink`     | Create a hard link                                          |
| `ln -s target symlink` | Create a symbolic link                                      |
| `ls -li file1 file2`   | Compare inode numbers                                       |
| `ls -l symlink`        | Display a symlink and its target                            |
| `readlink symlink`     | Display the stored target path                              |
| `readlink -f symlink`  | Resolve a path to its canonical absolute path when possible |
| `stat file`            | Inspect inode, link count, and metadata                     |
| `find PATH -type l`    | Find symbolic links                                         |
| `find PATH -xtype l`   | Find symlinks with unresolved targets                       |
| `unlink linkname`      | Remove one link entry                                       |

Be careful when deleting links: `rm symlink.txt` removes the symlink itself, not the target file. Avoid using recursive deletion or force options until you have verified the exact path.

Recommended learning order: Create both link types, compare their inode numbers with `ls -li`, modify the file through each link, then remove the original filename and observe the different outcomes. This demonstrates the core concept much more clearly than memorizing the definitions.
