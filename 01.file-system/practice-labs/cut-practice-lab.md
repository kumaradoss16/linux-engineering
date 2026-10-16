# Practical Linux Lab: `cut` Command

## 1. Create the Lab Directory

Create a directory for practicing the Linux `cut` command and navigate into it.

```bash
mkdir -p ~/fs-lab/cut-practice
cd ~/fs-lab/cut-practice
```

**Explanation:**
- `mkdir -p` creates the directory, including any missing parent directories.
- `cd` changes the current working directory.

## 2. Create the Sample File

Create a file named `users.txt` containing colon-separated fields.

```bash
printf 'hytrel:1000:Ubuntu\nalice:1001:Debain\nbob:1002:Fedore\n' > users.txt
```

**Explanation:**
- `printf` prints formatted text.
- `\n` inserts a newline after each record.
- `>` redirects the output into `users.txt`, creating the file or overwriting it if it already exists.
- Each record contains a username, UID, and operating system name separated by colons (`:`).

## 3. Display the File Contents

**Command:**

```bash
cat users.txt
```

**Output:**

```text
hytrel:1000:Ubuntu
alice:1001:Debain
bob:1002:Fedore
```

**Purpose:** The `cat` command displays the contents of `users.txt`.

## 4. Extract Usernames

**Command:**

```bash
cut -d ':' -f 1 users.txt
```

**Output:**

```text
hytrel
alice
bob
```

**Explanation:**
- `cut` extracts selected fields from each line.
- `-d ':'` specifies the colon as the field delimiter.
- `-f 1` selects the first field, which contains the usernames.
- `users.txt` is the input file.

## 5. Extract Usernames and UIDs

**Command:**

```bash
cut -d ':' -f 1,2 users.txt
```

**Output:**

```text
hytrel:1000
alice:1001
bob:1002
```

**Explanation:**
- `-d ':'` identifies the colon separator.
- `-f 1,2` selects the first and second fields.
- The output contains usernames and their corresponding UIDs.

## 6. Extract the Operating System Field

**Command:**

```bash
cut -d ':' -f 3 users.txt
```

**Output:**

```text
Ubuntu
Debain
Fedore
```

**Explanation:**
- `-f 3` selects the third field from each line.
- In this example, the third field contains the operating system name.

## 7. Extract All Fields Except the First

**Command:**

```bash
cut -d ':' --complement -f 1 users.txt
```

**Output:**

```text
1000:Ubuntu
1001:Debain
1002:Fedore
```

**Explanation:**
- `--complement` selects everything except the specified fields.
- `-f 1` identifies the first field to exclude.
- The output contains the UID and operating system fields.

## Summary

| Command | Purpose |
|---|---|
| `cut -d ':' -f 1 users.txt` | Extract usernames |
| `cut -d ':' -f 1,2 users.txt` | Extract usernames and UIDs |
| `cut -d ':' -f 3 users.txt` | Extract the third field |
| `cut -d ':' --complement -f 1 users.txt` | Extract all fields except the first |

## Key Takeaway

The Linux `cut` command is useful for extracting specific fields from structured text files. The `-d` option defines the delimiter, `-f` selects fields, and `--complement` selects all fields except those specified.

**Practical applications:** Processing colon-separated account records, extracting values from configuration files, and performing basic text processing during Linux system administration.
