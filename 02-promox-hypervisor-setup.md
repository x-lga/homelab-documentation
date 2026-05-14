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

