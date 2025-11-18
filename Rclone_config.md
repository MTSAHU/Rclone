
## 🛠️ Rclone Google Drive Auto-Mount Guide

This document details the installation, configuration, and setup of a persistent **FUSE-based Google Drive mount** using **Rclone** on Linux systems. The core commands are universal, but package commands are tailored for **Arch Linux**.

-----

### 1\. Installation

Update your system and install the `rclone` package.

  * *(Note: For **Debian/Ubuntu**, use `sudo apt install rclone`)*

<!-- end list -->

```bash
# System Update (Arch Linux)
sudo pacman -Syu

# Install Rclone (Arch Linux)
sudo pacman -S rclone
```

-----

### 2\. Remote Configuration (`rclone config`)

Run the interactive utility to set up the Google Drive remote, named `Gdrive`.

```bash
rclone config
```

| Prompt | Input | Rationale / Note |
| :--- | :--- | :--- |
| **New Remote?** | `n` | Create a new configuration entry. |
| **Remote Name** | `Gdrive` | The name used when mounting (e.g., `Gdrive:`). |
| **Storage Type** | `22` | Identifies the service as Google Drive. |
| **client\_id/secret** | **[Enter]** | Uses Rclone's built-in client credentials. |
| **scope** | `1` | Selects full read/write access (`drive`). |
| **service\_account\_file** | **[Enter]** | Leave blank for standard interactive OAuth flow. |
| **Edit advanced config?** | `n` | Skip advanced settings (default). |
| **Use web browser?** | `y` **or** `n` | Use `y` for desktop/local machine; `n` for headless (VPS/Server). |
| **Authentication** | **Follow Browser/Link** | Complete the Google OAuth flow. *(This step generates the sensitive token stored in `rclone.conf`)* |
| **Shared Drive?** | `n` **or** `y` | Set to `y` if mounting a Google Shared Drive (Team Drive). |
| **Keep remote?** | `y` | Accept and save the remote configuration. |
| **Quit** | `q` | Exit the `rclone config` utility. |

-----

### 3\. FUSE Mount Script

We will create the mount point and an executable script with robust flags for reliable mounting.

  * **Crucial:** Replace `/home/amits/Gdrive` with your system's **absolute path**.

#### A. Prepare Directories

```bash
# 1. Create the local directory where the drive will be mounted
mkdir -p ~/Gdrive

# 2. Create the script execution directory
mkdir -p ~/myscripts
```

#### B. Create the Executable Script

```bash
nano ~/myscripts/rclone_mount_gdrive.sh
```

**Content for `rclone_mount_gdrive.sh`:**

```bash
#!/bin/bash

# Configuration Variables
REMOTE_NAME="Gdrive"
MOUNT_POINT="/home/amits/Gdrive" # <-- !! REPLACE WITH YOUR ABSOLUTE PATH !!

# Gracefully unmount if a previous instance exists
fusermount -u "$MOUNT_POINT" 2>/dev/null

# Execute the Rclone FUSE mount in the background (&)
# Flags are set for performance and resilience.
rclone mount "$REMOTE_NAME": "$MOUNT_POINT" \
    --vfs-cache-mode writes \
    --allow-non-empty \
    --dir-cache-time 72h \
    --poll-interval 15s \
    &
```

**Mount Flag Rationale:**

| Flag | Description |
| :--- | :--- |
| **`--vfs-cache-mode writes`** | Recommended for general use; handles temporary file data, improving reliability for write operations. |
| **`--allow-non-empty`** | Permits mounting even if the directory contains files (e.g., hidden desktop configuration files). |
| **`--dir-cache-time 72h`** | Increases the duration for which directory listings are cached, reducing API calls and speeding up browsing. |
| **`--poll-interval 15s`** | Sets how often Rclone checks the remote for file changes, ensuring local synchronization is timely. |

**Save** (`CTRL+O`) and **Exit** (`CTRL+X`).

#### C. Permissions and Test

```bash
# Make the script executable
chmod +x ~/myscripts/rclone_mount_gdrive.sh

# Test the script manually
~/myscripts/rclone_mount_gdrive.sh

# Verify the mount status (should list the FUSE mount point)
findmnt -l -t fuse
```

-----

### 4\. Persistent Auto-Start

Create a Desktop Entry (`.desktop`) file to automatically run the script when the graphical session starts.

```bash
mkdir -p ~/.config/autostart
nano ~/.config/autostart/gdrive.desktop
```

**Content for `gdrive.desktop`:**

```desktop
[Desktop Entry]
Type=Application
# Exec path must be absolute to ensure proper startup
Exec=/home/amits/myscripts/rclone_mount_gdrive.sh 
Hidden=false
NoDisplay=false
X-GNOME-Autostart-enabled=true
Name=Mount Google Drive (Rclone)
Comment=Automount Google Drive using rclone and FUSE at user login.
```

**Save** (`CTRL+O`) and **Exit** (`CTRL+X`).

-----

### 5\. Unmounting and Cleanup

To stop the mount and cleanly release the resources, use the following command:

```bash
# Unmount the Google Drive remote
fusermount -u /home/amits/Gdrive
```

*(Replace the path with your exact mount point if different).*

+++++++------------------++++++++++++++++++++++++++------------++++++++++++++++++++++++++++++++++++

You want the mount to start **automatically at boot** but **stop at shutdown** without any manual commands. Here's the simple setup:

## Setup for Auto-Mount at Boot (Without Lingering)

**Step 1:** Create the service file:

```bash
nano ~/.config/systemd/user/rclone_gdrive.service
```

**Step 2:** Add this content:

```ini
[Unit]
Description=Rclone Mount Google Drive
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/rclone mount Gdrive: /home/amits/Gdrive --vfs-cache-mode writes
ExecStop=/bin/fusermount -u /home/amits/Gdrive
Restart=always
RestartSec=10

[Install]
WantedBy=graphical-session.target
```

**Key part:** `WantedBy=graphical-session.target` means: start this service when your KDE Plasma desktop session starts.[1]

**Step 3:** Enable and start it:

```bash
systemctl --user daemon-reload
systemctl --user enable rclone_gdrive
systemctl --user start rclone_gdrive
```

**That's it. Now:**
- ✅ When you **start your laptop** → mount automatically loads
- ✅ When you **shutdown** → mount stops (no lingering)
- ✅ **No bash commands needed** to use the mount
- ✅ No background resource consumption when laptop is off[1]

Done. Just login to your KDE desktop and the mount will be ready automatically.[1]

[1](https://forum.rclone.org/t/rclone-mount-w-systemd-when-user-logs-in-unmounts-logout/15101)