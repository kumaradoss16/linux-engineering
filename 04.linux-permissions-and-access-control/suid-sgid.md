In Linux, SUID (Set User ID) and SGID (Set Group ID, sometimes mistakenly referred to as GUID) are special permissions that temporarily grant a user extra privileges to execute a file or share directory access.

Here is how they work and what they look like.

### 1. SUID (Set User ID)

* **What it does:** When you apply SUID to an executable program, that program runs with the permissions of the file's **owner**, rather than the person who is actually running it.
* **The Permission:** It adds `4000` to the numeric permission. In symbolic format, it replaces the owner's execute `x` with an `s` (e.g., `-rwsr-xr-x`).
* **Simple Example:** The `passwd` command. A normal user needs to be able to change their own password. However, passwords are saved in the `/etc/shadow` file, which only the `root` administrator is allowed to edit. Because the `/usr/bin/passwd` program is owned by `root` and has the SUID bit set, when you run it, you temporarily "become" root just long enough to save your new password.

### 2. SGID (Set Group ID)

* **What it does:** While SGID can be applied to files (making them run as the file's group), it is almost entirely used on **directories** to enable team collaboration. When SGID is set on a directory, any new file created inside it automatically inherits the directory's group ownership, regardless of who created the file.
* **The Permission:** It adds `2000` to the numeric permission. In symbolic format, it replaces the group's execute `x` with an `s` (e.g., `drwxrwsr-x`).
* **Simple Example:** A shared company folder (`/srv/marketing`). If a user named `monal` creates a file in a normal directory, the file belongs exclusively to the `monal` group, locking out colleagues. By applying SGID to the `/srv/marketing` folder (which is owned by the `marketing` group), any file `monal` creates inside it instantly belongs to the `marketing` group, allowing the whole team to open and edit it.
