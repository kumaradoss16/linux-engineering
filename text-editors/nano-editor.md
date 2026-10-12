# Basics of Nano Editor in Linux

The Nano editor is a simple, beginner-friendly, command-line text editor used in Linux to create, view, and modify files directly from the terminal. It is especially useful for System Administrators, Network Engineers, and Linux users who need to edit configuration files without a graphical interface.

For example, you can use Nano to edit network configuration files, shell scripts, system settings, and documentation.

## 1. What is Nano Editor?

[Using the nano text editor · SFTP Gateway Support](https://images.openai.com/static-rsc-4/vGzkmjuhxbb7QmrolL1qAKgEJm0itVV5fjUglmM2oT0-ka4kSpdJMgPVmtqWL_R5c_0e9TRbWhMTdWJS4lNVOP4bOPI3zt7rIotvlri8Cf4OOvAmzOSYJlMsvZTfQWAm5mD5g-RoDAW8ijMh1LIrs5ni53_9DJNl0i5H5NE33Ro?purpose=inline)

[help.thorntech.com](https://help.thorntech.com/docs/2.001.00/sftp-gateway-azure/azure-nano/)

Nano is a terminal-based text editor that runs inside your command-line environment.

| Feature           | Description                                    |
| ----------------- | ---------------------------------------------- |
| Type              | Command-line text editor                       |
| Operating systems | Linux, Unix-like systems                       |
| Interface         | Terminal-based                                 |
| Beginner friendly | Yes                                            |
| Common uses       | Editing configuration files, scripts, and text |
| Shortcut style    | `Ctrl`, `Alt`, and keyboard combinations       |

Unlike graphical editors, Nano does not require a desktop environment. You can use it through an SSH connection to a remote Linux server.

## 2. How to open Nano

Open your Linux terminal and run the following commands.

### Open Nano without a filename

```
nano
```

This opens an empty editor.

### Create or edit a file

```
nano notes.txt
```

If `notes.txt` exists, Nano opens it. If it does not exist, Nano lets you create it.

### Edit a configuration file

```
nano config.txt
```

For a real system configuration file, use its actual path, such as:

```
sudo nano /etc/hosts
```

`sudo` runs Nano with administrative privileges, allowing you to edit files that require root permissions. Use it carefully.

## 3. Understanding the Nano interface

When you open Nano, the screen generally contains three main areas.

GNU nano - notes.txt

Welcome to Linux Nano Editor  This is my first text file. I am learning Linux administration.

^G Help   ^O Write Out   ^W Where Is

^X Exit   ^K Cut         ^U Paste

Illustrative Nano screen. The exact layout and available shortcuts can vary by version and terminal size.

- Top bar: Shows the editor and filename.
- Main editing area: Where you type and modify text.
- Bottom shortcut bar: Shows common commands and their keyboard shortcuts.

### What does `^X` mean?

The caret symbol `^` represents the Ctrl key.

| Nano notation | Keyboard action |
| ------------- | --------------- |
| `^X`          | `Ctrl + X`      |
| `^O`          | `Ctrl + O`      |
| `^W`          | `Ctrl + W`      |
| `^K`          | `Ctrl + K`      |
| `^U`          | `Ctrl + U`      |

For example, `^X` means press and hold `Ctrl`, then press `X`.

## 4. Essential Nano keyboard shortcuts

These are the most important commands to learn first.

| Shortcut   | Purpose                           |
| ---------- | --------------------------------- |
| `Ctrl + O` | Save the file (Write Out)         |
| `Ctrl + X` | Exit Nano                         |
| `Ctrl + W` | Search for text                   |
| `Ctrl + K` | Cut the current line              |
| `Ctrl + U` | Paste previously cut text         |
| `Ctrl + G` | Open help                         |
| `Ctrl + C` | Display cursor position           |
| `Ctrl + \` | Search and replace                |
| `Ctrl + _` | Go to a specified line and column |
| `Alt + U`  | Undo                              |
| `Alt + E`  | Redo                              |

Some shortcuts may differ across versions or terminal configurations. Nano's `Ctrl + G` help screen lists the supported commands.

## 5. How to create, edit, and save a file

Let's create a small Linux practice file.

### Step 1: Open a new file

```
nano linux-notes.txt
```

### Step 2: Type the following content

```
Linux Administration Notes

1. Linux is an operating system.
2. Nano is a command-line text editor.
3. Linux administrators edit configuration files.
```

Simply start typing. Nano does not require a separate insert mode.

### Step 3: Save the file

Press:

`Ctrl + O`

Nano displays a filename prompt at the bottom. Press `Enter` to confirm the filename.

### Step 4: Exit Nano

Press:

`Ctrl + X`

### Step 5: Verify the saved file

Back in the terminal, execute:

```
cat linux-notes.txt
```

This prints the saved contents to the terminal.

## 6. How to edit existing text

Open your file again:

```
nano linux-notes.txt
```

Use the arrow keys to move the cursor.

You can then:

- Add new text by typing.
- Remove characters using `Backspace` or `Delete`.
- Cut a line with `Ctrl + K`.
- Paste cut text with `Ctrl + U`.
- Search for a word with `Ctrl + W`.

After editing, press `Ctrl + O`, then `Enter` to save. Press `Ctrl + X` to exit.

Important: `Ctrl + K` cuts the line containing the cursor. If you cut several lines, Nano stores them in its cut buffer, and `Ctrl + U` pastes the buffered text.

## 7. Searching and replacing text

Suppose your file contains the word `server`, and you want to locate it.

### Search for a word

1. Press `Ctrl + W`.
2. Type `server`.
3. Press `Enter`.

Nano moves the cursor to a matching occurrence.

### Replace text

1. Press `Ctrl + \`.
2. Enter the text to search for, such as `server`.
3. Press `Enter`.
4. Enter the replacement, such as `linux-server`.
5. Press `Enter`.
6. Follow the prompts to replace a match or all occurrences, depending on your choice.

Review replacements before saving, especially when editing configuration files.

## 8. Practical example for a Linux System Administrator

One common task is adding a hostname mapping to `/etc/hosts`.

Open the file:

```
sudo nano /etc/hosts
```

Add a suitable entry, if required by your environment:

```
192.168.1.10    fileserver.lab.local fileserver
```

Save using `Ctrl + O`, confirm with `Enter`, and exit using `Ctrl + X`.

Verify the entry:

```
getent hosts fileserver.lab.local
```

If the system resolves the hostname using that entry, the command should return the configured IP address.

This example demonstrates a real administrative workflow: open a configuration file → edit it → save it → verify the result. Only use IP addresses and hostnames appropriate for your network.

## 9. Nano versus Vim

| Feature             | Nano                 | Vim                          |
| ------------------- | -------------------- | ---------------------------- |
| Learning difficulty | Easy                 | More advanced                |
| Typing text         | Type immediately     | Requires understanding modes |
| Saving              | `Ctrl + O`           | `:w`                         |
| Exiting             | `Ctrl + X`           | `:q`                         |
| Best for beginners  | Yes                  | Takes more practice          |
| Advanced editing    | Good for basic tasks | Extensive capabilities       |

Both are useful. Start with Nano to learn basic file editing, then consider Vim as you become more comfortable with Linux administration.

## 10. Practice exercises

## Nano practice checklist

<img width="718" height="462" alt="image" src="https://github.com/user-attachments/assets/2b7ea8f4-4f04-40f6-b61d-a5205ec827ee" />


## Key takeaway

For a beginner learning Linux and system administration, remember these five Nano commands first:

- `nano filename` - open or create a file.
- `Ctrl + O` - save.
- `Ctrl + X` - exit.
- `Ctrl + W` - search.
- `Ctrl + K` - cut a line.

Once you can create, edit, save, and verify a text file, you have learned the essential Nano workflow used in everyday Linux administration.
