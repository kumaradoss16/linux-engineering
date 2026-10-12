In Linux, `realpath` and `readlink` are useful commands for investigating file paths and symbolic links. They sound similar, but they serve different purposes.

Important: The standard Linux command is `readlink`, not `reallink`.

## 1. `realpath` - Resolve a path to its absolute path

The `realpath` command converts a pathname into an absolute, canonical path, resolving components such as `.` and `..` and normally resolving symbolic links.

Syntax:

```
realpath PATH
```

Example:

```
cd ~/linux-filesystem-labs
realpath files-lab
```

Possible output:

```
/home/hytrel/linux-filesystem-labs/files-lab
```

Here, `realpath` displays the full path instead of the relative path `files-lab`.

### More examples

| Command                            | Purpose                                              |
| ---------------------------------- | ---------------------------------------------------- |
| `realpath example.txt`             | Display the canonical absolute path of a file.       |
| `realpath ..`                      | Resolve the parent directory to an absolute path.    |
| `realpath ~/linux-filesystem-labs` | Resolve a path starting from your home directory.    |
| `realpath -e example.txt`          | Require the entire path to exist.                    |
| `realpath -m missing/file.txt`     | Resolve a path even if some components do not exist. |

Note that `realpath` options and behavior can vary across implementations.

## 2. `readlink` - Inspect symbolic links

A symbolic link, or symlink, is a filesystem object that points to another pathname.

Syntax:

```
readlink LINK_NAME
```

### Practical example

Create a lab directory and a file:

```
mkdir -p ~/linux-filesystem-labs/links-lab
cd ~/linux-filesystem-labs/links-lab

echo "Linux filesystem practice" > example.txt
ln -s example.txt example-link
```

Inspect the link:

```
readlink example-link
```

Output:

```
example.txt
```

This tells you that the symbolic link stores the relative target pathname `example.txt`.

To display the link itself and its target:

```
ls -l example-link
```

Possible output:

```
example-link -> example.txt
```

### Common `readlink` usage

| Command                    | Purpose                                                                        |
| -------------------------- | ------------------------------------------------------------------------------ |
| `readlink example-link`    | Display the link's stored target pathname.                                     |
| `readlink -f example-link` | Resolve the link and its target to a canonical absolute path, when resolvable. |
| `readlink -e example-link` | Resolve the path only if all components exist.                                 |
| `readlink -m example-link` | Canonicalize the path even if some components do not exist, where supported.   |

## 3. Compare `realpath` and `readlink`

Consider this structure:

```
links-lab/
├── example.txt
└── example-link -> example.txt
```

Run:

```
realpath example-link
readlink example-link
readlink -f example-link
```

Example output:

```
/home/hytrel/linux-filesystem-labs/links-lab/example.txt
example.txt
/home/hytrel/linux-filesystem-labs/links-lab/example.txt
```

| Feature                               | `realpath`         | `readlink`                                       |
| ------------------------------------- | ------------------ | ------------------------------------------------ |
| Main purpose                          | Resolve a pathname | Read a symbolic link's target                    |
| Displays an absolute path             | Yes, by default    | With options such as `-f`                        |
| Shows the stored link target directly | No                 | Yes, without resolution options                  |
| Useful for checking path locations    | Yes                | Sometimes                                        |
| Useful for diagnosing symbolic links  | Yes                | Yes, especially for inspecting the stored target |

## 4. Real-world system administration examples

Scenario A: Find the actual location of a configuration file

```
realpath /etc/os-release
```

This helps identify the resolved filesystem path.

Scenario B: Investigate a symbolic link

```
ls -l /etc/localtime
readlink /etc/localtime
readlink -f /etc/localtime
```

`/etc/localtime` is commonly a symbolic link to timezone data, although systems may configure it differently.

Scenario C: Detect a broken symbolic link

```
readlink example-link
readlink -e example-link
```

The first command may display the stored target even if the target is missing. The second fails if the link cannot be resolved to an existing path.

## Remember

- `realpath` answers: “What is the canonical absolute path for this pathname?”
- `readlink` answers: “What target pathname does this symbolic link store?”

These commands are particularly useful when troubleshooting broken symbolic links, locating configuration files, and investigating Linux filesystem paths.
