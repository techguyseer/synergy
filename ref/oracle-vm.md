# Oracle VirtualBox & Linux Mint Setup Guide

This guide details how to create an isolated virtual machine environment using **Oracle VM VirtualBox** and **Linux Mint** for hosting **Hermes Agent** and related services within the **Synergy Marketing Ecosystem**.

---

## Part 1: Overview & Prerequisites

Using a dedicated Virtual Machine (VM) creates a secure sandbox environment for autonomous AI agents like Hermes Agent. This protects your host operating system from untrusted file modifications and provides an environment identical to cloud VPS deployments.

### System Requirements
- **Host OS**: Windows, Linux, or Intel-based macOS
- **CPU**: 64-bit x86 processor with hardware virtualization (VT-x/AMD-V) enabled in BIOS/UEFI
- **RAM**: Minimum 8 GB host RAM (allocate 4 GB to the VM)
- **Disk**: 30 GB free SSD storage space

---

## Fast-Track: Automated CLI Setup Script

You can completely automate downloading software/ISO, allocating hardware, creating virtual disks, and attaching ISOs using `VBoxManage` CLI scripts.

### Option A: macOS & Linux Automated Shell Script (`setup-vm.sh`)

Save and run this script in your terminal (or run lines step-by-step):

```bash
#!/usr/bin/env bash
set -e

VM_NAME="Hermes-Agent-Environment"
RAM_MB=4096
CPUS=2
DISK_MB=30720
ISO_DIR="$HOME/Downloads"
ISO_PATH="$ISO_DIR/linuxmint-22-cinnamon-64bit.iso"
ISO_URL="https://mirrors.kernel.org/linuxmint/iso/stable/22/linuxmint-22-cinnamon-64bit.iso"

echo "=== 1. Checking / Installing VirtualBox ==="
if ! command -v VBoxManage &> /dev/null; then
    if [[ "$OSTYPE" == "darwin"* ]]; then
        echo "Installing VirtualBox via Homebrew..."
        brew install --cask virtualbox virtualbox-extension-pack
    else
        echo "Error: VBoxManage not found. Please install VirtualBox for your Linux distro."
        exit 1
    fi
fi

echo "=== 2. Downloading Linux Mint ISO ==="
mkdir -p "$ISO_DIR"
if [ ! -f "$ISO_PATH" ]; then
    echo "Downloading Linux Mint ISO..."
    curl -L -o "$ISO_PATH" "$ISO_URL"
else
    echo "ISO already exists at $ISO_PATH"
fi

echo "=== 3. Creating & Registering VM: $VM_NAME ==="
if VBoxManage showvminfo "$VM_NAME" &>/dev/null; then
    echo "VM '$VM_NAME' already exists."
else
    VBoxManage createvm --name "$VM_NAME" --ostype "Ubuntu_64" --register

    echo "=== 4. Allocating Hardware Specs (4GB RAM, 2 vCPUs) ==="
    VBoxManage modifyvm "$VM_NAME" \
        --memory $RAM_MB \
        --cpus $CPUS \
        --vram 128 \
        --graphicscontroller vmsvga \
        --clipboard-mode bidirectional \
        --draganddrop bidirectional \
        --boot1 dvd --boot2 disk

    echo "=== 5. Creating 30 GB Virtual Disk ==="
    VM_DIR="$HOME/VirtualBox VMs/$VM_NAME"
    mkdir -p "$VM_DIR"
    VDI_PATH="$VM_DIR/$VM_NAME.vdi"
    VBoxManage createmedium disk --filename "$VDI_PATH" --size $DISK_MB --format VDI

    echo "=== 6. Attaching Storage Controllers & ISO ==="
    VBoxManage storagectl "$VM_NAME" --name "SATA Controller" --add sata --controller IntelAhci
    VBoxManage storageattach "$VM_NAME" --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium "$VDI_PATH"
    VBoxManage storageattach "$VM_NAME" --storagectl "SATA Controller" --port 1 --device 0 --type dvddrive --medium "$ISO_PATH"

    echo "=== 7. Starting VM ==="
    VBoxManage startvm "$VM_NAME"
fi
```

