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

