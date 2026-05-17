# pfSense Firewall, VLAN Segmentation, and WireGuard VPN

**Cert alignment:** CompTIA Network+, CompTIA Security+
**Lab VM:** pfsense (1 GB RAM, 8 GB disk, no VLAN tag - receives raw trunk)
**Last reviewed:** 2026-07

---

## pfSense Architecture in This Lab

pfSense is deployed as a virtual machine on Proxmox with a single network
interface (vmbr0, no VLAN tag). It receives all traffic from the Proxmox
bridge and handles VLAN routing, firewall enforcement, DHCP for VLAN 20 and 99,
and WireGuard VPN termination.

The pfSense VM acts as the default gateway for all three VLANs. All inter-VLAN
traffic must pass through pfSense, which means all inter-VLAN communication
is subject to firewall rule evaluation.

---
