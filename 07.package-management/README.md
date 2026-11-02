# Linux Package Management: From Beginner to Server Administration

## Introduction

Package management installs, upgrades, configures, verifies, and removes software while tracking dependencies and metadata. On Debian and Ubuntu systems, **APT** is the high-level package manager and **dpkg** is the low-level package database and `.deb` package tool.

This tutorial focuses on Debian/Ubuntu APT-based systems. It also briefly compares APT with Snap and source compilation.

Run all labs only on a disposable Ubuntu 22.04/24.04 VM or container. Take a VM snapshot before starting. Do not remove packages, purge configurations, edit repositories, or upgrade production servers without an approved maintenance plan and rollback procedure.

Your package versions, repository contents, available updates, and output will differ from the examples.

## Lab safety

Create a lab directory:

```bash
mkdir -p ~/package-lab/reports
cd ~/package-lab
```

Check the system:

```bash
cat /etc/os-release
apt --version
dpkg --version
whoami
id
sudo -v
```

Safety rules:

- Always run `sudo apt update` before installing, upgrading, or checking available updates.
- Use `--dry-run` or `-s` before large installs, removals, or upgrades.
- Review `apt autoremove` output before accepting it.
- Do not blindly remove old kernels.
- Do not add untrusted repositories or run `curl ... | bash` installers.
- Do not use `--allow-insecure-repositories` except in a controlled recovery situation.
- Back up important data and configurations before major upgrades.
- Keep at least one known-good previous kernel available.

***

# Part 1: Package Management Fundamentals

## What is a package?

A package is a collection of files, metadata, scripts, configuration, and dependency information distributed as one installable unit.

A Debian binary package is a `.deb` archive containing:

- Executables and libraries.
- Configuration files.
- Documentation.
- Maintainer scripts.
- Package metadata.
- Dependency declarations.
- File checksums.

A **source package** contains source code and build instructions. A **binary package** contains compiled, installable files.

Inspect a package’s metadata:

```bash
apt show nginx
```

Example output; your values will differ:

```text
Package: nginx
Version: 1.24.0-2ubuntu7
Architecture: amd64
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Installed-Size: 1,554 kB
Depends: libc6, libpcre3, libssl3, zlib1g
Description: high performance web server and reverse proxy server
```

## Package-management layers

| Tool | Level | Main role |
|---|---|---|
| `dpkg` | Low-level | Install, remove, inspect, and configure local `.deb` packages |
| `apt` | High-level | Resolve dependencies, download packages, manage repositories |
| `apt-get` | High-level | Script-friendly APT interface |
| `apt-cache` | Query tool | Query package metadata and repository policy |
| `apt-mark` | Selection tool | Manage manual/auto and hold states |

### `apt`, `apt-get`, and `apt-cache`

- `apt` is the user-facing command intended for interactive use.
- `apt-get` is the traditional, script-oriented interface.
- `apt-cache` queries the local APT package cache.
- `dpkg` operates directly on installed packages and local `.deb` files.

APT resolves dependencies from configured repositories. `dpkg` generally does not download dependencies automatically.

Use `dpkg` for:

- Installing a local `.deb`.
- Inspecting installed files.
- Checking package ownership of a file.
- Recovering interrupted package configuration.
- Low-level package auditing.

Use APT for normal repository-based installation, upgrades, dependency repair, and removal.

## Repositories

An APT repository is a signed collection of package indexes and `.deb` files.

### Ubuntu repository components

| Component | Typical contents |
|---|---|
| `main` | Officially supported open-source software |
| `restricted` | Proprietary drivers and limited-support software |
| `universe` | Community-maintained software |
| `multiverse` | Software with legal or licensing restrictions |

### Repository types

| Repository type | Purpose |
|---|---|
| Release/archive | Base distribution packages |
| Security | Security updates |
| Updates | Recommended stable updates |
| Backports | Newer software backported to an older release |
| PPA | Personal Package Archive; third-party Ubuntu repository |
| Third-party repository | Vendor or community repository outside Ubuntu’s official sources |

### Repository configuration files

Traditional format:

```text
/etc/apt/sources.list
/etc/apt/sources.list.d/*.list
```

Modern deb822 format:

```text
/etc/apt/sources.list.d/*.sources
```

Example traditional entry:

```text
deb http://archive.ubuntu.com/ubuntu noble main restricted universe multiverse
deb http://security.ubuntu.com/ubuntu noble-security main restricted universe multiverse
```

Example deb822 entry:

