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

## Installation and Initial Configuration

1. Download pfSense CE ISO from `pfsense.org/download`
2. Create pfSense VM in Proxmox (1 CPU, 1 GB RAM, 8 GB disk, no VLAN tag on NIC)
3. Boot from ISO and follow the installer (accept defaults)
4. After first boot, assign interfaces:
   - WAN: em0 (the Proxmox bridge interface)
   - Note: In this lab, the WAN interface of pfSense connects to the home router
     via an untagged VLAN (VLAN 1 / native), while VLANs 10, 20, 99 are tagged
5. Access the pfSense web GUI from the Proxmox console initially, then from a
   browser once network is configured

---

## VLAN Interface Configuration

In pfSense, VLANs are created as sub-interfaces of the physical WAN interface:

```
pfSense Web UI → Interfaces → Assignments → VLANs → Add

VLAN 10:
  Parent Interface : em0 (or vtnet0 — your physical interface name)
  VLAN Tag         : 10
  Description      : Work

VLAN 20:
  Parent Interface : em0
  VLAN Tag         : 20
  Description      : Guest-Isolated

VLAN 99:
  Parent Interface : em0
  VLAN Tag         : 99
  Description      : Management
```

Then assign each VLAN as a pfSense interface:
```
pfSense Web UI → Interfaces → Assignments →
  Add VLAN10  → Assign → rename to OPT1 → rename to WORK
  Add VLAN20  → Assign → rename to OPT2 → rename to GUEST
  Add VLAN99  → Assign → rename to OPT3 → rename to MGMT
```

Configure each interface:
```
Interfaces → WORK:
  Enable      : Checked
  IPv4 Config : Static IPv4
  IPv4 Address: 10.10.10.1 / 24
  Description : VLAN10-Work

Interfaces → GUEST:
  Enable      : Checked
  IPv4 Config : Static IPv4
  IPv4 Address: 10.10.20.1 / 24
  Description : VLAN20-Guest

Interfaces → MGMT:
  Enable      : Checked
  IPv4 Config : Static IPv4
  IPv4 Address: 10.10.99.1 / 24
  Description : VLAN99-Management
```

---
