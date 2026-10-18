# StorageGuard
StorageGuard is a free, open-source Linux storage-access control utility designed to help protect against malicious scripts on USB storage devices and certain forms of USB device identity spoofing.

StorageGuard is distributed under the GNU General Public License v2.0 (GPL-2.0). The license is included with every installation in LICENSE.txt, located in the StorageGuard installation directory at /usr/local/src/StorageGuard/LICENSE.txt.

If your installation does not include this file, uninstall StorageGuard and obtain a legitimate copy from the official repository:

https://github.com/TheDungeonDeveloper/StorageGuard/

Installation

You can install StorageGuard form a bash terminal with the following Command:

git clone https://github.com/TheDungeonDeveloper/StorageGuard.git
cd StorageGuard
sudo bash install.sh

or you may use the apt-image package from the releases page

View the license
To display or validate the GNU2.0 license from the Bash terminal, run:
cat /usr/local/src/StorageGuard/LICENSE.txt
or navigate to the directory yourself

If after install the license file is missing please uninstall StorageGuard 
and install directly form this repository

File Hash Signatures

Hash Algorithm   Repository ZIP      StorageGuard.deb

MD5              Pending             Pending

SHA-256          Pending             Pending

SHA-512          Pending             Pending

StorageGuard is a work in progress Utility and is still in development
check the releases page to see if we have an update

StorageGuard uses a persistent whitelist to manage trusted storage devices and their authorized partitions. StorageGuard includes a graphical interface, command-line tools, authorization helpers, automatic mounting, and a login mount helper.

## 1. Application Components

| Component | Purpose |
|---|---|
| GTK4 GUI | Displays connected drives and manages authorization. |
| `sguard` CLI | Manages devices and displays storage status. |
| `sg-check-device` | Checks whether a device and partition are authorized. |
| `sg-set-policy` | Changes storage authorization policies. |
| Login mount helper | Checks and mounts authorized partitions at login. |
| Hot-plug watcher | Detects newly connected partitions and mounts authorized ones. |
| Whitelist | Stores trusted device identities and partition policies. |
| Installer and package | Installs, configures, and upgrades StorageGuard. |

A future GRUB plugin is planned to help prevent boot-level UID spoofing. 
It will be distributed separately or bundled in a separate StorageGuard 
installation package When and if it gets made.

## 2. Command-Line Interface

The CLI is installed at `/usr/local/bin/sguard`. All commands require root privileges and use `sudo`.

### Initialize StorageGuard

```bash
sudo sguard initialize
```

Detects the root filesystem's partition and parent drive, then authorizes the root partition for automatic mounting. It confirms required changes and rejects unsupported storage layouts or conflicting policies.

### List storage devices

```bash
sudo sguard devices
```

Displays connected drives, device identifiers, whitelist status, and mount points.

### List whitelist entries

```bash
sudo sguard whitelist list
```

Displays registered drives and their partition policies. `ALL (legacy entry)` indicates an older entry without explicit partition restrictions.

### Add a drive

```bash
sudo sguard whitelist add /dev/sdX
```

Registers a drive and authorizes all its partitions.

To authorize only selected partitions:

```bash
sudo sguard whitelist add /dev/sdX --partitions /dev/sdX1 /dev/sdX2
```

To authorize a single partition:

```bash
sudo sguard whitelist add /dev/sdX1
```

Specifying a partition registers its parent drive and authorizes only that partition.

### Set authorized partitions

```bash
sudo sguard whitelist set-partitions /dev/sdX --partitions /dev/sdX1 /dev/sdX2
```

Restricts an existing drive to the specified partitions.

To authorize only one partition:

```bash
sudo sguard whitelist set-partitions /dev/sdX1
```

Partitions must belong to the selected drive and have a PARTUUID. At least one partition is required.

### Remove a drive

```bash
sudo sguard whitelist remove /dev/sdX
```

Removes a drive from the whitelist after confirmation. This command accepts a whole-drive path, not an individual partition. The active system drive cannot be removed.

### Command notes

- Replace example paths with actual device paths from `sudo sguard devices`.
- Partition policies use PARTUUID identifiers rather than temporary device names.
- Adding a drive without `--partitions` authorizes all its partitions.
- Existing drives must use `set-partitions` to change their policies.
- Whitelist changes require confirmation before saving.

## 3. Authorization Model

StorageGuard stores its whitelist at:

`/etc/storageguard/whitelist.tsv`

The file is owned by root and protected by restrictive permissions.

| Status | Meaning |
|---|---|
| Allowed | All relevant partitions are authorized. |
| Disallow | The drive is restricted from automatic mounting. |
| Partial | Only selected partitions are authorized. |
| Unknown | Authorization could not be verified; the GUI disables the control. |

The system drive is registered during initialization and protected against accidental policy changes in the GUI. The CLI also prevents its removal.

Privileged authorization checks and policy changes use dedicated helpers with narrowly scoped sudo permissions.

## 4. Runtime Behavior

StorageGuard uses separate processes rather than a single continuously running daemon.

### GUI

The GUI lists connected storage devices, checks their authorization status, and applies policy changes through the privileged helper. The system drive's policy control remains disabled.

### Login mount helper

At login, the helper checks connected storage devices and attempts to mount authorized partitions with supported filesystems.

### Hot-plug watcher

The watcher detects newly connected storage partitions through Linux udev events. It runs independently of the GUI and starts automatically through the desktop session's autostart configuration.

For each detected partition, it:

1. Identifies the parent drive.
2. Checks authorization using `sg-check-device`.
3. Skips unauthorized partitions, unrecognized filesystems, and partitions already mounted.
4. Attempts to mount authorized partitions using `udisksctl mount`.
5. Records results and errors in the StorageGuard log.

The watcher is installed at `/usr/local/sbin/storageguard-watch.sh`.

Its autostart entry is:

`~/.config/autostart/storageguard-hotplug.desktop`

The watcher is intended to remain running throughout the desktop session.

### Logging

The login mount helper and hot-plug watcher write to:

`${XDG_STATE_HOME:-$HOME/.local/state}/storageguard/login-mount.log`

The log records authorization decisions, skipped partitions, mount attempts, and failures.

### Persistent configuration

StorageGuard keeps its whitelist separate from executable files so application upgrades can preserve existing device policies.

## 5. Security Limitations

StorageGuard helps reduce the risk of unauthorized storage being automatically mounted, including certain forms of USB device identity spoofing.

However, device identifiers can be spoofed, and a whitelist does not prove that a device is safe. The watcher checks authorization and attempts mounting, but it does not independently prevent every unauthorized mount.

Other applications, privileged commands, or alternative mounting mechanisms may bypass the watcher. Consistent system-wide enforcement requires additional system integration to block unauthorized mount operations.

StorageGuard is a supplementary security measure, not a complete defense against malicious USB devices.

## 6. Installation and Distribution

StorageGuard will be distributed in two formats:

- **Network installer:** Downloads and installs a versioned release from the official source repository.
- **Debian package (`.deb`):** Installs StorageGuard through Linux Mint's package-management tools.

The installed source tree will reside at:

`/usr/local/src/StorageGuard/`

Executables and privileged helpers will be installed in their designated system directories. A desktop entry will add StorageGuard to the Applications menu without creating a desktop shortcut.

Upgrades must preserve the existing whitelist and validate privileged helper permissions and sudoers configuration.
