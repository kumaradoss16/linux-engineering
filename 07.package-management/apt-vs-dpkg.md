`apt` and `dpkg` are package management tools in Debian-based Linux distributions such as Ubuntu, Debian, and Linux Mint.

The main difference:

- APT manages software packages, downloads packages from repositories, and automatically handles dependencies.
- DPKG installs, removes, and manages individual `.deb` package files locally. It does not automatically resolve dependencies.

Think of it this way:

- `apt` = High-level package manager.
- `dpkg` = Low-level package manager.

## 1. APT vs DPKG

| Feature               | APT                         | DPKG                      |
| --------------------- | --------------------------- | ------------------------- |
| Full form             | Advanced Package Tool       | Debian Package            |
| Level                 | High-level                  | Low-level                 |
| Package format        | `.deb`                      | `.deb`                    |
| Downloads packages    | Yes                         | No                        |
| Installs packages     | Yes                         | Yes                       |
| Dependency management | Automatic                   | Not automatic             |
| Repository support    | Yes                         | No                        |
| Updates package lists | Yes                         | No                        |
| Removes packages      | Yes                         | Yes                       |
| Common usage          | Everyday package management | Manual package operations |

## 2. APT commands with examples

APT is generally the recommended tool for installing and maintaining software on Ubuntu.

Example 1: Update package lists

```
sudo apt update
```

Downloads the latest package information from configured repositories. It does not upgrade installed software.

Example 2: Upgrade installed packages

```
sudo apt upgrade
```

Upgrades installed packages using available updates.

Example 3: Install software

```
sudo apt install nginx
```

APT:

1. Finds the Nginx package in configured repositories.
2. Checks its dependencies.
3. Downloads the required packages.
4. Installs and configures them.

Example 4: Remove software

```
sudo apt remove nginx
```

Removes Nginx but generally retains its system configuration files.

Example 5: Remove software and its configuration

```
sudo apt purge nginx
```

Removes the package and its package-managed configuration files.

Example 6: Search for packages

```
apt search nginx
```

Example 7: Display package information

```
apt show nginx
```

## 3. DPKG commands with examples

DPKG is useful when you have downloaded a `.deb` installer manually.

Example 1: Install a local `.deb` file

```
sudo dpkg -i package.deb
```

`-i` means install.

For example:

```
sudo dpkg -i google-chrome-stable_current_amd64.deb
```

DPKG attempts to install the package but does not automatically download missing dependencies.

Example 2: List installed packages

```
dpkg -l
```

Example 3: Check whether a package is installed

```
dpkg -l nginx
```

Example 4: Display package details

```
dpkg -s nginx
```

Example 5: List files installed by a package

```
dpkg -L nginx
```

Example 6: Remove a package

```
sudo dpkg -r nginx
```

Example 7: Remove a package and its configuration

```
sudo dpkg -P nginx
```

`-P` means purge.

## 4. Practical example: Installing software using APT vs DPKG

Method A - Install using APT

```
sudo apt update
sudo apt install nginx
```

Automatic dependency resolution

Method B - Install using DPKG

First, download a `.deb` package from a trusted source.

```
sudo dpkg -i nginx-package.deb
```

If dependencies are missing, DPKG may report dependency errors.

You can then ask APT to resolve and install missing dependencies:

```
sudo apt --fix-broken install
```

Manual installation may require dependency repair

## 5. How APT and DPKG work together

APT uses DPKG as its underlying package installation mechanism.


<img width="555" height="227" alt="image" src="https://github.com/user-attachments/assets/fed4fbd5-9fee-4d10-a377-312523e31f21" />


For example, when you execute:

```
sudo apt install nginx
```

APT handles repository lookup, dependency resolution and downloads. DPKG performs the actual package installation.

## 6. Practical commands to try on your Ubuntu Server

Run these commands one by one:

```
# Check APT version
apt --version

# Check DPKG version
dpkg --version

# Search for nginx
apt search nginx

# Check whether nginx is installed
dpkg -l nginx

# Display nginx package information
apt show nginx

# List installed files (if nginx is installed)
dpkg -L nginx

# Check package installation status
dpkg -s nginx
```

## 7. Interview question

Q: What is the difference between apt and dpkg in Linux?

Answer:

APT is a high-level package management tool used to install, update and remove software from repositories while automatically resolving dependencies. DPKG is a low-level Debian package management tool that directly installs, removes and manages `.deb` packages without automatically resolving dependencies. APT uses DPKG to perform package installation operations.

My recommendation: For everyday Ubuntu Server administration, use `apt` as your primary package manager. Learn `dpkg` for inspecting installed packages, managing local `.deb` files, and troubleshooting package installation problems.
