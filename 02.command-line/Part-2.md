# Chapter 4: Linux Command Line

## Part 2: Quoting, Wildcards, Text Streams, Redirection, Pipes, Chaining, and Exit Codes

> **Lab safety**
>
> Run all commands only in a disposable Ubuntu 24.04 Server VM or another lab system that you own or are authorized to use. Use `~/cli-lab` for all practice files.
>
> Do not modify or delete files under `/etc`, `/var/log`, `/usr`, or `/`.
>
> Before every use of `rm`, especially with wildcards or recursive options, run:
>
> ```bash
> pwd
> ls -lah
> ```
>
> Prefer read-only commands during investigations.

***

## 4.6 Quoting

**Level: Beginner**

> ## The 20% that matters
>
> - Use double quotes around variables: `"$variable"`.
> - Use quotes around filenames that may contain spaces.
> - Single quotes preserve text literally.
> - Double quotes allow variable expansion and command substitution.
> - A backslash escapes the next character.

## Concept

Bash treats some characters as special syntax. Examples include:

```text
$  "  '  \  *  ?  [  ]  >  <  |  &  ;  (  )
```

Quoting controls whether Bash interprets these characters or passes them as literal text.

Without correct quoting, Bash may:

- Split one value into multiple arguments.
- Expand a wildcard into many filenames.
- Expand a variable.
- Run command substitution.
- Redirect output unexpectedly.

