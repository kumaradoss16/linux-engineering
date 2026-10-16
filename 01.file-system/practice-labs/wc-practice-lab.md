# Linux `wc` Command: Practical Examples

The Linux `wc` (word count) command counts lines, words, bytes, characters, and the length of the longest line in a file. It is useful for inspecting text files, configuration files, logs, and system information.

In this lab, we use `/etc/passwd`, a system file containing user account information.

## 1. Count Lines

**Command:**

```bash
wc -l /etc/passwd
```

**Output:**

```text
54 /etc/passwd
```

**Explanation:**

- `wc` runs the word-count utility.
- `-l` counts newline characters, normally corresponding to the number of lines in a text file.
- `/etc/passwd` is the input file.

**Result:** The command reports 54 lines in `/etc/passwd`.

## 2. Count Words

**Command:**

```bash
wc -w /etc/passwd
```

**Output:**

```text
94 /etc/passwd
```

**Explanation:**

- `-w` counts words, defined by `wc` as sequences of non-whitespace characters.
- The command reports 94 words in the file.

**Result:** `/etc/passwd` contains 94 whitespace-separated word sequences according to `wc`.

## 3. Count Bytes

**Command:**

```bash
wc -c /etc/passwd
```

**Output:**

```text
3123 /etc/passwd
```

**Explanation:**

- `-c` counts bytes in the file.
- A byte is a unit of storage, typically 8 bits.

**Result:** The file contains 3,123 bytes.

## 4. Count Characters

**Command:**

```bash
wc -m /etc/passwd
```

**Output:**

```text
3123 /etc/passwd
```

**Explanation:**

- `-m` counts characters according to the current locale's character encoding.
- In UTF-8, one character may occupy multiple bytes.

**Result:** This output reports 3,123 characters. Because the character count equals the byte count here, the file's contents are consistent with one-byte characters under the locale used for this command.

## 5. Find the Length of the Longest Line

**Command:**

```bash
wc -L /etc/passwd
```

**Output:**

```text
99 /etc/passwd
```

**Explanation:**

- `-L` reports the length of the longest line.
- The result is measured in characters under GNU `wc`.

**Result:** The longest line in `/etc/passwd` has a reported length of 99 characters.

## Summary

| Option | Purpose | Example result |
|---|---|---:|
| `-l` | Count lines | 54 |
| `-w` | Count words | 94 |
| `-c` | Count bytes | 3123 |
| `-m` | Count characters | 3123 |
| `-L` | Find the longest line's length | 99 |

*Note: These output values are from the recorded terminal session. Counts can differ across systems because `/etc/passwd` contents vary.*

## Practical Applications

The `wc` command is useful for:

- Counting user-account records in `/etc/passwd`.
- Measuring the size of text files.
- Checking the approximate volume of log files.
- Counting lines in scripts and configuration files.
- Finding unusually long lines during text-file investigation.

**Key takeaway:** Use `wc -l` for lines, `wc -w` for words, `wc -c` for bytes, `wc -m` for characters, and `wc -L` for the longest line.