```ini
Types: deb
URIs: http://archive.ubuntu.com/ubuntu/
Suites: noble
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

### GPG signing

Repositories should be cryptographically signed. APT verifies repository metadata using keys stored in keyrings.

Modern repository configuration commonly uses:

```text
Signed-By=/etc/apt/keyrings/vendor-keyring.gpg
```

or:

```text
Signed-By=/usr/share/keyrings/vendor-keyring.gpg
```

Avoid untrusted repositories and random `curl | bash` installation. They can introduce unverified software, unstable dependencies, and supply-chain risk.

## Package metadata

| Metadata field | Meaning |
|---|---|
| Package | Package name |
| Version | Upstream and Debian/Ubuntu revision |
| Architecture | CPU architecture, such as `amd64` or `arm64` |
| Maintainer | Person or team responsible for the package |
| Description | Package purpose |
| Depends | Required dependencies |
| Recommends | Recommended but not strictly required packages |
| Suggests | Optional enhancements |
| Conflicts | Packages that cannot coexist |
| Replaces | Package whose files this package replaces |
| Provides | Virtual package or capability provided |
| Installed-Size | Approximate disk use after installation |
| Checksums | Integrity data for package files |

## Dependencies

- **Direct dependency**: A package required by another package.
- **Reverse dependency**: A package that depends on the package you are considering removing.
- **Automatically installed**: Installed as a dependency of another package.
- **Manually installed**: Explicitly requested by an administrator or user.
- **Broken dependency**: A required package version or package is missing or incompatible.

Removing a dependency package can affect other software. Always inspect reverse dependencies before removal:

```bash
apt-cache rdepends nginx
```

## Package states

| State | Meaning |
|---|---|
| `install` | Package is selected for installation |
| `hold` | Package is prevented from being upgraded |
| `deinstall` | Package is selected for removal |
| `purge` | Package and configuration files are selected for removal |
| `ii` | Installed successfully |
| `rc` | Removed but configuration files remain |
| `hi` | Held and installed |
| `un` | Not installed |

Inspect package states:

```bash
dpkg -l nginx
dpkg --get-selections | grep nginx
```

Example output:

```text
ii  nginx  1.24.0-2ubuntu7  amd64  high performance web server and reverse proxy server
```

***

# Part 2: APT Commands

## `apt update`

### Purpose

Refreshes package metadata from configured repositories.

### Syntax

```bash
sudo apt update [options]
```

### Important flags

| Flag | Meaning | Example |
|---|---|---|
| `--allow-insecure-repositories` | Allow insecure repositories for advanced recovery only | Avoid unless necessary |
| `-o` | Apply a temporary APT configuration option | `apt -o Debug::pkgProblemResolver=true update` |

### Beginner example

```bash
sudo apt update
```

Example output; your values will differ:

```text
Hit:1 http://archive.ubuntu.com/ubuntu noble InRelease
Get:2 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Reading package lists... Done
Building dependency tree... Done
All packages are up to date.
```

### Server-administration example

Save an update log for change management:

```bash
sudo apt update 2>&1 | tee ~/package-lab/reports/apt-update.log
```

### Key point

`apt update` downloads repository indexes. It does **not** install or upgrade packages.

Run it before:

```bash
sudo apt install PACKAGE
sudo apt upgrade
apt list --upgradable
```

### Common mistakes

- Assuming `apt update` upgrades software.
- Ignoring GPG, 404, or repository errors.
- Adding a repository without understanding its signing and support model.

### Verification

```bash
apt list --upgradable
apt-cache policy nginx
```

***

## `apt search`

### Purpose

Searches package names and descriptions.

### Syntax

```bash
apt search [options] pattern
```

### Important flags

| Flag | Meaning | Example |
|---|---|---|
| `--full` | Show full package description | `apt search --full nginx` |
| `--names-only` | Search only package names | `apt search --names-only nginx` |

### Beginner examples

```bash
apt search nginx
apt search --names-only nginx
apt search "web server"
apt search python3-flask
```

### Server-administration example

Find monitoring tools:

```bash
apt search --names-only monitoring | head -n 20
```

### Common mistakes

- Assuming search results are all installed.
- Searching before `apt update`, which can produce stale results.
- Confusing search results with package availability for your release.

### Verification

```bash
apt show PACKAGE
apt-cache policy PACKAGE
```

***

## `apt show`

### Purpose

Displays detailed metadata for a package.

### Syntax

```bash
apt show [options] package_name
```

### Important flags

| Flag | Meaning | Example |
|---|---|---|
| `-a` | Show all available versions | `apt show -a nginx` |
| `-o` | Apply temporary configuration option | `apt show -o APT::Get::List-Cleanup=0 nginx` |

### Beginner example

```bash
apt show nginx
```

### Server-administration example

Inspect dependencies before installing:

```bash
apt show python3-flask
```

### Reading output

| Field | Meaning |
|---|---|
| `Package` | Package name |
| `Version` | Candidate version |
| `Priority` | Package priority |
| `Section` | Repository section |
| `Maintainer` | Maintainer |
| `Installed-Size` | Approximate installed size |
| `Depends` | Required dependencies |
| `Recommends` | Recommended packages |
| `Suggests` | Optional packages |
| `Conflicts` | Incompatible packages |
| `Download-Size` | Download size |
| `APT-Manual-Installed` | Whether APT considers it manually installed |

### Common mistakes

- Assuming `Recommends` packages are mandatory.
- Ignoring `Conflicts` before installation.
- Reading metadata without refreshing the cache.

### Verification

```bash
apt-cache policy nginx
apt-cache depends nginx
```

***

## `apt list`

### Purpose

Lists packages based on installed, upgradable, or version criteria.

### Syntax

```bash
apt list [options] [pattern]
```

### Important flags

| Flag | Meaning | Example |
|---|---|---|
| `--installed` | Show installed packages | `apt list --installed` |
| `--upgradable` | Show upgradable packages | `apt list --upgradable` |
| `--all-versions` | Show all available versions | `apt list --all-versions nginx` |
| `-a` | Same as `--all-versions` | `apt list -a nginx` |

### Beginner examples

```bash
apt list --installed | grep nginx
apt list --upgradable
apt list --all-versions nginx
apt list "python3-*" | head -n 20
```

### Server-administration example

Generate an upgradable-package inventory:

```bash
apt list --upgradable > ~/package-lab/reports/upgradable-packages.txt
```

### Common mistakes

- Confusing `--installed` with `--upgradable`.
- Assuming `apt list` output is sorted or stable for scripting.
- Using `apt list` in automation when `dpkg-query` may be more suitable.

### Verification

```bash
dpkg -l nginx
apt-cache policy nginx
```

***

## `apt install`

### Purpose

Installs packages and resolves dependencies.

### Syntax

```bash
sudo apt install [options] package_name
```

### Important flags

| Flag | Meaning | Example |
|---|---|---|
| `-y` | Assume yes | `sudo apt install -y nginx` |
| `--no-install-recommends` | Skip recommended packages | `sudo apt install --no-install-recommends nginx` |
| `--install-suggests` | Install suggested packages | `sudo apt install --install-suggests nginx` |
| `--dry-run` | Simulate installation | `sudo apt install --dry-run nginx` |
| `-s` | Simulate | `sudo apt install -s nginx` |
| `--reinstall` | Reinstall package | `sudo apt install --reinstall nginx` |
| `--fix-broken` | Repair broken dependencies | `sudo apt install --fix-broken` |
| `--download-only` | Download without installing | `sudo apt install --download-only nginx` |
| `-o` | Temporary configuration option | `sudo apt -o Debug::pkgProblemResolver=true install nginx` |

### Beginner example

```bash
sudo apt install nginx
```

### Simulate first

```bash
sudo apt install --dry-run nginx
```

### Server-administration examples

Install a web server:

```bash
sudo apt install -y nginx
```

Install monitoring tools:

```bash
sudo apt install -y htop iotop sysstat
```

Install a local `.deb` with dependency resolution:

```bash
sudo apt install ./package.deb
```

Reinstall a package:

```bash
sudo apt install --reinstall nginx
```

### Common mistakes

- Running `apt install` without `apt update`.
- Using `-y` blindly during production changes.
- Installing packages from untrusted repositories.
- Assuming `--no-install-recommends` is always desirable.

### Verification

```bash
dpkg -s nginx
dpkg -L nginx
apt-cache policy nginx
```

***

## `apt remove` and `apt purge`

### Purpose

Removes packages; `purge` also removes system-wide configuration files.

### Syntax

```bash
sudo apt remove [options] package_name
sudo apt purge [options] package_name
```

### Important flags

| Flag | Meaning | Example |
|---|---|---|
| `-y` | Assume yes | `sudo apt remove -y nginx` |
| `--dry-run` or `-s` | Simulate | `sudo apt remove -s nginx` |
| `--autoremove` | Remove unneeded dependencies | `sudo apt remove --autoremove nginx` |
| `--purge` | Remove configuration files too | `sudo apt remove --purge nginx` |

### Beginner example

```bash
sudo apt remove nginx
sudo apt purge nginx
```

### Server-administration example

Simulate a removal before applying it:

```bash
sudo apt remove --dry-run nginx
sudo apt purge --dry-run nginx
```

Remove a package and no-longer-needed dependencies:

```bash
sudo apt remove --autoremove nginx
```

### Remove versus purge

| Command | Binaries/libraries | System-wide configuration |
|---|---|---|
| `apt remove` | Removed | Usually retained |
| `apt purge` | Removed | Removed |

### Common mistakes

- Using `purge` without confirming that configuration removal is intended.
- Running `autoremove` without reviewing proposed removals.
- Removing a package that is a reverse dependency of required software.

### Verification

```bash
dpkg -s nginx || echo "Package absent"
dpkg -l | grep '^rc' | grep nginx || true
```

***

## `apt upgrade` and `apt full-upgrade`

### Purpose

Installs available package upgrades.

### Syntax

```bash
sudo apt upgrade [options]
sudo apt full-upgrade [options]
```

### Important flags

| Flag | Meaning | Example |
|---|---|---|
| `-y` | Assume yes | `sudo apt upgrade -y` |
| `--dry-run` or `-s` | Simulate | `sudo apt full-upgrade --dry-run` |
| `--no-install-recommends` | Skip recommended packages | `sudo apt upgrade --no-install-recommends` |
| `--with-new-pkgs` | Allow installation of new dependencies during upgrade | `sudo apt upgrade --with-new-pkgs` |
| `-o` | Temporary configuration option | `sudo apt -o Debug::pkgProblemResolver=true upgrade` |

### Beginner example

```bash
sudo apt update
sudo apt upgrade
```

### Server-administration example

Simulate a full upgrade:

```bash
sudo apt update
sudo apt full-upgrade --dry-run
```

Apply it during an approved maintenance window:

```bash
sudo apt full-upgrade
```

### Upgrade versus full-upgrade

| Command | Installs upgrades | May remove packages |
|---|---|---|
| `apt upgrade` | Yes | Generally no |
| `apt full-upgrade` | Yes | Yes, when needed to resolve dependencies |

`full-upgrade` requires more caution on production servers because it may remove or replace packages.

### Reboot requirements

```bash
cat /var/run/reboot-required 2>/dev/null || echo "No reboot required"
cat /var/run/reboot-required.pkgs 2>/dev/null || true
```

Services may need restarting after library upgrades. Check for outdated restarted services using tools such as `needrestart` if installed.

### Common mistakes

- Running `full-upgrade` without a snapshot or backup.
- Ignoring package removal prompts.
- Assuming a package upgrade automatically restarts every dependent service.
- Upgrading during business hours without a maintenance window.

### Verification

```bash
apt list --upgradable
systemctl --failed
journalctl -p err -b --no-pager
```

***

## `apt autoremove`, `autoclean`, and `clean`

### Purpose

Removes unneeded packages and cleans downloaded package caches.

### Syntax

```bash
sudo apt autoremove [options]
sudo apt autoclean [options]
sudo apt clean [options]
```

### Important flags

| Flag | Meaning | Example |
|---|---|---|
| `-y` | Assume yes | `sudo apt autoremove -y` |
| `--dry-run` or `-s` | Simulate | `sudo apt autoremove --dry-run` |
| `--purge` | Remove configuration files too | `sudo apt autoremove --purge` |

### Beginner examples

```bash
sudo apt autoremove --dry-run
sudo apt autoremove
sudo apt autoclean
sudo apt clean
```

### Meaning

| Command | Effect |
|---|---|
| `autoremove` | Removes automatically installed packages no longer needed |
| `autoclean` | Removes obsolete downloaded `.deb` files |
| `clean` | Removes all cached downloaded `.deb` files |

### Server-administration example

Check cache size:

```bash
du -sh /var/cache/apt/archives
```

Clean safely:

```bash
sudo apt autoclean
```

### Common mistakes

- Running `autoremove -y` without reviewing the list.
- Assuming `autoclean` removes installed packages.
- Removing old kernels automatically without a tested recovery plan.

### Verification

```bash
du -sh /var/cache/apt/archives
apt-mark showauto
apt-mark showmanual
```

***

## `apt-cache`

### Purpose

Queries package metadata from the local APT cache.

### Syntax

```bash
apt-cache [options] COMMAND [package]
```

### Important commands

| Command | Meaning | Example |
|---|---|---|
| `search` | Search names and descriptions | `apt-cache search nginx` |
| `show` | Display package metadata | `apt-cache show nginx` |
| `policy` | Show installed/candidate versions and priorities | `apt-cache policy nginx` |
| `depends` | Show dependencies | `apt-cache depends nginx` |
| `rdepends` | Show reverse dependencies | `apt-cache rdepends nginx` |
| `showpkg` | Show detailed package information | `apt-cache showpkg nginx` |
| `stats` | Show cache statistics | `apt-cache stats` |
| `pkgnames` | List known package names | `apt-cache pkgnames python3 \| head` |
| `unmet` | Show unmet dependencies | `apt-cache unmet` |

### Beginner example

```bash
apt-cache policy nginx
```

Example output; your values will differ:

```text
nginx:
  Installed: 1.24.0-2ubuntu7
  Candidate: 1.24.0-2ubuntu7.1
  Version table:
 *** 1.24.0-2ubuntu7.1 500
        500 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages
        100 /var/lib/dpkg/status
     1.24.0-2ubuntu7 500
        500 http://archive.ubuntu.com/ubuntu noble/main amd64 Packages
