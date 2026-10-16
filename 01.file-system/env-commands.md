# Basic `env` Commands in Linux

The `env` command in Linux is used to display environment variables, run commands with modified environment variables, and set or remove environment variables for a specific command.

Environment variables store information that programs and shells use, such as the username, home directory, shell, and command search paths.

## 1. Basic `env` commands

| Command                 | Purpose                                              |
| ----------------------- | ---------------------------------------------------- |
| `env`                   | Display environment variables                        |
| `env \\| sort`          | Display variables in alphabetical order              |
| `env \\| grep HOME`     | Display variables containing `HOME`                  |
| `printenv`              | Display environment variables                        |
| `printenv HOME`         | Display the value of `HOME`                          |
| `echo "$USER"`          | Display the current username                         |
| `echo "$HOME"`          | Display the user's home directory                    |
| `echo "$SHELL"`         | Display the user's configured login shell            |
| `echo "$PATH"`          | Display directories searched for executable commands |
| `env -0`                | Display variables separated by null characters       |
| `env -i command`        | Run a command with an almost empty environment       |
| `env VAR=value command` | Run a command with a temporary environment variable  |

## 2. Display environment variables

### Command 1: Display all environment variables

```
env
```

Purpose: Displays environment variables available to the current command environment.

Illustrative output:

```
SHELL=/bin/bash
PWD=/home/hytrel
LOGNAME=hytrel
HOME=/home/hytrel
LANG=en_US.UTF-8
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
USER=hytrel
```

Your actual output will vary depending on your Linux system and session.

### Command 2: Display variables alphabetically

```
env | sort
```

- `env` displays the variables.
- `|` sends the output to another command.
- `sort` arranges the lines alphabetically.

### Command 3: Display a specific variable

```
printenv HOME
```

Illustrative output:

```
/home/hytrel
```

You can also use:

```
echo "$HOME"
```

Both display the value of the `HOME` variable.

### Command 4: Search for a variable

```
env | grep PATH
```

Purpose: Displays environment-variable lines containing `PATH`.

To search for several common variables:

```
env | grep -E '^(HOME|USER|SHELL|PATH|LANG)='
```

## 3. Understand important environment variables

| Variable | Purpose                           | Example          |
| -------- | --------------------------------- | ---------------- |
| `HOME`   | User's home directory             | `/home/hytrel`   |
| `USER`   | Username in the environment       | `hytrel`         |
| `SHELL`  | Configured login shell path       | `/bin/bash`      |
| `PATH`   | Directories searched for commands | `/usr/bin:/bin`  |
| `PWD`    | Current working directory         | `/home/hytrel`   |
| `LANG`   | Default locale setting            | `en_US.UTF-8`    |
| `TERM`   | Terminal type                     | `xterm-256color` |

Inspect them with:

```
printenv HOME
printenv USER
printenv SHELL
printenv PATH
printenv PWD
printenv LANG
printenv TERM
```

Note: `SHELL` usually indicates the configured login shell, not necessarily the shell currently executing your commands.

## 4. Run a command with a temporary variable

Use this syntax:

```
env VARIABLE=value command
```

Example:

```
env APP_MODE=testing printenv APP_MODE
```

Illustrative output:

```
testing
```

Here, `APP_MODE=testing` is available to `printenv` for that command. It does not permanently set the variable in your current shell.

Verify that it was not retained:

```
printenv APP_MODE
```

If `APP_MODE` was not previously defined, this normally prints nothing and returns a nonzero exit status.

Another practical example:

```
env EDITOR=nano printenv EDITOR
```

Output:

```
nano
```

## 5. Run a command with an almost empty environment

```
env -i printenv
```

Purpose: Runs `printenv` without inheriting the usual environment variables. The output is normally empty.

You can provide selected variables explicitly:

```
env -i HOME="$HOME" PATH=/usr/bin:/bin printenv
```

This is useful when testing scripts or investigating whether a program depends on environment variables. Some programs may require additional variables to work correctly.

## 6. Difference between `env`, `printenv`, and `export`

| Command             | What it does                                                              |
| ------------------- | ------------------------------------------------------------------------- |
| `env`               | Displays environment variables or runs a command with environment changes |
| `printenv`          | Displays environment-variable values                                      |
| `export NAME=value` | Sets a shell variable and exports it to child processes                   |
| `unset NAME`        | Removes a shell variable from the current shell environment               |

Example:

```
export LAB_NAME=linux-basics
printenv LAB_NAME
```

Output:

```
linux-basics
```

Remove it:

```
unset LAB_NAME
```

Check again:

```
printenv LAB_NAME
```

If the variable is absent, no value is printed.

## 7. Practical beginner lab

Run these commands one by one on your Ubuntu Server:

```
# 1. Display environment variables
env

# 2. Display the home directory
printenv HOME

# 3. Display the username
printenv USER

# 4. Display the command search path
printenv PATH

# 5. Find variables related to the shell
env | grep -E '^(SHELL|TERM)='

# 6. Create a temporary environment variable for one command
env LAB_STATUS=learning printenv LAB_STATUS

# 7. Set a variable for this shell and its child processes
export LAB_STATUS=learning

# 8. Display the exported variable
printenv LAB_STATUS

# 9. Remove the variable
unset LAB_STATUS
```

Recommended learning order: Start with `env`, `printenv`, `echo "$VARIABLE"`, and `export`. Then practise `env VARIABLE=value command` and `env -i` to understand how environment variables affect scripts and server applications.
