# Linux `chage` Command - All Flags and Options

The `chage` command in Linux is used to view and manage password aging and expiration settings for user accounts. It is useful for system administrators who need to configure password expiry, minimum password-change intervals, account expiration dates, and password-aging warnings.

## 1. Basic syntax

```
chage [options] USERNAME
```

For example:

```
sudo chage -l hytrel
```

This displays the password aging information for the user `hytrel`.

## 2. Common `chage` flags

| Flag | Purpose                                                                                | Example                           |
| ---- | -------------------------------------------------------------------------------------- | --------------------------------- |
| `-l` | List password aging information                                                        | `sudo chage -l hytrel`            |
| `-d` | Set the date of the last password change                                               | `sudo chage -d 2026-10-21 hytrel` |
| `-E` | Set the account expiration date                                                        | `sudo chage -E 2027-12-31 hytrel` |
| `-I` | Set the number of inactive days after password expiration before the account is locked | `sudo chage -I 30 hytrel`         |
| `-m` | Set the minimum number of days between password changes                                | `sudo chage -m 1 hytrel`          |
| `-M` | Set the maximum number of days a password remains valid                                | `sudo chage -M 90 hytrel`         |
| `-W` | Set the number of days before password expiration to display a warning                 | `sudo chage -W 7 hytrel`          |
| `-h` | Display help information                                                               | `chage -h`                        |

The commands above that modify settings require appropriate privileges. They are examples, not recommended defaults for every account.

## 3. Complete `chage` option reference

The following options are documented for the Linux `chage` utility in current shadow-utils man pages. Availability can vary with the installed version.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://man7.org\&sz=32)

Linux manual page

+1



| Flag | Long option    | Purpose                                                                       |
| ---- | -------------- | ----------------------------------------------------------------------------- |
| `-d` | `--lastday`    | Set the date of the last password change                                      |
| `-E` | `--expiredate` | Set the account expiration date                                               |
| `-h` | `--help`       | Display help                                                                  |
| `-i` | `--iso8601`    | Display dates in `YYYY-MM-DD` format                                          |
| `-I` | `--inactive`   | Set days of inactivity after password expiration before the account is locked |
| `-l` | `--list`       | Display password aging information                                            |
| `-m` | `--mindays`    | Set the minimum days between password changes                                 |
| `-M` | `--maxdays`    | Set the maximum password validity period in days                              |
| `-R` | `--root`       | Apply changes using a specified chroot directory                              |
| `-P` | `--prefix`     | Apply changes to configuration files under a specified prefix directory       |
| `-W` | `--warndays`   | Set the number of days before password expiration to warn the user            |

The `-R` and `-P` options are mainly useful for administrators managing alternate or offline Linux root filesystems.

## 4. Practical examples for Ubuntu Server

### View password aging information

```
sudo chage -l hytrel
```

### Force a password change at the next login

```
sudo chage -d 0 hytrel
```

### Set password validity to 90 days

```
sudo chage -M 90 hytrel
```

### Require at least one day between password changes

```
sudo chage -m 1 hytrel
```

### Warn the user seven days before password expiration

```
sudo chage -W 7 hytrel
```

### Lock the account after 30 days of inactivity following password expiration

```
sudo chage -I 30 hytrel
```

### Set an account expiration date

```
sudo chage -E 2027-12-31 hytrel
```

This sets the account expiration date, which is different from setting the password's maximum age.

### Remove the account expiration date

```
sudo chage -E -1 hytrel
```

### Display dates in ISO format

```
sudo chage -i -l hytrel
```

### Display help

```
chage --help
```

Caution: These commands modify account settings. Test them with a lab account before applying them to production users. In particular, `-E` can prevent an account from being used after its expiration date.

## 5. Understand the difference between `-E`, `-I`, and `-M`

| Flag            | What expires or changes?                                           |
| --------------- | ------------------------------------------------------------------ |
| `-M 90`         | Password validity period                                           |
| `-I 30`         | Inactivity period after password expiration before account locking |
| `-E 2027-12-31` | Account expiration date                                            |

These options control different parts of account lifecycle management.

## 6. Recommended practice sequence

Run these commands to explore `chage` safely:

```
# 1. View current account aging settings
sudo chage -l hytrel

# 2. View available options
chage --help

# 3. Check the current account expiration settings again
sudo chage -l hytrel
```

For the complete reference, consult the [`chage`](https://man7.org/linux/man-pages/man1/chage.1.html?utm_source=chatgpt.com)[ Linux manual page](https://man7.org/linux/man-pages/man1/chage.1.html?utm_source=chatgpt.com).