```

### Server-administration examples

```bash
apt-cache depends nginx
apt-cache rdepends nginx
apt-cache unmet
```

### Common mistakes

- Querying stale cache without `apt update`.
- Confusing candidate version with installed version.
- Ignoring repository priorities.

### Verification

```bash
apt-cache policy nginx
apt-cache policy
```

***

## `apt-mark`

### Purpose

Manages manual/automatic package selection and upgrade holds.

### Syntax

```bash
apt-mark [options] COMMAND [package]
```

### Important commands

| Command | Meaning | Example |
|---|---|---|
| `showmanual` | Show manually installed packages | `apt-mark showmanual` |
| `showauto` | Show automatically installed packages | `apt-mark showauto` |
| `showhold` | Show held packages | `apt-mark showhold` |
| `hold` | Prevent package upgrades | `sudo apt-mark hold nginx` |
| `unhold` | Allow upgrades again | `sudo apt-mark unhold nginx` |
| `auto` | Mark package automatically installed | `sudo apt-mark auto PACKAGE` |
| `manual` | Mark package manually installed | `sudo apt-mark manual PACKAGE` |

### Beginner example

```bash
apt-mark showhold
sudo apt-mark hold nginx
apt-mark showhold
sudo apt-mark unhold nginx
```

### Server-administration example

Hold a package during a maintenance window:

```bash
sudo apt-mark hold nginx
apt list --upgradable
```

After maintenance:

```bash
sudo apt-mark unhold nginx
sudo apt update
sudo apt install --only-upgrade nginx
```

### Common mistakes

- Holding a package indefinitely.
- Forgetting held packages during security patching.
- Marking an important package as automatic, allowing future `autoremove`.

### Verification

```bash
apt-mark showhold
apt-cache policy nginx
dpkg --get-selections | grep nginx
```

***

# Part 3: dpkg Commands

## Purpose

`dpkg` is the low-level Debian package manager. It installs local `.deb` files, maintains the package database, and inspects installed packages.

### Syntax

```bash
dpkg [options] action
```

### Important commands and flags

| Command/flag | Meaning | Example |
|---|---|---|
| `-i`, `--install` | Install a `.deb` package | `sudo dpkg -i package.deb` |
| `-r`, `--remove` | Remove package | `sudo dpkg -r package` |
| `-P`, `--purge` | Remove package and configuration | `sudo dpkg -P package` |
| `-l`, `--list` | List packages | `dpkg -l` |
| `-L`, `--listfiles` | List files installed by a package | `dpkg -L nginx` |
| `-s`, `--status` | Show package status | `dpkg -s nginx` |
| `-S`, `--search` | Find package owning a file | `dpkg -S /usr/sbin/nginx` |
| `-C`, `--audit` | Find partially installed packages | `dpkg -C` |
| `--configure` | Configure unpacked package | `sudo dpkg --configure -a` |
| `--unpack` | Unpack without configuring | `sudo dpkg --unpack package.deb` |
| `--verify`, `-V` | Verify installed files | `dpkg --verify nginx` |
| `--get-selections` | Show package selections | `dpkg --get-selections` |
| `--set-selections` | Apply package selections | `dpkg --set-selections < selections.txt` |

### Beginner example

```bash
dpkg -l nginx
dpkg -L nginx
dpkg -s nginx
dpkg -S /usr/sbin/nginx
```

### Server-administration example

Install a local `.deb` and repair dependencies if needed:

```bash
sudo dpkg -i package.deb || sudo apt --fix-broken install
```

Recover interrupted configuration:

```bash
sudo dpkg --configure -a
dpkg -C
```

### Why dpkg matters

`dpkg` is best for local `.deb` files and low-level inspection. It does not automatically download dependencies from repositories.

If a `dpkg -i` installation fails because of dependencies:

```bash
sudo apt --fix-broken install
```

`dpkg --configure -a` is important after an interrupted installation because it completes configuration of unpacked packages.

### Common mistakes

- Using `dpkg -i` for repository packages instead of APT.
- Ignoring dependency errors.
- Running `dpkg --configure -a` without reading its output.
- Assuming `dpkg --verify` detects every form of compromise.

### Verification

```bash
dpkg -C
dpkg -s PACKAGE
apt-cache policy PACKAGE
```

***

# Part 4: Package Verification and Security

## Package verification

Package verification helps detect accidental corruption, incomplete upgrades, and unexpected file changes. It is one control among several; combine it with logs, file-integrity monitoring, backups, and configuration management.

### dpkg verification

```bash
dpkg --verify nginx
dpkg -V nginx
```

No output usually means no mismatch was detected for files with recorded checksums.

Configuration files may legitimately differ from package defaults. Modified binaries or libraries deserve investigation.

### debsums

Install:

```bash
sudo apt install debsums
```

Check one package:

```bash
debsums nginx
```

Check changed files only:

```bash
sudo debsums -c
```

Silent mode; report only problems:

```bash
sudo debsums -s
```

## GPG keys and repositories

Modern APT repository configuration should use `Signed-By` and a dedicated keyring.

Example:

```ini
Types: deb
URIs: https://example.com/apt
Suites: stable
Components: main
Signed-By: /etc/apt/keyrings/example-archive-keyring.gpg
```

The older `apt-key` interface is deprecated. Prefer downloading a key into `/etc/apt/keyrings/` or `/usr/share/keyrings/` and referencing it with `Signed-By`.

## Security updates

A security update fixes a vulnerability or security defect. A regular update may include bug fixes or feature improvements. A full upgrade may change dependencies and remove packages.

Check available updates:

```bash
sudo apt update
apt list --upgradable
```

Upgrade only one package:

```bash
sudo apt install --only-upgrade nginx
```

Review package policy and changelog:

```bash
apt-cache policy nginx
sudo apt-get changelog nginx
```

## Unattended upgrades

Install:

```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
```

Inspect configuration:

```bash
cat /etc/apt/apt.conf.d/20auto-upgrades
cat /etc/apt/apt.conf.d/50unattended-upgrades
```

Important settings:

| Setting | Meaning |
|---|---|
| `Update-Package-Lists` | Refresh package indexes automatically |
| `Unattended-Upgrade` | Apply automatic upgrades |
| `Allowed-Origins` | Repositories allowed for automatic upgrades |
| `Automatic-Reboot` | Reboot automatically if required |
| `Automatic-Reboot-Time` | Scheduled reboot time |
| `MailReport` | Email reporting configuration |

Check status:

```bash
systemctl status unattended-upgrades --no-pager
systemctl list-timers | grep apt
cat /var/log/unattended-upgrades/unattended-upgrades.log
cat /var/log/unattended-upgrades/unattended-upgrades-dpkg.log
```

Third-party repositories are not automatically included unless configured in `Allowed-Origins`.

Automatic reboots should be planned carefully on production servers.

***

# Part 5: Package Cache and Disk Management

APT stores downloaded `.deb` files under:

```text
/var/cache/apt/archives/
```

Inspect cache use:

```bash
du -sh /var/cache/apt/archives
ls -lh /var/cache/apt/archives | head
```

Clean obsolete downloads:

```bash
sudo apt autoclean
```

Remove all cached package files:

```bash
sudo apt clean
```

Review orphaned packages:

```bash
sudo apt autoremove --dry-run
sudo apt autoremove
```

Review installed kernels:

```bash
dpkg -l | grep linux-image
uname -r
```

## Kernel safety warning

Do not blindly remove old kernels. Keep at least one known-good previous kernel. Review `apt autoremove` output carefully before confirming.

A targeted kernel purge should be performed only after confirming:

```bash
uname -r
dpkg -l | grep linux-image
```

Example:

```bash
sudo apt purge linux-image-X.X.XX-XX-generic
```

Replace the version with a kernel you have verified is not currently running and is no longer required.

***

# Part 6: Snap Package Management

Snap is an alternative packaging system. Snaps are container-like, distribution-independent packages that include dependencies and update automatically by default.

Install Snap:

```bash
sudo apt install snapd
systemctl status snapd --no-pager
```

## `snap` commands

### Syntax

```bash
snap [options] COMMAND
```

### Important commands

| Command | Meaning | Example |
|---|---|---|
| `version` | Show Snap version | `snap version` |
| `list` | List installed snaps | `snap list` |
| `info` | Show snap details | `snap info firefox` |
| `search` | Search snaps | `snap search "code editor"` |
| `install` | Install snap | `sudo snap install firefox` |
| `refresh` | Update snaps | `sudo snap refresh` |
| `remove` | Remove snap | `sudo snap remove firefox` |
| `revert` | Revert to previous revision | `sudo snap revert firefox` |
| `changes` | Show recent Snap changes | `snap changes` |
| `services` | Show snap services | `snap services` |
| `enable` | Enable snap service | `sudo snap enable SNAP` |
| `disable` | Disable snap service | `sudo snap disable SNAP` |
| `refresh --channel=CHANNEL` | Change/update channel | `sudo snap refresh --channel=stable firefox` |

### Examples

```bash
snap version
snap list
snap info firefox
snap search "code editor"
sudo snap install firefox
sudo snap refresh
sudo snap refresh firefox
sudo snap remove firefox
snap changes
snap services
sudo snap refresh --channel=stable firefox
```

## APT versus Snap versus source compilation

| Approach | Best for | Updates | Dependency handling | Caution |
|---|---|---|---|---|
| APT/dpkg | Most Debian/Ubuntu server software | Repository-based | Automatic with APT | Repository trust and compatibility |
| Snap | Newer desktop and cross-distribution applications | Automatic by default | Bundled | More disk use; update behavior |
| Source compilation | Custom builds, patches, unavailable versions | Manual | Manual | No automatic security updates |

APT remains the primary package manager for most Debian/Ubuntu server packages.

***

# Part 7: Source Compilation

## When to compile from source

Compile from source when:

- Software is unavailable in repositories.
- A specific version or custom build is required.
- A patch or custom feature is needed.
- You are learning how software is built.

## Risks

- No automatic dependency management.
- No automatic security updates.
- Manual upgrade and removal burden.
- Potential library and version conflicts.
- Files may be installed outside package management.

## Typical workflow

```bash
sudo apt update
sudo apt install build-essential git curl
sudo apt build-dep package_name
```

`apt build-dep` requires source repositories to be enabled on many Ubuntu systems.

Clone and build:

```bash
git clone https://example.com/project.git
cd project
./configure
make
sudo make install
```

### Build tools

| Tool | Purpose |
|---|---|
| `build-essential` | Compiler, build tools, and development essentials |
| `make` | Build automation |
| `gcc` | GNU C compiler |
| `./configure` | Detects system features and generates build configuration |
| `make install` | Installs built files |
| `checkinstall` | Optional tool that can create a package from a source build |

Document every source-built application: version, commit, build flags, install prefix, dependencies, upgrade procedure, and removal procedure.

***

# Hands-On Labs

# Lab 9.1 — Package Lifecycle

## Objective

Install, inspect, update, and remove packages safely.

Use small, safe packages such as:

```text
htop
tree
sl
cowsay
```

## Task

Complete the full package lifecycle for `htop`.

## Solution

Refresh metadata:

```bash
sudo apt update
```

Search:

```bash
apt search htop
```

Show details:

```bash
apt show htop
```

Simulate installation:

```bash
sudo apt install --dry-run htop
```

Install:

```bash
sudo apt install -y htop
```

Verify installation:

```bash
dpkg -s htop
dpkg -L htop
command -v htop
```

Check version and repository:

```bash
apt-cache policy htop
```

Check dependencies:

```bash
apt-cache depends htop
```

Mark manually installed:

```bash
sudo apt-mark manual htop
apt-mark showmanual | grep htop
```

Check upgrade availability:

```bash
apt list --upgradable
```

Upgrade a single package if an update exists:

```bash
sudo apt install --only-upgrade htop
```

Remove:

```bash
sudo apt remove -y htop
dpkg -s htop || echo "Package removed"
```

Purge configuration:

```bash
sudo apt purge -y htop
dpkg -s htop || echo "Package and configuration removed"
```

Clean cache:

```bash
sudo apt autoremove
sudo apt autoclean
```

## Expected-results checklist

- [ ] Package metadata was refreshed.
- [ ] `htop` was installed and inspected.
- [ ] Installed files and dependencies were identified.
- [ ] `remove` and `purge` behavior was distinguished.
- [ ] Package cache was cleaned safely.

***

# Lab 9.2 — Broken Package Recovery

## Objective

Create controlled package/configuration problems and recover safely.

Use a test server or VM snapshot. Do not perform this on production.

## Scenario 1: Interrupted dpkg installation

Create a harmless local package only if you have a trusted test `.deb`. Do not download random packages.

If you have a valid local package:

```bash
sudo dpkg -i package.deb
```

If dependencies fail, observe the error and repair:

```bash
sudo apt --fix-broken install
```

Verify:

```bash
dpkg -C
dpkg -s package_name
```

## Scenario 2: Unconfigured package

Simulate an interrupted configuration state using a safe test package. Then run:

```bash
sudo dpkg --configure -a
```

Verify:

```bash
dpkg -C
apt list --upgradable
```

## Scenario 3: Broken dependency simulation

Install a package with a missing dependency using `dpkg`, only with a trusted test package:

```bash
sudo dpkg -i test-package.deb
```

Observe and repair:

```bash
sudo apt install -f
sudo apt --fix-broken install
```

Review proposed changes before confirming.

Verify:

```bash
dpkg -s package_name
apt-cache policy package_name
```

## Scenario 4: Repository error recovery

Create an intentionally invalid repository file:

```bash
sudo tee /etc/apt/sources.list.d/invalid.list >/dev/null <<'EOF'
deb https://invalid.example.com/ubuntu stable main
EOF
```

Run:

```bash
sudo apt update
```

Expected style of error:

```text
Err: https://invalid.example.com/ubuntu stable InRelease
  Could not resolve host: invalid.example.com
