# Proxmox VE Hypervisor Setup

**Cert alignment:** CompTIA A+ (Virtualisation)
**Last reviewed:** 2026-07

---

## Why Proxmox

Proxmox VE (Virtual Environment) is a free, open-source Type 1 hypervisor based
on KVM and LXC. It provides a web-based management interface, built-in backup and
snapshot capabilities, and handles all the VM lifecycle management needed for this
lab. It runs directly on bare metal - not inside another OS - which gives VMs
near-native performance.

Proxmox was chosen over alternatives for these specific reasons:
- **Free:** No licence cost, no feature restrictions, full enterprise capability
- **Web UI:** Complete management from a browser - no separate management station needed
- **Snapshots:** Ability to snapshot before breaking things and revert after is invaluable
  in a learning lab
- **Network flexibility:** Linux-bridge-based networking allows VLAN tagging without
  physical managed switches - the entire segmented network runs in software

---

## Installation

1. Download Proxmox VE ISO from `proxmox.com/downloads`
2. Write to a USB drive: `dd if=proxmox-ve_8.x.iso of=/dev/sdX bs=1M` (Linux/Mac)
   or use Rufus (Windows) in DD mode
3. Boot the target machine from USB
4. Follow the installer:
   - Target disk: select your SSD
   - Country/timezone: configure
   - Admin password and email: set strong admin password
   - Management interface: select your physical NIC
   - Hostname: `pve.contoso.local` (or your chosen FQDN)
   - IP: assign a static IP on your home network (e.g., 192.168.1.200/24)
5. Remove USB after install and reboot
6. Access the web UI: `https://192.168.1.200:8006`
   Accept the self-signed certificate warning

---

## Initial Configuration

### Update Proxmox (no subscription required)

The default Proxmox repository requires a paid subscription. For a home lab,
use the no-subscription community repository:

```bash
# SSH to Proxmox host or open Shell in the web UI
# Remove the enterprise repo
rm /etc/apt/sources.list.d/pve-enterprise.list

# Add the no-subscription community repo
cat >> /etc/apt/sources.list << 'EOF'
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription
EOF

# Remove the Ceph enterprise repo (not needed for this lab)
rm /etc/apt/sources.list.d/ceph.list 2>/dev/null

# Update and upgrade
apt update && apt dist-upgrade -y
```

### Create the Network Bridge Structure

For VLANs to work, Proxmox needs a single bridge with VLAN awareness enabled.
This allows a single physical NIC to carry all VLAN-tagged traffic.

```
Proxmox Web UI → Node → Network → Create → Linux Bridge

Name       : vmbr0
IP address : 192.168.1.200/24  (Proxmox management IP — on your home network)
Gateway    : 192.168.1.1
Bridge ports: enp2s0 (or your physical NIC name — check with: ip link show)
VLAN aware : YES ← this is critical
Comment    : Main bridge — VLAN-aware for lab segmentation

→ Apply Configuration
```

**Why VLAN-aware bridge:** Without this setting, Proxmox passes all traffic
untagged and pfSense cannot segment it into VLANs. With VLAN-aware enabled,
pfSense can tag outbound traffic with 802.1Q VLAN IDs and Proxmox forwards
the tags to the correct VMs.


### Configure Storage

For the lab, local-lvm (the default thin-provisioned LVM storage) is used for
VM disks:

```
Proxmox Web UI → Datacenter → Storage → local-lvm
This is already configured for VM disk images.

For ISO uploads (Windows Server, Kali, Ubuntu):
Proxmox Web UI → Node → local → ISO Images → Upload
```

---

## Creating Virtual Machines

### General VM creation process

```
Proxmox Web UI → Create VM

General:
  Node : pve
  VM ID: [auto-assigned or set manually, e.g., 100, 101, 102...]
  Name : [e.g., dc01, pfsense, ubuntu-admin]

OS:
  Storage : local
  ISO     : [select uploaded ISO]

System:
  BIOS    : SeaBIOS (for most VMs)
              OVMF (UEFI) for VMs requiring Secure Boot (e.g., modern Windows 11)
  Machine : q35 (recommended for newer OSes)

Disks:
  Bus     : VirtIO SCSI (best performance on Linux)
             SATA (more compatible for Windows if VirtIO drivers not pre-installed)
  Size    : [as per VM inventory above]

CPU:
  Sockets : 1
  Cores   : [as per VM inventory above]
  Type    : host (passes through CPU flags — required for nested virtualisation)

Memory:
  MiB     : [as per VM inventory above — e.g., 4096 for dc01]
  Ballooning: Disable for Windows VMs (ballooning causes instability on Windows)

Network:
  Bridge  : vmbr0
  VLAN Tag: [leave blank for pfSense — pfSense manages its own VLAN tags]
             [set to VLAN ID for other VMs — e.g., 10 for dc01, 99 for splunk-server]
  Model   : VirtIO (best performance)
```

### VM VLAN assignment in Proxmox

Set the VLAN tag on each VM's network interface to place it in the correct segment:

```
Proxmox Web UI → VM → Hardware → Network Device → Edit
  VLAN Tag: 10    (for Work segment VMs - dc01, win10-client, ubuntu-admin)
  VLAN Tag: 20    (for Isolated/Kali VM)
  VLAN Tag: 99    (for Management/Splunk VM)
  VLAN Tag: (none) (for pfSense - it receives raw trunk traffic and handles tagging itself)

```

### Snapshots - use before breaking things

```
Proxmox Web UI → VM → Snapshots → Take Snapshot

Name       : before-ad-connect-install
Description: Clean state before installing AD Connect on dc01

→ Take Snapshot
```

**When to snapshot:**
- Before installing any major software
- Before making network configuration changes
- Before testing anything that might break AD, DNS, or DHCP
- After getting each lab component to a known-good working state

To revert:
```
Proxmox Web UI → VM → Snapshots → [snapshot name] → Rollback
```

---

