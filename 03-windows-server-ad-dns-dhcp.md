# Windows Server 2022 - Active Directory, DNS, DHCP, and Group Policy

**Cert alignment:** CompTIA A+, CompTIA Network+, CompTIA Security+
**Lab VM:** dc01 (10.10.10.10, VLAN 10)
**Last reviewed:** 2026-07

---

## Installation Notes

Windows Server 2022 Standard Evaluation is available free for 180 days from
Microsoft's Evaluation Centre. The evaluation licence can be reset by running
`slmgr /rearm` for additional evaluation periods during lab use.

After installing Windows Server 2022 in the Proxmox VM:
- Install VirtIO guest drivers for best disk/network performance
  (download from `fedorapeople.org/groups/virt/virtio-win/direct-downloads`)
- Set static IP: `10.10.10.10 / 255.255.255.0 / GW: 10.10.10.1 / DNS: 127.0.0.1`
- Set hostname to `dc01` before promoting to domain controller

---