```

Remove the invalid file:

```bash
sudo rm /etc/apt/sources.list.d/invalid.list
```

Refresh:

```bash
sudo apt update
```

Verify:

```bash
apt-cache policy
apt list --upgradable
```

## Expected-results checklist

- [ ] Dependency failures were recognized.
- [ ] Interrupted package configuration was repaired.
- [ ] A bad repository was identified and removed.
- [ ] System package health was verified.

***

# Lab 9.3 — Patch Management Report

## Objective

Create an update and security-patch report for a Linux server.

## Solution

Create the report directory:

```bash
mkdir -p ~/package-lab/reports
cd ~/package-lab
```

Refresh metadata:

```bash
sudo apt update
```

List upgradable packages:

```bash
apt list --upgradable > reports/upgradable-packages.txt
```

Show package policy:

```bash
apt-cache policy nginx > reports/nginx-policy.txt
```

Simulate upgrades:

```bash
sudo apt-get -s upgrade | tee reports/upgrade-simulation.txt
sudo unattended-upgrade --dry-run --debug | tee reports/security-update-dry-run.txt
```

Check reboot requirement:

```bash
cat /var/run/reboot-required 2>/dev/null || echo "No reboot required"
cat /var/run/reboot-required.pkgs 2>/dev/null || true
```

Check unattended-upgrades status:

```bash
systemctl status unattended-upgrades --no-pager
systemctl list-timers | grep apt
```

Generate package inventory:

```bash
dpkg-query -W -f='${Package}\t${Version}\t${Status}\n' > reports/package-inventory.txt
```

Check held packages:

```bash
apt-mark showhold > reports/held-packages.txt
```

Check automatic and manual packages:

```bash
apt-mark showauto > reports/auto-packages.txt
apt-mark showmanual > reports/manual-packages.txt
```

Create a Markdown patch report:

```bash
cat > reports/patch-report.md <<EOF
# Patch Management Report

