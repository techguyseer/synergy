# UTM Virtualization Setup Guide for macOS

This guide details how to install and configure **UTM** — a free and open-source virtual machine host for macOS — to run an isolated **Linux Mint** or **Ubuntu** environment for hosting **Hermes Agent** within the **Synergy Marketing Ecosystem**.

---

## Part 1: Overview & Why UTM for macOS

UTM is a lightweight, high-performance virtual machine host built natively for macOS using Apple's Hypervisor.framework and QEMU.

### Key Features
- **Apple Silicon Native**: Native ARM64 hypervisor acceleration on M1/M2/M3/M4 Macs.
- **Intel Mac Compatible**: Full x86_64 virtualization support.
- **Free & Open Source**: No subscription fees or proprietary license restrictions.
- **Spice Guest Tools**: Integrated clipboard sharing, directory sharing, and resolution scaling.

---

## Part 2: Installing UTM & Downloading Linux ISO

### 1. Download UTM for macOS
- Download the free DMG installer from [mac.getutm.app](https://mac.getutm.app/).
- Drag `UTM.app` into your macOS `/Applications` folder.

### 2. Download Linux ISO Image
- **Apple Silicon (M1/M2/M3/M4 Macs)**: Download **Ubuntu Server / Desktop ARM64** or **Debian ARM64** ISO.
- **Intel Macs**: Download **Linux Mint Cinnamon 64-bit** ISO from [linuxmint.com](https://linuxmint.com/download.php).

---

## Part 3: Creating a Virtual Machine in UTM

1. Launch **UTM** on macOS and click **Create a New Virtual Machine** (`+`).
2. Select **Virtualize** (recommended for native speed on Apple Silicon/Intel).
3. Select **Linux**.
4. Click **Browse** under *Boot ISO Image* and select your downloaded Linux ISO.
5. Click **Continue**.

### Hardware Allocation Settings
- **Memory (RAM)**: Set to `4096 MB` (4 GB).
- **CPU Cores**: Set to `4` cores (or leave default).
- Click **Continue**.

### Storage Allocation
- **Size**: Set virtual disk size to at least `30 GB`.
- Click **Continue**.

### Shared Directory (Optional)
- Select a folder on your Mac (e.g., `~/Downloads` or a dedicated workspace folder) to mount as a shared drive inside the Linux VM.
- Click **Save**.

---

## Part 4: OS Installation & SPICE Guest Agent Setup

1. Click the **Play** button on your new VM inside UTM.
2. Complete the standard Linux installation wizard (Language, User Account, Password, Storage partitioning).
3. After installation completes, unmount the ISO in UTM (**Drive settings > Clear ISO**) and reboot the VM.

### Installing SPICE Guest Agent (Shared Clipboard & Resolution Scaling)
Inside the Linux VM terminal, install the SPICE guest tools:

```bash
# Update repositories
sudo apt update

# Install SPICE agent and QEMU guest agent
sudo apt install -y spice-vdagent qemu-guest-agent

# Enable and start services
sudo systemctl enable --now spice-vdagent
sudo systemctl enable --now qemu-guest-agent
```

Reboot the VM to activate clipboard sharing and dynamic screen resolution scaling:
```bash
sudo reboot
```

---

## Part 5: Installing Hermes Agent & GitHub CLI inside UTM

Once your UTM Linux guest VM is operational, open the terminal and install the required tools for Hermes Agent:

```bash
# Install core dependencies
sudo apt update && sudo apt install -y curl git jq unzip build-essential

# Install GitHub CLI (gh)
type -p curl >/dev/null || (sudo apt update && sudo apt install curl -y)
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh -y

# Install Hermes Agent
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

For complete Hermes Agent configuration details, see **[Hermes Agent Setup Guide](hermes.md)**.

---

## Common Troubleshooting Guide

| Symptom / Error Message | Probable Cause | Recommended Solution |
| :--- | :--- | :--- |
| UTM displays a black screen or freezes during Linux guest boot | Display card driver mismatch with macOS host GPU or Emulation mode selected. | In UTM VM settings > **Display**, set emulation to `virtio-gpu-pci` or `virtio-ramfb`. Under **System**, ensure **Virtualize** is selected. |
| Shared clipboard or dynamic display resolution scaling fails | SPICE guest agent tools (`spice-vdagent`) missing or service disabled in Linux VM. | Open Linux terminal, run `sudo apt install -y spice-vdagent qemu-guest-agent`, enable daemon (`sudo systemctl enable --now spice-vdagent`), and reboot. |
| Shared directory (macOS folder) does not mount inside Linux guest | SPICE WebDAV folder sharing daemon (`spice-webdavd`) not installed inside guest. | Run `sudo apt install -y spice-webdavd` in Linux VM, and configure UTM shared directory path under VM settings > **Sharing**. |
| High CPU usage and fan noise on macOS host while VM is idle | Excessive CPU core allocation or CPU power management features disabled. | Reduce VM allocation to 2–4 CPU cores in UTM settings, enable main memory ballooning, and set CPU model to **Default**. |
| Linux ISO boot error `Exec format error` / kernel panics on launch | CPU architecture mismatch (e.g. running x86_64 ISO on Apple Silicon Mac without emulation). | Download **ARM64** Linux ISOs (Ubuntu ARM64 / Debian ARM64) for M1/M2/M3/M4 Macs. Use x86_64 ISOs exclusively on Intel Macs. |
| Linux guest VM cannot connect to local host Ollama instance | UTM Virtual Network isolates container/VM network traffic from host `localhost`. | Find host IP on macOS (`ifconfig en0`), or set network mode to **Bridged (Advanced)** in UTM settings to access host services directly. |
| Linux VM reports `No space left on device` inside UTM | Initial virtual disk allocation size (30 GB) depleted by packages and logs. | Open UTM VM settings > **Drives**, expand virtual disk size (e.g. 50 GB), then use `gparted` or `resize2fs` inside Linux to expand partition. |

---

## Related Documents
- [Overview Page](../overview.md)
- Next Step: [Hermes Agent Setup Guide](hermes.md)



