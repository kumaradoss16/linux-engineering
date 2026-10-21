# Ubuntu Server: Fix “Login incorrect” Using GRUB Recovery Mode

Since you can open the GRUB recovery menu, you can troubleshoot your Ubuntu Server locally without relying on `sudo` from `developer01`.

The safest approach is to check the account first, reset its password if necessary, and verify that the account is not locked or otherwise misconfigured.

Your earlier error, `developer01 is not in the sudoers file`, is a separate issue: it means the account lacks sudo authorization. It does not, by itself, explain why console login fails.

## Step 1: Open the recovery root shell

1. Restart your Ubuntu Server VM.
2. Display the GRUB menu. If it is hidden, try holding Shift on a legacy BIOS boot or tapping Esc on a UEFI boot during startup.
3. Select Advanced options for Ubuntu.
4. Select the kernel entry ending in `(recovery mode)`.
5. In the recovery menu, select root — Drop to root shell prompt.

You should reach a prompt similar to:

```
root@Ubuntu-Server:~#
```

This is a privileged shell. Use it only to repair the system you administer.

## Step 2: Remount the root filesystem as writable

The recovery shell may start with the root filesystem mounted read-only.

Run:

```
mount -o remount,rw /
```

Verify the mount status:

```
findmnt -no TARGET,OPTIONS /
```

Look for `rw` in the options. If it still shows `ro`, stop and troubleshoot the mount issue before attempting account changes.

## Step 3: Check whether the account exists

At the root prompt, run:

```
getent passwd developer01
```

If the account exists, you should see an entry similar to:

```
developer01:x:1001:1001:Developer:/home/developer01:/bin/bash
```

Your actual UID, GID, description, home directory, and shell may differ.

If there is no output, check the local account file:

```
grep '^developer01:' /etc/passwd
```

Do not create a duplicate account if it already exists.

## Step 4: Reset the user's password

If the account exists, run:

```
passwd developer01
```

Enter a new strong password when prompted, and enter it again to confirm. Password characters will not appear while you type.

Successful output should resemble:

```
passwd: password updated successfully
```

Ubuntu's password-recovery guidance recommends resetting the affected user's password from the recovery root shell.&#x20;


## Step 5: Check the account's password status

Run:

```
passwd -S developer01
```

The output typically includes the username, a password-status indicator, and password-aging dates.

- `P` — a usable password is set.
- `L` — the password is locked.
- `NP` — no password is set.

If the account remains locked after resetting the password, investigate why before unlocking it. Avoid blindly changing authentication settings.

## Step 6: Restore sudo privileges if intended

Because your earlier error indicated that `developer01` was not authorized to use `sudo`, you may also need to restore its administrator-group membership.

If `developer01` is supposed to be an administrator on this lab server, run:

```
usermod -aG sudo developer01
```

This adds the user to the `sudo` group without replacing existing supplementary group memberships. Ubuntu commonly uses this group to authorize administrative commands.&#x20;


Only do this if `developer01` is intended to have administrative privileges.

## Step 7: Reboot and test

Run:

```
sync
reboot
```

After the server restarts, sign in at the console using:

- Username: `developer01`
- Password: the new password you just set

Once logged in, test:

```
whoami
groups
sudo -l
```

If `sudo -l` works and lists your permitted commands, your sudo authorization is restored.

## If the recovery root shell asks for a password

Stop there rather than repeatedly guessing. Tell me the exact message displayed. The next recovery method may require booting a trusted Ubuntu installation ISO and repairing the installed system from there.

Important: These steps assume this is your own Ubuntu Server VM or a system you are authorized to administer. Resetting a password and changing group membership are separate operations; both may be necessary to resolve the problems you described.