### Option B: Windows Automated PowerShell Script

Run in PowerShell:

```powershell
$VMName = "Hermes-Agent-Environment"
$RAM = 4096
$CPUs = 2
$DiskMB = 30720
$IsoDir = "$env:USERPROFILE\Downloads"
$IsoPath = "$IsoDir\linuxmint-22-cinnamon-64bit.iso"
$IsoUrl = "https://mirrors.kernel.org/linuxmint/iso/stable/22/linuxmint-22-cinnamon-64bit.iso"

# 1. Download ISO if missing
if (!(Test-Path $IsoPath)) {
    Write-Host "Downloading Linux Mint ISO..."
    Invoke-WebRequest -Uri $IsoUrl -OutFile $IsoPath
}

# 2. Create VM & Allocate Hardware via VBoxManage
$VBoxCli = "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe"
& $VBoxCli createvm --name $VMName --ostype "Ubuntu_64" --register
& $VBoxCli modifyvm $VMName --memory $RAM --cpus $CPUs --vram 128 --graphicscontroller vmsvga --clipboard-mode bidirectional --draganddrop bidirectional

# 3. Create Disk & Mount ISO
$VmDir = "$env:USERPROFILE\VirtualBox VMs\$VMName"
& $VBoxCli createmedium disk --filename "$VmDir\$VMName.vdi" --size $DiskMB --format VDI
& $VBoxCli storagectl $VMName --name "SATA Controller" --add sata --controller IntelAhci
& $VBoxCli storageattach $VMName --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium "$VmDir\$VMName.vdi"
& $VBoxCli storageattach $VMName --storagectl "SATA Controller" --port 1 --device 0 --type dvddrive --medium $IsoPath

# 4. Launch VM
& $VBoxCli startvm $VMName
```

---

## Part 2: Manual GUI Setup - Downloading Required Software & ISO