- Hostname: $(hostname)
- Date: $(date --iso-8601=seconds)
- OS: $(grep PRETTY_NAME /etc/os-release | cut -d= -f2- | tr -d '"')
- Kernel: $(uname -r)
- Installed packages: $(dpkg-query -W -f='${db:Status-Abbrev}' | grep -c '^ii')
- Upgradable packages: $(apt list --upgradable 2>/dev/null | grep -c upgradable)
- Held packages: $(apt-mark showhold | wc -l)
- Reboot required: $(test -f /var/run/reboot-required && echo yes || echo no)

## Recommended maintenance window

Schedule during the approved change window.

## Risk assessment

Review upgradable packages, security advisories, service dependencies, and reboot requirements before applying changes.

## Rollback plan

1. Restore VM snapshot or backup.
2. Reinstall affected packages from trusted repositories.
3. Restore application configuration from backup.
4. Verify service health and logs.
EOF
```

Review:

```bash
less reports/patch-report.md
```

## Expected-results checklist

- [ ] Upgradable package list was generated.
- [ ] Security-update dry run was captured.
- [ ] Reboot requirement was checked.
- [ ] Package inventory was created.
- [ ] Held, automatic, and manual package lists were saved.
- [ ] Markdown patch report includes risk and rollback sections.

***

# Part 8: Real-World Server Use Cases

## Web server maintenance

```bash
nginx -v
sudo apt update
apt list --upgradable | grep nginx
sudo apt install --only-upgrade nginx
sudo nginx -t
sudo systemctl reload nginx
systemctl status nginx --no-pager
journalctl -u nginx -n 50 --no-pager
```

## Database server patching

Before upgrading a database package:

```bash
sudo apt update
apt-cache policy postgresql
apt-cache rdepends postgresql
sudo apt-mark hold postgresql
```

Take an application-approved backup first. Upgrade only during the approved maintenance window, then unhold:

```bash
sudo apt-mark unhold postgresql
sudo apt update
sudo apt install --only-upgrade postgresql
```

## Server provisioning

```bash
sudo apt update
sudo apt install -y curl git vim htop unzip
sudo apt autoclean
dpkg-query -W -f='${Package}\t${Version}\n' > base-packages.txt
```

## Security compliance

```bash
sudo apt update
apt list --upgradable
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
sudo unattended-upgrade --dry-run --debug
```

Generate patch reports regularly and track CVE remediation through your vulnerability-management process.

## Disk-space recovery

```bash
du -sh /var/cache/apt/archives
sudo apt autoclean
sudo apt autoremove --dry-run
sudo apt autoremove
```

Review old kernels carefully before removal.

## Software deployment

```bash
sudo apt install ./application.deb
sudo apt --fix-broken install
dpkg -s application
dpkg -L application
dpkg --verify application
```

***

# Cheat Sheets

## APT command cheat sheet

```bash
sudo apt update
apt search PATTERN
apt search --names-only PATTERN
apt show PACKAGE
apt show -a PACKAGE
apt list --installed
apt list --upgradable
apt list --all-versions PACKAGE

