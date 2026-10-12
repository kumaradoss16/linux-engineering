# Basics of Vim Editor for Server Administration

**Recommendation:** Learn Vim's three main modes, basic file operations, navigation, and search commands first. As a Linux System Administrator or Server Engineer, you will use these skills to edit configuration files, troubleshoot services, modify scripts, and manage remote servers through SSH.

Vim (Vi Improved) is a terminal-based text editor available on many Linux and Unix systems. It is particularly useful when administering headless servers without a graphical desktop.

## 1. What Is Vim?

Vim is an enhanced version of the traditional `vi` editor. Unlike Nano, Vim uses different operating modes for navigation, text insertion, and editing commands.

Common server administration tasks include:

- Editing SSH and network configuration files.
- Modifying Nginx or Apache configuration.
- Editing Linux service and system configuration files.
- Writing Bash scripts and automation scripts.
- Reviewing logs and configuration files.
- Editing files on remote Linux servers over SSH.

### Check whether Vim is installed

```bash
vim --version
```

### Open or create a file

```bash
vim notes.txt
```

If the file exists, Vim opens it. Otherwise, you can create it by entering text and saving the file.

## 2. Understand Vim's Three Main Modes

Understanding Vim's modes is the most important step before learning advanced commands.

### 2.1 Normal Mode

Normal Mode is used to navigate through text and execute editing commands.

Examples:

- `h`, `j`, `k`, `l` — Navigate through text.
- `dd` — Delete a line.
- `yy` — Copy a line.
- `p` — Paste text.

Vim starts in Normal Mode. Press `i` to enter Insert Mode.

### 2.2 Insert Mode

Insert Mode allows you to type, add, and modify text.

Enter Insert Mode using:

- `i` — Insert before the cursor.
- `a` — Append after the cursor.
- `o` — Open a new line below the current line.

Press `Esc` to return to Normal Mode.

### 2.3 Command-Line Mode

Command-Line Mode allows you to execute file operations, searches, replacements, and other commands.

Examples:

```vim
:w
:q
:wq
```

To enter Command-Line Mode, press `Esc`, type a command beginning with `:`, and press `Enter`.

**Remember:** Vim starts in Normal Mode. Press `i` to type text. When you finish editing, press `Esc` before saving or quitting.

## 3. Open, Create, Save, and Exit Files

Practice these commands in a terminal.

| Command | Purpose |
|---|---|
| `vim file.txt` | Open or create a file |
| `:w` | Save changes |
| `:q` | Quit if there are no unsaved changes |
| `:wq` | Save changes and quit |
| `:q!` | Quit without saving unsaved changes |
| `:w newfile.txt` | Save the current content under another filename |

Enter these commands after pressing `Esc`. Press `Enter` to execute them.

### Practical example

Open a file:

```bash
vim server-notes.txt
```

1. Press `i` to enter Insert Mode.
2. Type the following content:

```text
Linux Server Administration
Hostname: server01
Role: Web Server
Status: Active
```

3. Press `Esc` to return to Normal Mode.
4. Type `:wq` and press `Enter`.

Verify the saved file:

```bash
cat server-notes.txt
```

### What if you accidentally enter Vim and cannot exit?

Press `Esc` a couple of times, then type:

```vim
:q!
```

Press `Enter` to exit without saving unsaved changes. Be careful: this discards your unsaved edits.

## 4. Basic Navigation Commands

In Normal Mode, you can navigate through a file without using the mouse.

| Key | Action |
|---|---|
| `h` | Move left |
| `j` | Move down |
| `k` | Move up |
| `l` | Move right |
| `w` | Move to the beginning of the next word |
| `b` | Move to the beginning of the previous word |
| `0` | Move to the beginning of the line |
| `$` | Move to the end of the line |
| `gg` | Go to the first line |
| `G` | Go to the last line |
| `:25` | Go to line 25 |
| `Ctrl + f` | Move forward one screen |
| `Ctrl + b` | Move backward one screen |

For example, when editing a long configuration file, `:150` takes you directly to line 150.

## 5. Insert and Modify Text

Vim provides several commands for entering and modifying text.

| Key | Action |
|---|---|
| `i` | Insert before the cursor |
| `a` | Append after the cursor |
| `I` | Insert at the beginning of the line |
| `A` | Append at the end of the line |
| `o` | Open a new line below |
| `O` | Open a new line above |
| `Esc` | Return to Normal Mode |

**Example:** To add a configuration line, move the cursor to the desired location, press `o`, type the new line, and press `Esc`.

## 6. Delete, Copy, Paste, and Undo

These commands are frequently used when modifying scripts and configuration files.

| Command | Action |
|---|---|
| `x` | Delete the character under the cursor |
| `dd` | Delete the current line |
| `dw` | Delete from the cursor to the next word boundary |
| `yy` | Copy (yank) the current line |
| `p` | Paste after the cursor or below the current line |
| `P` | Paste before the cursor or above the current line |
| `u` | Undo the last change |
| `Ctrl + r` | Redo an undone change |
| `D` | Delete from the cursor to the end of the line |

**Important:** Vim's `yy` command copies text into Vim's internal register. It does not necessarily copy text to your operating system's clipboard.

## 7. Search and Replace Text

Searching is particularly useful when locating a setting in a long server configuration file.

### Search for a word

