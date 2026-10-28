`grep` is a Linux command used to search for text patterns in files or command output. It is commonly used for log analysis, configuration troubleshooting, and finding errors on Linux servers.

## 1. Basic `grep` commands

| Command                              | Purpose                                                       |
| ------------------------------------ | ------------------------------------------------------------- |
| `grep "error" file.txt`              | Search for `error` in a file.                                 |
| `grep -i "error" file.txt`           | Search case-insensitively.                                    |
| `grep -n "error" file.txt`           | Display matching lines with line numbers.                     |
| `grep -v "error" file.txt`           | Display lines that do not match.                              |
| `grep -c "error" file.txt`           | Count matching lines.                                         |
| `grep -w "root" file.txt`            | Match the whole word `root`.                                  |
| `grep -r "error" /path/`             | Search recursively through files in a directory.              |
| `grep -l "error" *.log`              | List filenames containing the match.                          |
| `grep -E "error\\|warning" file.txt` | Search for either pattern using extended regular expressions. |

Note: In the last example, use this actual command to search for either word:

```
grep -E "error|warning" file.txt
```

## 2. Practical examples

### Search a file

```
grep "failed" auth.log
```

This displays lines containing `failed`. By default, matching is case-sensitive.

### Search without case sensitivity

```
grep -i "failed" auth.log
```

This matches `failed`, `Failed`, and `FAILED`.

### Display line numbers

```
grep -n "error" system.log
```

Example output:

```
12: Database connection error
27: Service error detected
```

### Search multiple words

```
grep -E "error|warning|failed" system.log
```

This displays lines containing any of those three patterns.

## 3. Use `grep` with pipes

You can search the output of another Linux command.

Find error messages in system logs:

```
journalctl -b | grep -i "error"
```

Find processes containing `ssh`:

```
ps aux | grep "ssh"
```

The second command may also match the `grep` process itself. To avoid that, you can use:

```
pgrep -a ssh
```

## 4. Linux server troubleshooting

Search for SSH authentication failures:

```
sudo grep -i "failed password" /var/log/auth.log
```

Search for errors in the system journal:

```
journalctl -b | grep -i "error"
```

Search for a setting in a configuration file:

```
grep -n "Port" /etc/ssh/sshd_config
```

The exact log file depends on your distribution and logging configuration.

## 5. Useful options to remember

| Option | Meaning                                  |
| ------ | ---------------------------------------- |
| `-i`   | Ignore case                              |
| `-n`   | Show line numbers                        |
| `-v`   | Invert match                             |
| `-c`   | Count matching lines                     |
| `-r`   | Search recursively                       |
| `-w`   | Match whole words                        |
| `-E`   | Use extended regular expressions         |
| `-A 3` | Show 3 lines after each match            |
| `-B 3` | Show 3 lines before each match           |
| `-C 3` | Show 3 lines before and after each match |

## 6. Try this hands-on lab

Create a sample log file:

```
cat > app.log <<'EOF'
INFO Application started
WARNING Disk usage is high
ERROR Database connection failed
INFO User logged in
ERROR Permission denied
EOF
```

Now practise:

```
grep "ERROR" app.log
grep -n "ERROR" app.log
grep -i "warning" app.log
grep -c "ERROR" app.log
grep -v "INFO" app.log
grep -E "ERROR|WARNING" app.log
```

Key takeaway: `grep` searches text, `journalctl` retrieves system logs, and combining them with a pipe (`|`) lets you filter logs efficiently.
