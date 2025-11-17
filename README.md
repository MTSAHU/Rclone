

## 💾 Rclone Google Drive Auto-Mount Configuration

This repository provides the guide and scripts to establish a resilient, **FUSE-based Google Drive mount**, configured for **automatic startup on Linux systems**.

-----

### 📋 Prerequisites

Before starting the setup, ensure the following are available:

  * **Operating System:** Linux (Arch, Ubuntu, Debian, etc.).
  * **Permissions:** User has `sudo` access for package installation.
  * **Software:** Rclone and FUSE support must be operational.

-----

### 📄 Comprehensive Setup Guide

The complete, step-by-step technical documentation, including all required commands and configuration prompts, is located in the dedicated file:

| Document | Description | Focus |
| :--- | :--- | :--- |
| **[`Rclone_config.md`](Rclone_config.md)** | **Full Technical Setup Guide** (Installation, `rclone config`, Scripting, Auto-Start). Applicable to all Linux distros. | **Actionable Commands & Flags** |

-----

### 🚀 Setup Flow Overview

The process involves four key stages:

1.  **Preparation:** Install Rclone and ensure FUSE support.
2.  **Remote Definition:** Run `rclone config` to establish the **`Gdrive`** remote connection using **OAuth**.
3.  **Scripting:** Create the executable mount script (`rclone_mount_gdrive.sh`) incorporating **FUSE flags** (`--vfs-cache-mode writes`).
4.  **Persistence:** Set up a **`.desktop` entry** to run the script automatically upon graphical login.

### 📂 Key Configuration Files

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| **`rclone_mount_gdrive.sh`** | `~/myscripts/` | Primary executable script for FUSE mounting. |
| **`gdrive.desktop`** | `~/.config/autostart/` | Triggers the mount script upon user login. |
| **`rclone.conf`** | `~/.config/rclone/` | Stores the encrypted Google Drive credentials. **(Ignored via .gitignore)** |

-----

This `README.md` is now ready to be committed and pushed to your **`rclone-configs`** repository\!