sudo apt install PACKAGE
sudo apt install -y PACKAGE
sudo apt install --dry-run PACKAGE
sudo apt install --no-install-recommends PACKAGE
sudo apt install --reinstall PACKAGE
sudo apt install --fix-broken
sudo apt install ./package.deb

sudo apt remove PACKAGE
sudo apt purge PACKAGE
sudo apt remove --autoremove PACKAGE
sudo apt purge -y PACKAGE

sudo apt upgrade
sudo apt full-upgrade
sudo apt full-upgrade --dry-run
sudo apt install --only-upgrade PACKAGE

sudo apt autoremove --dry-run
sudo apt autoremove
sudo apt autoclean
sudo apt clean
```

## dpkg command cheat sheet

```bash
sudo dpkg -i package.deb
sudo dpkg -r PACKAGE
sudo dpkg -P PACKAGE
dpkg -l
dpkg -l PACKAGE
dpkg -L PACKAGE
dpkg -s PACKAGE
dpkg -S /path/to/file
dpkg -C
sudo dpkg --configure -a
dpkg --verify PACKAGE
dpkg --get-selections
```

## apt-cache command cheat sheet

```bash
apt-cache search PATTERN
apt-cache show PACKAGE
apt-cache policy PACKAGE
apt-cache depends PACKAGE
apt-cache rdepends PACKAGE
apt-cache showpkg PACKAGE
apt-cache stats
apt-cache pkgnames PREFIX
apt-cache unmet
```

## apt-mark command cheat sheet

```bash
apt-mark showmanual
apt-mark showauto
apt-mark showhold
sudo apt-mark hold PACKAGE
sudo apt-mark unhold PACKAGE
sudo apt-mark auto PACKAGE
sudo apt-mark manual PACKAGE
```

## Snap command cheat sheet

```bash
snap version
snap list
snap info SNAP
snap search PATTERN
sudo snap install SNAP
sudo snap refresh
sudo snap refresh SNAP
sudo snap remove SNAP
sudo snap revert SNAP
snap changes
snap services
sudo snap refresh --channel=stable SNAP
```

## Package-state cheat sheet

| State | Meaning |
|---|---|
| `install` | Selected for installation |
| `hold` | Upgrade prevented |
| `deinstall` | Selected for removal |
| `purge` | Package and configuration selected for removal |
| `ii` | Installed successfully |
| `rc` | Removed but configuration remains |
| `hi` | Held and installed |
| `un` | Not installed |

## Repository troubleshooting cheat sheet

```bash
sudo apt update
apt-cache policy
apt-cache policy PACKAGE
ls -l /etc/apt/sources.list
ls -l /etc/apt/sources.list.d/
grep -R --exclude='*.save' -E '^(deb|Types|URIs|Suites|Components|Signed-By)' \
  /etc/apt/sources.list /etc/apt/sources.list.d/
