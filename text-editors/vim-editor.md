# Basics of Vim Editor for Server Administration

Recommendation: Learn Vim's three main modes, basic file operations, and search commands first. As a Linux System Administrator or Server Engineer, you will use these skills to edit configuration files, troubleshoot services, modify scripts, and work on remote servers through SSH.

Vim (Vi Improved) is a terminal-based text editor available on many Linux and Unix systems. It is particularly useful when administering headless servers without a graphical desktop.

## 1. What is Vim?

[From Confused to Confident: How Vim Made Me Better in Cloud & DevOps - DEV Community](https://images.openai.com/static-rsc-4/4WlLXGj3jM9OrKLSD_6DN1sGTuXcYgOOIZmsdGHwD6_j4GUPghGBQyQlzDUctjXx35Y_nCnl_AY41gVHye0Xj3UR9JT-F4fdP_f7FQlbMTtMemPFrY3ojIkMCea0eT_YPlEIifQtCaj8_7-4PjvveiYz9ek3ohLvIInrYWwI91M?purpose=inline)

[dev.to](https://dev.to/dtemi/from-confused-to-confident-how-vim-made-me-better-in-cloud-devops-17jk)

Vim is an enhanced version of the traditional `vi` editor. Unlike Nano, Vim uses different operating modes for navigation, text insertion, and editing commands.

Common server administration tasks include:

- Editing SSH and network configuration files.
- Modifying Nginx or Apache configuration.
- Editing Linux service and system configuration files.
- Writing Bash scripts and automation scripts.
- Reviewing logs and configuration files.
- Editing files on remote Linux servers over SSH.

Check whether Vim is installed:

```
vim --version
```

Open a file:

```
vim notes.txt
```

If the file exists, Vim opens it. Otherwise, you can create it by editing and saving.

## 2. Understand Vim's three main modes

This is the most important concept to understand before using Vim.

<img width="535" height="415" alt="image" src="https://github.com/user-attachments/assets/8066e2ee-bb27-4117-98fc-e270601e181b" />


Remember: Vim starts in Normal Mode. If you want to type text, press `i`. When you finish typing, press `Esc` before executing commands such as saving or quitting.

## 3. Open, create, save, and exit files

Start by practicing these commands in a terminal.

| Command          | Purpose                                         |
| ---------------- | ----------------------------------------------- |
| `vim file.txt`   | Open or create a file                           |
| `:w`             | Save changes                                    |
| `:q`             | Quit Vim if there are no unsaved changes        |
| `:wq`            | Save and quit                                   |
| `:q!`            | Quit without saving changes                     |
| `:w newfile.txt` | Save the current content under another filename |

These commands are entered after pressing `Esc` and then typing the command beginning with `:`. Press `Enter` to execute it.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://vimhelp.org\&sz=32)

vimhelp.org

+1



### Practical example

```
vim server-notes.txt
```

1. Press `i` to enter Insert Mode.
2. Type:

```
Linux Server Administration
Hostname: server01
Role: Web Server
Status: Active
```

3. Press `Esc` to return to Normal Mode.
4. Type `:wq` and press `Enter`.

Verify the saved file:

```
cat server-notes.txt
```

### What if you accidentally enter Vim and cannot exit?

Press `Esc` a couple of times, then type:

```
:q!
```

Press `Enter`. This exits without saving changes made in the current editing session. Be careful: it discards unsaved edits.

## 4. Basic navigation commands

In Normal Mode, you can navigate without using the mouse.

| Key        | Action                                     |
| ---------- | ------------------------------------------ |
| `h`        | Move left                                  |
| `j`        | Move down                                  |
| `k`        | Move up                                    |
| `l`        | Move right                                 |
| `w`        | Move to the beginning of the next word     |
| `b`        | Move to the beginning of the previous word |
| `0`        | Move to the beginning of the line          |
| `$`        | Move to the end of the line                |
| `gg`       | Go to the first line                       |
| `G`        | Go to the last line                        |
| `:25`      | Go to line 25                              |
| `Ctrl + f` | Move forward one screen                    |
| `Ctrl + b` | Move backward one screen                   |

For example, when editing a long configuration file, `:150` takes you directly to line 150.

## 5. Insert and modify text

Vim provides several commands for entering text.

| Key   | Action                              |
| ----- | ----------------------------------- |
| `i`   | Insert before the cursor            |
| `a`   | Append after the cursor             |
| `I`   | Insert at the beginning of the line |
| `A`   | Append at the end of the line       |
| `o`   | Open a new line below               |
| `O`   | Open a new line above               |
| `Esc` | Return to Normal Mode               |

Example: To add a new configuration line, move the cursor to the desired location, press `o`, type the line, and press `Esc`.

## 6. Delete, copy, paste, and undo

These commands are used frequently when modifying scripts and configuration files.

| Command    | Action                                            |
| ---------- | ------------------------------------------------- |
| `x`        | Delete the character under the cursor             |
| `dd`       | Delete the current line                           |
| `dw`       | Delete from the cursor to the next word boundary  |
| `yy`       | Copy (yank) the current line                      |
| `p`        | Paste after the cursor or below the current line  |
| `P`        | Paste before the cursor or above the current line |
| `u`        | Undo the last change                              |
| `Ctrl + r` | Redo an undone change                             |
| `D`        | Delete from the cursor to the end of the line     |

Important: Vim's `yy` copies text into Vim's internal register; it does not necessarily copy text to your operating system's clipboard.

## 7. Search and replace text

Searching is particularly useful when you need to locate a setting in a long server configuration file.

### Search for a word

In Normal Mode, type:

```
/PermitRootLogin
```

Press `Enter` to find the next match.

- `n` — Next match.
- `N` — Previous match.
- `?PermitRootLogin` — Search backward.

### Replace text

Replace the first occurrence on the current line:

```
:s/old/new/
```

Replace all occurrences on the current line:

```
:s/old/new/g
```

Replace all occurrences throughout the file, with confirmation for each match:

```
:%s/old/new/gc
```

The `c` flag lets you review each proposed replacement before accepting it. This is particularly useful for configuration changes.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://vimhelp.org\&sz=32)

vimhelp.org

+1



## 8. Real-world server administration examples

These examples demonstrate how Vim fits into routine Linux administration.

[Security Hardening Ubuntu 20.04 | IT Blog](https://images.openai.com/static-rsc-4/Rcleh7_l0cbH4YnYAf0x7DpD23-JRIveu0lIgsm4L1gl96AakApJf7DH672GoOjeAwjvomFzOU7LIyCMhPvbLpnkhwRJVDR6_4Hw8_pfxVNltfS-1mT-jsx0b02wvIIoepDkB_rJ4x3vGaf7GvNfY9ytVfzPPvFvDd2OyV0Cgbc?purpose=inline)

Example 1: Edit SSH server configuration

```
sudo vim /etc/ssh/sshd_config
```

Use `/PermitRootLogin` to find the setting. Review the effective configuration and your organization's access policy before changing it.

After saving, validate the file before reloading SSH:

```
sudo sshd -t
```

Only reload the service after validation succeeds and you have confirmed the correct service name for your distribution. Keep your existing SSH session open while testing a new connection.

[Mac 환경에서 NGINX 설치하고 도메인 붙이기](https://images.openai.com/static-rsc-4/02CMr2sVax97DOrce7vUJmJ-1x3fCmA7oa3RcpOLBbdzqF4ruSsTiw65lw46pTipPLgibtYZhXJN2zSMjhYbf-Q8ayHBSiRpRniHi4na-pA7EJNZIN-KoJ8xPzz1toKk2BdbZFDaK2aHVSAZk8zLL8KPI0_g_Mc6_89LE46jO7I?purpose=inline)

Example 2: Edit Nginx configuration

```
sudo vim /etc/nginx/nginx.conf
```

Find directives using `/worker_processes` or another relevant setting.

Before reloading Nginx, validate the configuration:

```
sudo nginx -t
```

If the test succeeds, reload the service using the appropriate system service command for your environment.

[Using Tabbed Interface in Vim](https://images.openai.com/static-rsc-4/DgSYWd-f3RQk2bm32mNckL4JiG5mCJQJ4jExVkMDop7H5ysgJGq4I_qZW96mxAwhXXDnhQSMr-7FQjT1R4YuZ7zkq6hzbjzn56DyQrnTGqKGJFEw1vSgyjqrw4OLRM6OAq9SWsC7QNSAla7BU7FijPyWyUJAhu4BXzFISkl5qEo?purpose=inline)

Example 3: Create a Bash script

```
vim check-disk.sh
```

Enter Insert Mode and add:

```
#!/bin/bash
df -h
```

Save and exit. Test it with:

```
bash check-disk.sh
```

This displays filesystem usage in a human-readable format.

## 9. Working with Vim over SSH

Vim is especially useful on remote servers because you can edit files directly without transferring them to your local computer.

A typical workflow is:

```
ssh admin@192.168.1.10
```

Once connected:

```
vim /path/to/configuration-file
```

Make the required change, save it, validate the configuration, and test the service.

For important production changes, make a backup first, for example:

```
sudo cp /etc/hosts /etc/hosts.bak
sudo vim /etc/hosts
```

This backup command is suitable for `/etc/hosts`; use an appropriate backup procedure and permissions for other files. Avoid blindly overwriting configuration files or making changes without understanding their effects.

## 10. Vim versus Nano

| Feature                           | Vim                        | Nano                |
| --------------------------------- | -------------------------- | ------------------- |
| Editing modes                     | Multiple modes             | Direct text editing |
| Learning curve                    | Steeper                    | Easier              |
| Navigation                        | Powerful keyboard commands | Keyboard shortcuts  |
| Bulk editing                      | Extensive                  | More limited        |
| Remote server use                 | Excellent                  | Excellent           |
| Automation and advanced workflows | Strong                     | Basic               |

Nano is a good starting point for simple edits. Vim is worth learning for more advanced navigation, bulk changes, and efficient terminal workflows.

## 11. Practice exercises

## Hands-on Vim checklist

0 of 6

Create a file

Enter Insert Mode and write five lines about Linux.

vim practice.txt

Save and exit

Return to Normal Mode first, then save and quit.

:wq

Navigate

Move to the first line, last line, and line 3.

gg  →  G  →  :3

Copy and paste

Copy a line and paste it below.

yy  →  p

Search and replace

Try replacing a word in your practice file and confirm the changes.

:%s/Linux/Ubuntu/gc

Undo and redo

Undo a change, then redo it.

u  →  Ctrl + r

## 12. Learn Vim interactively

Most Vim installations include a guided tutorial. Run:

```
vimtutor
```

It introduces editing, navigation, deletion, and other essential commands through hands-on exercises. If the command is unavailable, check whether the Vim tutorial package is installed on your Linux distribution. The official Vim documentation also provides a reference manual.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://vimhelp.org\&sz=32)

vimhelp.org

+1



Recommended learning order: Modes → navigation → save and exit → editing commands → search and replace → server configuration → safe production changes.

The most important habit is to return to Normal Mode with `Esc` before executing Vim commands. Once that becomes natural, editing files on Linux servers becomes much easier.