Bash performs several expansions before executing a command. These include parameter expansion, command substitution, word splitting, filename expansion, and quote removal. The order matters when a command combines variables, quotes, and wildcards. [gnu](https://www.gnu.org/software/bash/manual/bash.html)

## Quoting methods

| Quoting method | Main behavior |
|---|---|
| Single quotes `'...'` | Preserve almost everything literally. |
| Double quotes `"..."` | Allow variable expansion and command substitution. |
| Backslash `\` | Escapes the next character. |
| No quotes | Allows word splitting and filename expansion. |

## Single quotes

```bash
echo '$HOME'
```

**Sample output:**

```text
$HOME
```

Bash does not expand `$HOME` inside single quotes.

Single quotes are useful when you want exact text:

```bash
grep '$HOME' file.txt
```

This searches for the literal characters `$HOME`.

Single quotes cannot contain an unescaped single quote. To include one, use another quoting strategy.

## Double quotes

```bash
echo "$HOME"
```

**Sample output:**

```text
/home/student
```

Bash expands `$HOME` inside double quotes.

Command substitution also works inside double quotes:

```bash
echo "Current directory: $(pwd)"
```

**Sample output:**

```text
Current directory: /home/student/cli-lab
```

## Backslash escaping

```bash
echo "A quote: \""
echo 'A backslash: \'
```

The first command uses `\"` so the double quote becomes literal inside double quotes.

Backslashes behave differently inside single quotes. A backslash is usually literal inside single quotes.

## Filenames with spaces

Create a safe practice file:

```bash
cd ~/cli-lab
touch "A file with spaces.txt"
```

List it:

```bash
ls -l "A file with spaces.txt"
```

**Sample output:**

```text
-rw-r--r-- 1 student student 0 Oct 16 17:50 A file with spaces.txt
```

If you omit the quotes:

```bash
ls -l A file with spaces.txt
```

Bash passes four separate arguments:

```text
A
file
with
spaces.txt
```

`ls` then tries to find four separate paths.

You can also escape spaces:

```bash
ls -l A\ file\ with\ spaces.txt
```

Double quotes are easier to read.

Remove only the practice file after verifying your location:

```bash
pwd
ls -lah
rm -i "A file with spaces.txt"
```

**Sample output:**

```text
rm: remove regular empty file 'A file with spaces.txt'?
```

Type `y` only after confirming the path.

## Empty variables

Consider:

```bash
FILE=""
cat "$FILE"
```

The quotes preserve the empty value as one argument. The command will usually report that an empty pathname is invalid.

Without quotes:

```bash
cat $FILE
```

The empty variable disappears during word splitting. `cat` may then read from standard input and wait for keyboard input.

This is one reason to quote variables consistently.

## Real-world use case

A deployment script receives a directory path from an environment variable:

```bash
APP_DIR="$HOME/cli-lab/application data"
mkdir -p "$APP_DIR"
ls -lah "$APP_DIR"
```

Quoting keeps the path as one argument even though it contains spaces.

## Common mistakes

1. Using single quotes around a variable that must expand.
2. Leaving file paths and variables unquoted.
3. Assuming a backslash behaves the same way inside and outside single quotes.

***

## 4.7 Wildcards and Globbing

**Level: Beginner**

> ## The 20% that matters
>
> - Globbing is filename matching performed by the shell.
> - `*` matches any number of characters, including none.
> - `?` matches exactly one character.
> - `[...]` matches one character from a set or range.
> - Inspect wildcard matches before using a command that changes files.

## Concept

A wildcard, also called a glob pattern, selects filenames.

Bash expands the pattern before starting the command. The command usually receives the resulting filenames, not the original pattern.

Example:

```bash
ls *.txt
```

If the directory contains:

```text
a.txt
b.txt
notes.md
```

Bash effectively runs:

```bash
ls a.txt b.txt
```

Globbing is not the same as a regular expression.

- Globbing usually selects pathnames.
- Regular expressions describe text patterns for tools such as `grep`.

## Common glob patterns

| Pattern | Meaning | Example matches |
|---|---|---|
| `*` | Any string of characters | `a`, `report.txt`, `server.log` |
| `?` | Exactly one character | `a`, `1` in `file?.txt` |
| `[abc]` | One character from `a`, `b`, or `c` | `a.txt`, `b.txt` |
| `[0-9]` | One character in a range | `report1.txt` |
| `[!2]` | One character other than `2` in Bash | `report1.txt`, `report3.txt` |
| `*.txt` | Names ending in `.txt` | `notes.txt` |
| `report?.txt` | `report`, one character, `.txt` | `report1.txt` |

## Demonstration

Create controlled practice files:

```bash
cd ~/cli-lab
touch report1.txt report2.txt report3.txt notes.txt data.csv
```

List all text files:

```bash
ls *.txt
```

**Sample output:**

```text
notes.txt  report1.txt  report2.txt  report3.txt
```

List one-character report variants:

```bash
ls report?.txt
```

**Sample output:**

```text
report1.txt  report2.txt  report3.txt
```

Try an exact filename that does not exist:

```bash
ls report.txt
```

**Sample output:**

```text
ls: cannot access 'report.txt': No such file or directory
```

Match reports except `report2.txt`:

```bash
ls report[!2].txt
```

**Sample output:**

```text
report1.txt  report3.txt
```

Match multiple extensions:

```bash
ls *.txt *.csv
```

**Sample output:**

```text
data.csv  notes.txt  report1.txt  report2.txt  report3.txt
```

## Hidden files

The pattern `*` does not normally match names beginning with `.`.

```bash
touch .hidden-note
ls *
ls .*
```

`ls *` normally excludes `.hidden-note`. The pattern `.*` includes hidden names, but it also matches `.` and `..` in many contexts. Use it carefully.

Prefer:

```bash
ls -lah
```

The `-a` option explicitly asks `ls` to show hidden entries.

## Wildcard safety

Never begin with:

```bash
rm *
```

The shell expands `*` before `rm` runs. The command may remove every matching file in the current directory.

For a safe lab workflow:

```bash
pwd
ls -lah
ls -lah *.txt
```

Review the exact matches. Then remove one known file interactively:

```bash
rm -i report1.txt
```

**Sample output:**

```text
rm: remove regular empty file 'report1.txt'?
```

Remove remaining practice files individually or with a carefully verified pattern:

```bash
pwd
ls -lah
rm -i report2.txt report3.txt notes.txt data.csv .hidden-note
```

## Globbing versus regular expressions

This glob matches filenames:

```bash
ls report?.txt
```

This regular expression searches text:

```bash
grep -E '^report[0-9]+\.txt$' names.txt
```

The glob `?` means exactly one filename character. In an extended regular expression, `?` usually modifies the item before it. The two pattern languages are not interchangeable.

## Real-world use case

Before archiving application reports:

```bash
pwd
ls -lah "$HOME/cli-lab/reports"
find "$HOME/cli-lab/reports" -maxdepth 1 -type f -name '*.txt' -print
```

Inspect the result before copying or removing anything.

## Common mistakes

1. Assuming `*` matches hidden files.
2. Running `rm` with a wildcard before inspecting the expanded matches.
3. Treating glob patterns as regular expressions.

***

## 4.8 Standard Input, Output, and Error

**Level: Beginner**

> ## The 20% that matters
>
> - `stdin` is file descriptor `0`.
> - `stdout` is file descriptor `1`.
> - `stderr` is file descriptor `2`.
> - Normal output and error output are separate streams.
> - Redirection and pipes connect these streams to files, terminals, or other commands.

## Concept

Unix-like programs use standard streams so they do not need to know whether their input comes from a keyboard, a file, or another program.

| Stream | Name | Descriptor | Typical default |
|---|---|---:|---|
| Input | Standard input | `0` | Keyboard |
| Output | Standard output | `1` | Terminal |
| Error | Standard error | `2` | Terminal |

## Text stream diagram

```text
Keyboard -> stdin -> command -> stdout -> terminal or file
                          -> stderr -> terminal or file
```

A command can read from standard input, write normal results to standard output, and write diagnostic messages to standard error.

## Demonstration

### Standard output

```bash
ls -lah
```

The normal directory listing is standard output.

### Redirect standard output

```bash
ls -lah > listing.txt
```

- `>` redirects descriptor `1`, standard output.
- If `listing.txt` exists, Bash truncates it before running `ls`.
- If it does not exist, Bash creates it in the current directory.

Read the result:

```bash
cat listing.txt
```

**Sample output:**

```text
total 20K
drwxr-xr-x 2 student student 4.0K Oct 16 17:55 .
drwxr-xr-x 8 student student 4.0K Oct 16 17:48 ..
-rw-r--r-- 1 student student  180 Oct 16 17:55 listing.txt
```

### Standard error

```bash
ls /nonexistent-directory
```

**Sample output:**

```text
ls: cannot access '/nonexistent-directory': No such file or directory
```

The error message is written to standard error.

### Redirect standard error

```bash
ls /nonexistent-directory 2> error.txt
```

- `2>` redirects descriptor `2`, standard error.
- The normal output remains connected to the terminal.
- The error message goes to `error.txt`.

Read it:

```bash
cat error.txt
```

**Sample output:**

```text
ls: cannot access '/nonexistent-directory': No such file or directory
```

## Why separate streams matter

Suppose a command processes many paths:

```bash
find ~/cli-lab -type f -readable
```

Normal matches go to standard output. Permission or traversal errors may go to standard error.

You can save results and errors separately:

```bash
find ~/cli-lab -type f -readable > readable-files.txt 2> find-errors.txt
```

This allows later review without mixing successful results and diagnostics.

## Real-world use case

A monitoring script may need:

- Standard output for a report.
- Standard error for an operator or error log.
- A non-zero exit code to indicate failure.

Keeping streams separate makes automation easier to debug.

## Common mistakes

1. Assuming `>` captures standard error automatically.
2. Mixing diagnostics with data and then trying to parse the combined output.
3. Forgetting that an output redirection can overwrite an existing file.

***

## 4.9 Redirection

**Level: Beginner**

> ## The 20% that matters
>
> - `>` overwrites a file with standard output.
> - `>>` appends standard output.
> - `2>` captures standard error.
> - `2>&1` sends standard error to the current destination of standard output.
> - Redirection order matters.

## Redirection operators

| Operator | Meaning |
|---|---|
| `>` | Redirect stdout and overwrite the destination. |
| `>>` | Redirect stdout and append to the destination. |
| `<` | Read stdin from a file. |
| `2>` | Redirect stderr and overwrite the destination. |
| `2>>` | Redirect stderr and append to the destination. |
| `&>` | Redirect stdout and stderr in Bash. |
| `2>&1` | Make stderr point to the current stdout destination. |
| `|` | Send stdout to the next command’s stdin. |

Bash performs redirections before executing the command. File descriptor numbers identify streams. Bash’s manual specifically documents why `command > file 2>&1` differs from `command 2>&1 > file`. [gnu](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)

## Overwrite with `>`

```bash
echo "first line" > output.txt
cat output.txt
```

**Sample output:**

```text
first line
```

Run:

```bash
echo "replacement line" > output.txt
cat output.txt
```

**Sample output:**

```text
replacement line
```

The original line was overwritten.

## Append with `>>`

```bash
echo "first line" > output.txt
echo "second line" >> output.txt
cat output.txt
```

**Sample output:**

```text
first line
second line
```

`>>` preserves existing content and adds new output at the end.

## Read stdin with `<`

```bash
sort < output.txt
```

`sort` reads from standard input. The shell supplies `output.txt` as that input.

**Sample output:**

```text
first line
second line
```

Create an example where sorting changes order:

```bash
printf '%s\n' zebra apple mango > words.txt
sort < words.txt
```

**Sample output:**

```text
apple
mango
zebra
```

The source file remains unchanged.

## Capture errors separately

```bash
ls /nonexistent-directory 2> errors.txt
cat errors.txt
```

**Sample output:**

```text
ls: cannot access '/nonexistent-directory': No such file or directory
```

## Combine stdout and stderr

Bash supports:

```bash
ls /etc /nonexistent-directory &> all-output.txt
cat all-output.txt
```

**Sample output:**

```text
/etc:
adduser.conf
...
ls: cannot access '/nonexistent-directory': No such file or directory
```

`&>` is Bash syntax. Use `> file 2>&1` when you want a form commonly recognized across Bourne-style shells.

## Understand `2>&1`

Preferred form:

```bash
command > file 2>&1
```

Interpret it left to right:

1. `> file` sends stdout to `file`.
2. `2>&1` sends stderr to the same destination currently used by stdout.
3. Both streams go to `file`.

Example:

```bash
ls /etc /nonexistent-directory > all-output.txt 2>&1
cat all-output.txt
```

Both the directory listing and error message go into `all-output.txt`.

## Why order matters

This form:

```bash
command 2>&1 > file
```

is different:

1. `2>&1` first sends stderr to the terminal destination that stdout currently uses.
2. `> file` then sends only stdout to `file`.
3. Stderr still points to the terminal.

Bash documents this exact distinction. [gnu](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)

Demonstration:

```bash
ls /etc /nonexistent-directory 2>&1 > first-order.txt
```

Expected behavior:

- Normal `/etc` listing goes to `first-order.txt`.
- The error may still appear on the terminal.

Now use:

```bash
ls /etc /nonexistent-directory > second-order.txt 2>&1
```

Expected behavior:

- Both normal output and errors go to `second-order.txt`.

## Prevent accidental overwrite

Use `set -o noclobber` in the current shell:

```bash
set -o noclobber
```

Now:

```bash
echo "new data" > output.txt
```

may fail if `output.txt` already exists.

**Sample output:**

```text
bash: output.txt: cannot overwrite existing file
```

Append still works:

```bash
echo "additional data" >> output.txt
```

Turn the option off:

```bash
set +o noclobber
```

This setting affects the current shell only.

## Real-world use case

Capture a diagnostic command while preserving both data and errors:

```bash
REPORT="$HOME/cli-lab/diagnostic.txt"
df -hT > "$REPORT" 2>&1
cat "$REPORT"
```

The command is read-only. Only the report file is created or overwritten in the lab directory.

## Common mistakes

1. Using `>` when you meant `>>`, causing previous report data to be lost.
2. Writing `2>&1 > file` when you meant to capture both streams.
3. Redirecting output to a path you have not verified.

***

## 4.10 Pipes and Command Chaining

**Level: Intermediate**

> ## The 20% that matters
>
> - A pipe sends one command’s standard output to the next command’s standard input.
> - Build pipelines one stage at a time.
> - `&&` continues only after success.
> - `||` continues only after failure.
> - `;` continues regardless of status.

## Pipes

A pipe connects commands:

```text
command1 -> stdout -> stdin -> command2 -> stdout -> command3
```

Bash runs the commands in a pipeline concurrently and connects their streams. The output of each command becomes input to the next command. [gnu](https://www.gnu.org/software/bash/manual/html_node/Pipelines.html)

### Basic pipeline

```bash
ls -lah | less
```

- `ls -lah` produces directory output.
- `|` sends that output to `less`.
- `less` displays it one screen at a time.

Exit `less` with `q`.

### Filter a listing

```bash
ls -lah | grep txt
```

`grep` receives the listing through standard input and prints lines containing `txt`.

**Sample output:**

```text
-rw-r--r-- 1 student student 18 Oct 16 17:55 notes.txt
-rw-r--r-- 1 student student  0 Oct 16 17:55 report1.txt
```

### Count matching lines

```bash
ls -lah | grep txt | wc -l
```

Read the pipeline from left to right:

1. `ls -lah` lists entries.
2. `grep txt` keeps lines containing `txt`.
3. `wc -l` counts those lines.

**Sample output:**

```text
2
```

For robust filename counting, prefer `find` with NUL-delimited output rather than parsing formatted `ls` output:

```bash
find ~/cli-lab -maxdepth 1 -type f -name '*.txt' -print0 | tr -cd '\0' | wc -c
```

That example counts NUL separators. It is more complex because filenames can contain spaces and unusual characters.

## Pipeline errors

By default, the exit status of a pipeline is the status of its last command. Bash can enable `pipefail` so the pipeline fails if any component fails. [gnu](https://www.gnu.org/software/bash/manual/html_node/Pipelines.html)

Example:

```bash
false | true
echo "$?"
```

**Sample output:**

```text
0
```

The final command, `true`, succeeded.

Enable `pipefail`:

```bash
set -o pipefail
false | true
echo "$?"
```

**Sample output:**

```text
1
```

Now the failed `false` command causes the pipeline to return a non-zero status.

Disable it if needed:

```bash
set +o pipefail
```

Later, robust scripts commonly use:

```bash
set -euo pipefail
```

We will cover this in Part 4.

## Command chaining operators

| Operator | Meaning |
|---|---|
| `;` | Run the next command regardless of the previous status. |
| `&&` | Run the next command only if the previous command succeeds. |
| `||` | Run the next command only if the previous command fails. |
| `&` | Run the command in the background. |
| Newline | Usually separates sequential commands. |

### Sequential execution with `;`

```bash
mkdir demo ; cd demo ; pwd
```

This runs all three commands in sequence even if an earlier command fails.

If `mkdir demo` fails because the directory already exists and `-p` was not used, Bash still attempts `cd demo`.

### Conditional execution with `&&`

```bash
mkdir -p demo2 && cd demo2 && pwd
```

Each next command runs only if the previous command succeeds.

This is useful for safe multi-step operations:

```bash
mkdir -p "$HOME/cli-lab/report" &&
printf 'created at %s\n' "$(date)" > "$HOME/cli-lab/report/status.txt" &&
cat "$HOME/cli-lab/report/status.txt"
```

### Failure handling with `||`

```bash
cd /nonexistent || echo "Directory does not exist"
```

**Sample output:**

```text
bash: cd: /nonexistent: No such file or directory
Directory does not exist
```

`echo` runs because `cd` failed.

### Background execution with `&`

```bash
sleep 5 &
```

**Sample output:**

```text
 [gnu](https://www.gnu.org/software/bash/manual/bash.html) 24871
```

- ` [gnu](https://www.gnu.org/software/bash/manual/bash.html)` is the shell job number.
- `24871` is the process ID in this sample.

Inspect jobs:

```bash
jobs
```

**Sample output:**

```text
 [gnu](https://www.gnu.org/software/bash/manual/bash.html)+  Running                 sleep 5 &
```

After five seconds:

```bash
jobs
```

The job may no longer appear because it finished.

Use `wait` to wait for a specific job:

```bash
sleep 2 &
PID=$!
wait "$PID"
echo "$?"
```

`$!` contains the process ID of the most recent background command.

## Operator precedence

Bash treats `&&` and `||` with equal precedence. They are evaluated left to right. `;` and `&` separate commands at a lower practical level.

Use braces or parentheses when grouping matters:

```bash
{
    echo "first"
    echo "second"
} > grouped-output.txt
```

The braces group commands in the current shell. The final command before `}` needs a semicolon or newline.

Use a subshell with parentheses:

```bash
(
    cd /tmp
    pwd
)
pwd
```

The `cd` affects only the subshell. The final `pwd` returns to the original directory.

## Real-world use case

A service deployment should stop if a required directory cannot be created:

```bash
mkdir -p "$HOME/cli-lab/deployment" &&
cp "$HOME/cli-lab/config.txt" "$HOME/cli-lab/deployment/" &&
echo "Deployment files copied"
```

If `mkdir` or `cp` fails, the success message does not print.

## Common mistakes

1. Building a long pipeline without testing each stage separately.
2. Assuming the pipeline status reports every failed command without `pipefail`.
3. Using `;` where `&&` is required for safe conditional execution.

***

## 4.11 Exit Codes

**Level: Intermediate**

> ## The 20% that matters
>
> - Every command returns an exit status.
> - `0` usually means success.
> - A non-zero value usually means failure or a special condition.
> - `$?` contains the status of the most recent command.
> - Check status immediately because another command changes `$?`.

## Concept

An **exit code**, also called an exit status, is a small integer returned by a command when it finishes.

General convention:

```text
0       success
nonzero failure or another condition
```

The exact meaning of non-zero values depends on the command. Read the manual page when the difference matters.

## Demonstration

### Successful command

```bash
true
echo $?
```

**Sample output:**

```text
0
```

### Failed command

```bash
false
echo $?
```

**Sample output:**

```text
1
```

### Missing path

```bash
ls /nonexistent
echo $?
```

**Sample output:**

```text
ls: cannot access '/nonexistent': No such file or directory
2
```

The exact `ls` status can depend on the failure and implementation. The important point is that it is non-zero.

### `grep` status

Create a small file:

```bash
printf '%s\n' alpha beta gamma > output.txt
```

Search for a match:

```bash
grep "beta" output.txt
echo $?
```

**Sample output:**

```text
beta
0
```

Search for a non-matching pattern:

```bash
grep "delta" output.txt
echo $?
```

**Sample output:**

```text
1
```

For GNU `grep`, status `1` normally means no lines matched. A status of `2` or another non-zero value generally indicates an error, such as an invalid pattern or unreadable file. Verify with `man grep` on your system.

## Check status immediately

This is unreliable:

```bash
false
echo "status was $?"
pwd
echo "status is now $?"
```

The `pwd` command replaces the value available as `$?`.

Use:

```bash
false
STATUS=$?
echo "status was $STATUS"
```

**Sample output:**

```text
status was 1
```

## Use status with `&&`

```bash
mkdir -p "$HOME/cli-lab/exit-demo" &&
echo "directory creation succeeded"
```

The message runs only if `mkdir` returns zero.

## Use status with `||`

```bash
cd "$HOME/cli-lab/missing-directory" ||
echo "could not change directory"
```

**Sample output:**

```text
bash: cd: /home/student/cli-lab/missing-directory: No such file or directory
could not change directory
```

## Exit status of pipelines

By default:

```bash
false | true
echo "$?"
```

**Sample output:**

```text
0
```

With `pipefail`:

```bash
set -o pipefail
false | true
echo "$?"
set +o pipefail
```

**Sample output:**

```text
1
```

## Exit-code reference

| Status | General meaning | Important qualification |
|---:|---|---|
| `0` | Success | Usually means the requested operation completed. |
| `1` | General failure or false condition | `grep 1` often means no match, not a processing error. |
| `2` | Misuse, invalid argument, or command-specific error | Exact meaning depends on the command. |
| `126` | Found but cannot execute | Permissions or executable format may be involved. |
| `127` | Command not found | Bash could not resolve the command. |
| `128` | Invalid exit argument or command-specific status | Check the command documentation. |
| `128+n` | Often terminated by signal `n` | This is a common shell convention, not a universal semantic guarantee. |

## Real-world use case

A health check must signal failure to an automation system:

```bash
if df -P "$HOME" | awk 'NR == 2 { gsub("%", "", $5); exit ($5 >= 90) }'; then
    echo "Disk usage is below threshold"
else
    echo "Disk usage is at or above threshold" >&2
    exit 1
fi
```

This example will be explained more fully in Part 4. It returns non-zero when the threshold is exceeded.

## Common mistakes

1. Checking `$?` after running another command.
2. Treating every non-zero status as the same type of failure.
3. Forgetting that a pipeline may hide an earlier failure unless `pipefail` is enabled.

***

# Text stream diagrams

## Standard streams

```text
Keyboard -> stdin -> command -> stdout -> terminal or file
                          -> stderr -> terminal or file
```

## Pipeline

```text
command1 -> stdout -> stdin -> command2 -> stdout -> command3
```

Example:

```bash
grep "ERROR" web.log | cut -d' ' -f1 | sort | uniq -c
```

Interpret it from left to right:

1. `grep` selects error records.
2. `cut` extracts one field.
3. `sort` groups equal values together.
4. `uniq -c` counts adjacent equal values.

## Bash execution flow

```text
Bash reads command
  -> expands variables and globs
  -> resolves command using PATH
  -> executes process
  -> returns exit code
```

***

# Part 2 Core Exercises

## Core Exercise 1: Quoted filenames

**Goal:** Create, inspect, and remove a filename containing spaces.

**Commands:**

```bash
cd ~/cli-lab
touch "server report.txt"
ls -lah "server report.txt"
file "server report.txt"
```

Before removal:

```bash
pwd
ls -lah
```

Then:

```bash
rm -i "server report.txt"
```

**Expected result:** The file is created and removed only after confirmation.

**Undo step:** If you remove it accidentally, recreate it with:

```bash
touch "server report.txt"
```

***

## Core Exercise 2: Wildcard inspection

**Goal:** Use glob patterns without deleting files.

**Commands:**

```bash
cd ~/cli-lab
touch check1.txt check2.txt check3.log
ls -lah
ls -lah *.txt
ls -lah check?.txt
```

**Expected result:** `*.txt` selects the three `.txt` files. `check?.txt` selects files with one character after `check`.

**Undo step:** No files need to be removed for this exercise.

***

## Core Exercise 3: Separate standard output and errors

**Goal:** Save normal output and errors in different files.

**Commands:**

```bash
cd ~/cli-lab
ls -lah > directory-listing.txt
ls /path-that-does-not-exist 2> command-errors.txt
cat directory-listing.txt
cat command-errors.txt
```

**Expected result:**

- `directory-listing.txt` contains the listing.
- `command-errors.txt` contains the error message.

**Undo step:**

```bash
pwd
ls -lah
rm -i directory-listing.txt command-errors.txt
```

***

## Core Exercise 4: Append and read redirected input

**Goal:** Use `>`, `>>`, and `<`.

**Commands:**

```bash
cd ~/cli-lab
printf '%s\n' zebra > words.txt
printf '%s\n' apple >> words.txt
printf '%s\n' mango >> words.txt

cat words.txt
sort < words.txt
```

**Expected result:**

```text
zebra
apple
mango
```

Then sorted output:

```text
apple
mango
zebra
```

**Undo step:**

```bash
pwd
ls -lah
rm -i words.txt
```

***

## Core Exercise 5: Build a pipeline

**Goal:** Count text files in the lab directory.

**Commands:**

```bash
cd ~/cli-lab
find . -maxdepth 1 -type f -name '*.txt' -print
find . -maxdepth 1 -type f -name '*.txt' -print | wc -l
```

**Expected result:** The first command lists matching files. The second prints their count.

**Undo step:** None. These commands are read-only.

***

## Core Exercise 6: Conditional commands

**Goal:** Use `&&` and `||` to control execution.

**Commands:**

```bash
cd ~/cli-lab
mkdir -p condition-demo &&
printf 'created successfully\n' > condition-demo/status.txt &&
cat condition-demo/status.txt
```

Now test failure handling:

```bash
cd "$HOME/cli-lab/missing-directory" ||
echo "Directory change failed"
```

**Expected result:** The first sequence creates and reads a status file. The second prints the failure message.

**Undo step:**

```bash
pwd
ls -lah
rm -ri "$HOME/cli-lab/condition-demo"
```

***

## Core Exercise 7: Check exit codes

**Goal:** Observe success, failure, and no-match statuses.

**Commands:**

```bash
true
echo "$?"

false
echo "$?"

printf '%s\n' alpha beta > status-input.txt
grep beta status-input.txt
echo "$?"

grep gamma status-input.txt
echo "$?"
```

**Expected result:**

```text
0
1
beta
0
1
```

The exact behavior of `grep` should be verified with `man grep`.

**Undo step:**

```bash
pwd
ls -lah
rm -i status-input.txt
```

***

# Part 2 Extension Exercises

## Extension Exercise 1: Capture a complete diagnostic

**Goal:** Save standard output and standard error together.

**Commands:**

```bash
cd ~/cli-lab
{
    echo "Directory listing:"
    ls -lah
    echo
    echo "Intentional diagnostic:"
    ls /path-that-does-not-exist
} > combined-report.txt 2>&1

cat combined-report.txt
```

**What additional skill it develops:** Grouping commands, redirection order, and producing a reproducible report.

***

## Extension Exercise 2: Compare pipeline status with and without `pipefail`

**Goal:** Observe why robust scripts use `pipefail`.

**Commands:**

```bash
set +o pipefail
false | true
echo "Without pipefail: $?"

set -o pipefail
false | true
echo "With pipefail: $?"

set +o pipefail
```

**What additional skill it develops:** Pipeline reliability and failure propagation.

***

## Extension Exercise 3: Use NUL-safe filename processing

**Goal:** Process filenames safely when spaces are present.

**Commands:**

```bash
cd ~/cli-lab
mkdir -p filename-demo
touch "filename-demo/one file.txt" "filename-demo/two file.txt"

find filename-demo -type f -print0 |
    xargs -0 -n 1 printf 'Found: <%s>\n'
```

**Sample output:**

```text
Found: <filename-demo/one file.txt>
Found: <filename-demo/two file.txt>
```

**What additional skill it develops:** Safe handling of spaces and other special characters in filenames.

Clean up after verification:

```bash
pwd
ls -lah
rm -ri filename-demo
```

***

## Part 2 command summary

| Task | Command pattern |
|---|---|
| Show current path | `pwd` |
| List including hidden files | `ls -lah` |
| Quote a variable | `"$VARIABLE"` |
| Match text files | `*.txt` |
| Save stdout | `command > file` |
| Append stdout | `command >> file` |
| Save stderr | `command 2> file` |
| Save both streams | `command > file 2>&1` |
| Pipe output | `command1 \| command2` |
| Continue after success | `command1 && command2` |
| Continue after failure | `command1 \|\| command2` |
| Run sequentially regardless of status | `command1 ; command2` |
| Run in background | `command &` |
| Read last exit code | `echo "$?"` |
| Enable pipeline failure reporting | `set -o pipefail` |

Part 2 establishes the stream and shell-control concepts needed for log analysis, operational reports, safe scripts, and defensive investigation. Bash’s official documentation defines redirections, pipelines, lists, expansion behavior, and exit-status rules used throughout this chapter. [gnu](https://www.gnu.org/software/bash/manual/html_node/Lists.html)