```

***

# Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| `apt update` fails with 404 | Repository suite no longer exists or URL is wrong | Inspect `/etc/apt/sources.list` and `/etc/apt/sources.list.d/`; correct or remove the repository |
| GPG key or `NO_PUBKEY` error | Missing, expired, or incorrectly referenced signing key | Use the vendor’s documented key and `Signed-By` keyring configuration |
| Repository is unsigned or insecure | Missing signing configuration or deliberately insecure source | Do not use `--allow-insecure-repositories` casually; configure trusted signing properly |
| Package cannot be located | Wrong name, repository not enabled, stale cache, wrong release | Run `apt update`; use `apt search`, `apt-cache policy`, and verify repositories |
| Broken dependencies | Interrupted install, mixed repositories, incompatible versions | Use `sudo apt --fix-broken install`; review proposed changes carefully |
| dpkg interrupted installation | Installation was stopped or system lost power | Run `sudo dpkg --configure -a`, then `dpkg -C` |
| Package is held accidentally | `apt-mark hold` was used | Run `apt-mark showhold`, then `sudo apt-mark unhold PACKAGE` |
| `apt autoremove` wants important packages | Package was marked automatic or dependency assumptions changed | Review output; use `apt-mark manual PACKAGE` for packages that must remain |
| Disk full due to APT cache | Cached `.deb` files accumulated | Use `sudo apt autoclean` or `sudo apt clean`; verify disk space before and after |
| Snap will not refresh | Network, snapd service, store, or channel issue | Check `snap changes`, `systemctl status snapd`, and retry with the documented channel |
| Source compilation fails | Missing compiler, headers, libraries, or build dependencies | Install `build-essential`, relevant `-dev` packages, or use `apt build-dep` where available |

***

# Practice Exercises

1. Refresh APT metadata and save the output to a report file.
2. Search for a package using name-only search.
3. Show the installed version, candidate version, and repository priorities for Nginx.
4. Simulate installation of a package before installing it.
5. Install `htop`, inspect its files, then remove and purge it.
6. Find which package owns `/usr/bin/ls`.
7. Hold a package, verify it, then unhold it.
8. Run `apt autoremove --dry-run` and explain each proposed removal.
9. Check whether a reboot is required and identify the packages involved.
10. Generate a package inventory using `dpkg-query`.

***

# Interview Questions

## 1. What is the difference between `apt update` and `apt upgrade`?

`apt update` refreshes repository metadata. `apt upgrade` installs available package upgrades using that metadata.

## 2. What is the difference between `apt remove` and `apt purge`?

`remove` deletes the package binaries and libraries but usually retains system-wide configuration. `purge` removes the package and its system-wide configuration files.

## 3. Why is `apt` generally preferred over `dpkg -i` for repository software?

APT resolves dependencies, downloads packages from repositories, and manages repository metadata. `dpkg` mainly handles local `.deb` files and low-level package operations.

## 4. What is the risk of `apt full-upgrade` on a production server?

It may install or remove packages to resolve dependency changes. This can affect services, configurations, and reboot requirements.

## 5. Why should you avoid random `curl | bash` installers?

They execute remote code with the current user’s privileges, may bypass package verification, and can make upgrades, rollback, and security auditing harder.

***

# Final Capstone Project

## Objective

Prepare an Ubuntu Server VM, configure repositories securely, install and inspect a web server, apply security updates, configure unattended upgrades, manage a package hold, recover from a controlled package issue, and generate a patch-management report.

## 1. Prepare the VM

```bash
sudo apt update
sudo apt install -y curl git vim htop unzip
mkdir -p ~/package-lab/reports
cd ~/package-lab
```

## 2. Review repositories securely

```bash
cat /etc/apt/sources.list
ls -l /etc/apt/sources.list.d/
grep -R --exclude='*.save' -E '^(deb|Types|URIs|Suites|Components|Signed-By)' \
  /etc/apt/sources.list /etc/apt/sources.list.d/
