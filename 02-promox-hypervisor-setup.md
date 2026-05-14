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
