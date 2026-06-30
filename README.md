# Proxmox Windows 11 MECM LAB Automation

This project automates the creation of a fully configured Windows 11 Development VM on Proxmox. It handles VM creation, unattended Windows installation, debloating, and the installation of essential development tools (VS2022, VS Code, Git, OpenSSH).

## ✨ Features

- **Fully Automated VM Creation**: One-command deployment from Proxmox shell
- **Unattended Windows 11 Installation**: No manual intervention required
- **VirtIO Drivers**: Automatically loads storage and network drivers during setup
- **Debloated Windows**: Telemetry, bloatware, and search suggestions disabled
- **Pre-installed Development Environment**:
  - Hyper V
  - Visual Studio Code
  - Git
- **SSH Access Enabled**: OpenSSH server pre-configured and ready
- **Resource Optimized**: 32GB RAM and 16 CPU cores by default (customizable)

## 🚀 Quick Start


```bash
bash -c "$(wget -qLO - https://raw.githubusercontent.com/prathikgopal/proxmox-win11-autoinstall/main/install.sh)" -- -i 3000 -n "Win11-HyperV" -p "P@ssw0rd"
```
## ⚙️ Configuration

### Command Line Arguments
The `Proxmox script.sh` accepts the following flags to customize the deployment without editing files:

| Flag | Description | Default |
|------|-------------|---------|
| `-i` | **VM ID**: The unique ID for the new VM. | `2000` |
| `-n` | **VM Name**: The name label for the VM. | `Win11-HyperV` |
| `-m` | **Memory (MB)**: RAM allocated to the VM. | `32768` (32GB) |
| `-c` | **Cores**: Number of CPU cores allocated. | `16` |
| `-p` | **Password**: Local `Admin` user password. | `P@ssw0rd` |

**Example:**
```bash
./Proxmox\ script.sh -i 4000 -n "Build-Server" -m 32768 -c 8 -p "P@ssw0rd"
```

### Script Variables (Advanced)
Open `Proxmox script.sh` to edit these variables if your Proxmox environment differs from the defaults:

*   **`DISK_STORAGE`**: Storage ID for the VM disk (Default: `local-lvm`).
*   **`ISO_STORAGE_ID`**: Storage ID for ISOs (Default: `nfs`).
*   **`ISO_PATH_ROOT`**: Filesystem path to your ISO storage (`/mnt/pve/SAS1/template/iso` Default: `/mnt/pve/nfs/template/iso`).
*   **`WIN_ISO`**: Filename of your Windows 11 ISO.
*   **`VIRTIO_ISO`**: Filename of your VirtIO drivers ISO.

### Unattended Installation (`autounattend.xml`)
The answer file handles the Windows setup. Key configurations include:

*   **User**: Creates a local user named `Admin`.
*   **Debloat**: Automatically disables Telemetry, "Consumer Features" (Candy Crush, etc.), and Search Suggestions.
*   **Software**: Automatically installs the following via Chocolatey:
    *   Git
    *   Visual Studio Code
    * Hyper-V Role
    *   OpenSSH Server (Enabled & Firewall Rule Added)

## 📋 Prerequisites

### 1. Proxmox VE
Tested on Proxmox VE 8.x. Should work on 7.x as well.

### 2. Required ISOs
You must upload these ISOs to your Proxmox ISO storage before running the script:

**Windows 11 ISO:**
- **Auto-Download:** The script will ask for a download link if the ISO is missing.
- **Get Link:** Go to [Microsoft](https://www.microsoft.com/software-download/windows11), select "Windows 11 (multi-edition ISO)", choose language, and copy the "64-bit Download" link.
- **Manual Upload:** Alternatively, download it yourself and upload to Proxmox.
- Filename in script: `Win11_25H2_x64v2_auto.iso` (or whatever the download provides)

**VirtIO Drivers ISO:**
- Download from [Fedora Project](https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md)
- Latest stable release: [virtio-win-0.1.285.iso](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso)
- Filename in script: `virtio-win-0.1.285.iso`

### 3. Tools
The script requires `genisoimage` to generate the answer file ISO:
```bash
apt install genisoimage
```
*(The install.sh wrapper handles this automatically)*

## 🎯 Post-Installation

After the VM finishes installing (approximately 30-60 minutes depending on your hardware):

### Default Credentials
- **Username:** `Admin`
- **Password:** As configured via `-p` flag or default `P@ssw0rd`

### Access Methods
- **Console:** Via Proxmox web interface
- **RDP:** Port 3389 (use Remote Desktop)
  ```bash
  mstsc /v:<VM-IP>
  ```
- **SSH:** Port 22
  ```bash
  ssh Admin@<VM-IP>
  ```

### Verify Installation
1. Check that Hyper-V, VS Code, and Git are installed
2. Verify OpenSSH is running:
   ```powershell
   Get-Service sshd
   ```
3. Confirm debloat settings via Registry Editor

## ⚠️ Troubleshooting

### "File not found" errors
Ensure `ISO_PATH_ROOT` in the script matches the actual path on your Proxmox host where your ISO storage is mounted. Check with:
```bash
pvesm path nfs:iso/Win11_25H2_x64v2_auto.iso
```

### Network issues
- The VM requires internet access on `vmbr0` during first login to download packages
- Ensure your Proxmox bridge has DHCP or configure static IP in `autounattend.xml`

### VirtIO drivers not loading
- Verify the VirtIO ISO is correctly attached to the VM
- The answer file checks both `E:\` and `F:\` drive letters automatically
- If needed, manually browse to the VirtIO ISO during Windows setup

## 📝 License

This project is open source and available under the MIT License.

## 🤝 Contributing

Contributions, issues, and feature requests are welcome! Feel free to check the issues page or submit a pull request.