In Normal Mode, type:

```vim
/PermitRootLogin
```

Press `Enter` to search forward.

| Command | Purpose |
|---|---|
| `/word` | Search forward for a word or pattern |
| `?word` | Search backward |
| `n` | Move to the next match |
| `N` | Move to the previous match |

### Replace text

Replace the first occurrence on the current line:

```vim
:s/old/new/
```

Replace all occurrences on the current line:

```vim
:s/old/new/g
```

Replace occurrences throughout the file, confirming each replacement:

```vim
:%s/old/new/gc
```

The `c` flag allows you to review each proposed replacement before accepting it. This is particularly useful when modifying configuration files.

## 8. Real-World Server Administration Examples

These examples demonstrate how Vim fits into routine Linux administration.

### Example 1: Edit SSH Server Configuration

Open the SSH server configuration file:

```bash
sudo vim /etc/ssh/sshd_config
```

Find the relevant setting:

```vim
/PermitRootLogin
```

Review the configuration and your organization's access policy before making changes.

Validate the configuration before reloading the SSH service:

```bash
sudo sshd -t
```

Only proceed if validation succeeds. Confirm the correct service name for your Linux distribution before reloading it.

**Production safety:** Keep your existing SSH session open while testing a new connection. An incorrect SSH configuration can prevent new remote connections.

### Example 2: Edit Nginx Configuration

Open the Nginx configuration file:

```bash
sudo vim /etc/nginx/nginx.conf
```

Search for a directive:

```vim
/worker_processes
```

After making the required changes, validate the configuration:

```bash
sudo nginx -t
```

If validation succeeds, reload Nginx using the appropriate service command for your environment.

### Example 3: Create a Bash Script

Create a script:

```bash
vim check-disk.sh
```

Press `i` and enter:

```bash
#!/bin/bash

df -h
```

Press `Esc`, then save and exit:

```vim
:wq
```

Run the script:

```bash
bash check-disk.sh
```

The `df -h` command displays filesystem usage in a human-readable format.

## 9. Working with Vim over SSH

Vim is especially useful on remote servers because you can edit files directly without transferring them to your local computer.

Connect to a remote Linux server:

```bash
ssh admin@192.168.1.10
```

After connecting, open the required configuration file:

```bash
vim /path/to/configuration-file
```

Make the required changes, save the file, validate the configuration, and test the service.

### Back up a configuration file before editing

For example, create a backup of `/etc/hosts`:

```bash
sudo cp /etc/hosts /etc/hosts.bak
sudo vim /etc/hosts
```

The backup preserves the original file for recovery. For other configuration files, use an appropriate backup procedure that preserves required permissions and metadata.

Avoid blindly overwriting configuration files or making changes without understanding their effects.

## 10. Vim vs Nano

| Feature | Vim | Nano |
|---|---|---|
| Editing modes | Multiple modes | Direct text editing |
| Learning curve | Steeper | Easier |
| Navigation | Powerful keyboard commands | Straightforward keyboard shortcuts |
| Bulk editing | Extensive capabilities | More limited |
| Remote server use | Excellent | Excellent |
| Automation and advanced workflows | Strong | Basic |

Nano is a good starting point for simple edits. Vim is worth learning for advanced navigation, bulk changes, and efficient terminal workflows.

Both editors are useful for Linux system administration.

## 11. Practice Exercises

Complete these exercises to develop your Vim skills.

- [ ] **Create a file:** Open `practice.txt`, enter Insert Mode, and write five lines about Linux.

  ```bash
  vim practice.txt
  ```

- [ ] **Save and exit:** Return to Normal Mode, then save and quit.

  ```vim
  :wq
  ```

- [ ] **Navigate:** Move to the first line, last line, and line 3.

  ```vim
  gg
  G
  :3
  ```

- [ ] **Copy and paste:** Copy a line and paste it below.

  ```vim
  yy
  p
  ```

- [ ] **Search and replace:** Replace occurrences of `Linux` with `Ubuntu`, confirming each replacement.

  ```vim
  :%s/Linux/Ubuntu/gc
  ```

- [ ] **Undo and redo:** Undo a change, then redo it.

  ```text
  u
  Ctrl + r
  ```

## 12. Learn Vim Interactively

Most Vim installations include a guided tutorial. Run:

```bash
vimtutor
```

The tutorial introduces navigation, editing, deletion, and other essential commands through hands-on exercises.

If `vimtutor` is unavailable, check whether the Vim tutorial package is installed on your Linux distribution.

For additional information, consult the official documentation:

- [Vim Help Documentation](https://vimhelp.org/)
- [Vim Official Website](https://www.vim.org/)

## Recommended Learning Order

1. Understand Normal Mode, Insert Mode, and Command-Line Mode.
2. Practice navigation.
3. Learn how to save and exit.
4. Practice insertion, deletion, copying, pasting, and undo.
5. Learn search and replace.
6. Edit Linux server configuration files in a test environment.
7. Practice safe configuration changes and validation before applying them to production.

The most important habit is to return to Normal Mode using `Esc` before executing Vim commands. Once this becomes natural, editing files on Linux servers becomes much easier.

**Conclusion:** Vim is a valuable tool for Linux system administrators and server engineers. Start with basic commands, practise with sample files, and gradually apply your skills to configuration management, scripting, and remote server administration.