1. **Download Oracle VM VirtualBox**:
   - Download VirtualBox installer from [virtualbox.org](https://www.virtualbox.org/).
   - Download the **VirtualBox Extension Pack** (matches your VirtualBox version).
2. **Download Linux Mint ISO**:
   - Download **Linux Mint Cinnamon Edition (64-bit)** ISO from [linuxmint.com](https://linuxmint.com/download.php).

---

## Part 3: Creating and Configuring the Virtual Machine

### 1. Create New Virtual Machine in VirtualBox
1. Launch **VirtualBox** and click **New**.
2. **Name**: `Hermes-Agent-Environment`
3. **Folder**: Choose installation path.
4. **ISO Image**: Browse and select your downloaded `linuxmint-xx-cinnamon-64bit.iso`.
5. Check **Skip Unattended Installation** (recommended for full desktop setup control).
6. Click **Next**.

### 2. Hardware Allocation
- **Base Memory (RAM)**: `4096 MB` (4 GB)
- **Processors**: `2` vCPUs (or 4 if host has 8+ cores)
- Check **Enable EFI** (optional, standard BIOS is also compatible)

### 3. Virtual Hard Disk Allocation
- Select **Create a Virtual Hard Disk Now**.
- **Disk Size**: Set to at least `30.00 GB`.
- Click **Finish**.

---

## Part 4: Installing Linux Mint OS

1. Select `Hermes-Agent-Environment` in VirtualBox and click **Start**.
2. Boot into the live Linux Mint environment and double-click **Install Linux Mint** on the desktop.
3. Select Language, Keyboard Layout, and check **Install multimedia codecs**.
4. Installation Type: Choose **Erase disk and install Linux Mint** (applies only to the virtual disk).
5. Set your location, username, computer name, and password.
6. Complete installation and click **Restart Now**. Remove the virtual ISO when prompted and press **Enter**.

---

## Part 5: Installing VirtualBox Guest Additions & Desktop Extensions

Guest Additions provide shared clipboard, shared folders, automatic display resizing, and hardware acceleration between your host OS and Linux Mint VM.

### 1. Install Dependencies in Linux Mint Terminal
Launch Terminal inside Linux Mint and run:
```bash
sudo apt update
sudo apt install -y build-essential dkms linux-headers-$(uname -r)
```

### 2. Mount Guest Additions CD Image
1. In the VirtualBox window top menu bar, click **Devices > Insert Guest Additions CD image...**
2. In Linux Mint, a prompt will appear asking to run the installer. Click **Run**.
3. Authenticate with your Linux Mint user password.
4. Wait for the terminal compilation to finish ("Press Return to close this window...").

### 3. Enable Shared Clipboard & Drag and Drop
1. In VirtualBox menu bar: **Devices > Shared Clipboard > Bidirectional**.
2. In VirtualBox menu bar: **Devices > Drag and Drop > Bidirectional**.
3. Reboot the VM:
   ```bash
   sudo reboot
   ```

---

## Part 6: Preparing VM for Hermes Agent Deployment

Once Linux Mint is running with Guest Additions, prepare the system for Hermes Agent:

```bash
# Update package repositories
sudo apt update && sudo apt upgrade -y

# Install essential CLI tools & Git
sudo apt install -y curl git jq unzip Python3 python3-pip

# Install GitHub CLI (gh)
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install -y gh

# Install Hermes Agent
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

For complete Hermes Agent configuration instructions, refer to **[Hermes Agent Setup Guide](hermes.md)**.

---

## Common Troubleshooting Guide

| Symptom / Error Message | Probable Cause | Recommended Solution |
| :--- | :--- | :--- |
| Guest Additions build fails with `kernel headers not found` | Linux Mint guest kernel header packages or build tools missing. | Open terminal in Linux Mint and execute `sudo apt update && sudo apt install -y build-essential dkms linux-headers-$(uname -r)` before running Guest Additions installer. |
| Shared clipboard (copy/paste) or drag-and-drop between host and VM fails | VirtualBox menu setting disabled or `VBoxClient` daemon inactive in guest. | Set **Devices > Shared Clipboard > Bidirectional** and **Devices > Drag and Drop > Bidirectional** in VM window menu, then run `VBoxClient-all` or reboot VM. |
| VirtualBox throws `VT-x/AMD-V hardware acceleration is not available` | CPU hardware virtualization disabled in host BIOS/UEFI or Hyper-V conflict on Windows. | Reboot host, enter BIOS/UEFI setup, and enable **Intel VT-x** or **AMD-V**. On Windows hosts, disable Hyper-V feature (`Disable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V`). |
| VM screen resolution locked to low 800x600 resolution | VMSVGA graphics controller driver unmounted or video memory underallocated. | Set **Settings > Display > Video Memory** to `128 MB`, set Graphics Controller to `VMSVGA`, enable 3D Acceleration, and re-run Guest Additions setup inside Linux. |
| Linux Mint guest has no network/internet access | VM Network Adapter mode set to invalid adapter binding or disabled. | Open VM Settings > **Network** > Adapter 1, ensure **Enable Network Adapter** is checked, and set Attached to **NAT** or **Bridged Adapter**. |
| `VBoxManage` command not recognized in host terminal (Windows) | VirtualBox installation path not added to system `PATH` environment variable. | Add `C:\Program Files\Oracle\VirtualBox` to your Windows System `PATH` variable, or call full path `& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe"`. |
| Linux Mint VM reports `Low Disk Space` / 30 GB VDI full | Virtual disk capacity exhausted by cached packages and build logs. | Expand VDI file on host via `VBoxManage modifymedium disk <path.vdi> --resize 51200` (50 GB), then use GParted inside Linux Mint to expand the main ext4 partition. |

---

## Related Documents
- [Overview Page](../overview.md)
- Next Step: [Hermes Agent Installation & Integration](hermes.md)