apt-cache policy
```

Remove only unauthorized or invalid repository files after review.

## 3. Install and inspect a web server

```bash
sudo apt install -y nginx
nginx -v
dpkg -s nginx
dpkg -L nginx
apt-cache policy nginx
apt-cache depends nginx
systemctl status nginx --no-pager
curl -I http://127.0.0.1/
```

## 4. Apply security updates

```bash
sudo apt update
apt list --upgradable
sudo apt upgrade
sudo apt install --only-upgrade nginx
sudo systemctl reload nginx
systemctl status nginx --no-pager
```

Check reboot requirement:

```bash
cat /var/run/reboot-required 2>/dev/null || echo "No reboot required"
```

## 5. Configure unattended-upgrades

```bash
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
cat /etc/apt/apt.conf.d/20auto-upgrades
systemctl status unattended-upgrades --no-pager
sudo unattended-upgrade --dry-run --debug
```

## 6. Hold a package during maintenance

```bash
sudo apt-mark hold nginx
apt-mark showhold
apt list --upgradable
```

After the simulated maintenance window:

```bash
sudo apt-mark unhold nginx
sudo apt update
sudo apt install --only-upgrade nginx
```

## 7. Recover from a controlled package issue

Create an invalid repository file:

```bash
sudo tee /etc/apt/sources.list.d/capstone-invalid.list >/dev/null <<'EOF'
deb https://invalid.example.com/ubuntu stable main
EOF
```

Observe the failure:

```bash
sudo apt update
```

Remove it:

```bash
sudo rm /etc/apt/sources.list.d/capstone-invalid.list
sudo apt update
dpkg -C
sudo dpkg --configure -a
```

## 8. Generate the patch-management report

```bash
cd ~/package-lab

sudo apt update
apt list --upgradable > reports/upgradable-packages.txt
apt-cache policy nginx > reports/nginx-policy.txt
sudo apt-get -s upgrade | tee reports/upgrade-simulation.txt
sudo unattended-upgrade --dry-run --debug | tee reports/security-update-dry-run.txt

dpkg-query -W -f='${Package}\t${Version}\t${Status}\n' > reports/package-inventory.txt
apt-mark showhold > reports/held-packages.txt
apt-mark showauto > reports/auto-packages.txt
apt-mark showmanual > reports/manual-packages.txt

cat > reports/capstone-patch-report.md <<EOF
# Capstone Patch Management Report

- Hostname: $(hostname)
- Date: $(date --iso-8601=seconds)
- OS: $(grep PRETTY_NAME /etc/os-release | cut -d= -f2- | tr -d '"')
- Kernel: $(uname -r)
- Installed packages: $(dpkg-query -W -f='${db:Status-Abbrev}' | grep -c '^ii')
- Upgradable packages: $(apt list --upgradable 2>/dev/null | grep -c upgradable)
- Held packages: $(apt-mark showhold | wc -l)
- Reboot required: $(test -f /var/run/reboot-required && echo yes || echo no)

## Maintenance plan

Apply updates during an approved window. Review package removals, service restarts, and reboot requirements before proceeding.

## Rollback plan

1. Restore the VM snapshot or server backup.
2. Reinstall affected packages from trusted repositories.
3. Restore application configuration from backup.
4. Verify package integrity, service health, logs, and application endpoints.

## Reboot procedure

1. Confirm no critical jobs are running.
2. Notify stakeholders.
3. Run \`sudo systemctl reboot\`.
4. Verify boot completion, service status, and application health.
EOF
```

Review the report:

```bash
less reports/capstone-patch-report.md
```

## Expected capstone results

- [ ] Repositories were reviewed and invalid sources removed.
- [ ] Nginx was installed, inspected, and verified.
- [ ] Security updates were checked and applied where appropriate.
- [ ] `unattended-upgrades` was configured and tested with a dry run.
- [ ] A package hold and unhold cycle was completed.
- [ ] A controlled repository error was created and repaired.
- [ ] Package inventory and patch report were generated.
- [ ] Rollback and reboot procedures were documented.
