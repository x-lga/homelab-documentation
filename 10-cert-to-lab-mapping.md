# Certification to Lab Component Mapping

This document maps every major certification exam objective covered in this lab
to the specific component, configuration, or exercise where that skill was practised.
Use this as a study reference - if a topic area needs more practice, the
corresponding lab component is listed here.

---

## CompTIA A+ Core 1 (220-1101)

| Exam Objective | Lab Component | Where to Find It |
|---------------|--------------|-----------------|
| 1.1 - Mobile device hardware | (out of scope for this lab) | - |
| 1.2 - Display components | (out of scope for this lab) | - |
| 3.1 - Storage types and characteristics | Proxmox VM disk configuration | 02-proxmox-hypervisor-setup.md |
| 3.2 - Motherboard, CPU, RAM specs | Proxmox VM hardware settings | 02-proxmox-hypervisor-setup.md |
| 3.4 - Virtualisation concepts | Proxmox hypervisor deployment | 02-proxmox-hypervisor-setup.md |
| 3.6 - Cloud computing concepts | Azure Entra ID, Intune, Monitor | 05-azure-hybrid-connection.md |
| 4.1 - Network types and topologies | VLAN 10/20/99 design | 04-pfsense-firewall-and-vlans.md |
| 4.2 - Network protocols and ports | pfSense firewall rule ports | 04-pfsense-firewall-and-vlans.md |
| 5.1 - Printer troubleshooting | Clear-PrintQueue.ps1 | Repo #6 scripts |


## CompTIA A+ Core 2 (220-1102)

| Exam Objective | Lab Component | Where to Find It |
|---------------|--------------|-----------------|
| 1.1 - Windows command line tools | PowerShell scripts throughout | Repo #6 scripts |
| 1.3 - Windows features and tools | AD DS, DHCP, DNS, GPO | 03-windows-server-ad-dns-dhcp.md |
| 1.6 - Windows Security Settings | GPO security baseline | 03-windows-server-ad-dns-dhcp.md |
| 2.1 - Active Directory | User creation, OU structure, domain join | 03-windows-server-ad-dns-dhcp.md |
| 2.2 - Scripting for automation | PowerShell AD scripts | Repo #1 scripts |
| 2.6 - Common security threats | Phishing playbook, malware response | Repo #3 playbooks |
| 4.1 - Troubleshooting methodology | All runbooks and playbooks | Repos #2, #3, #5 |
| 4.2 - Troubleshoot Windows | Event log analysis, service restart | Repo #6 scripts |

---

## CompTIA Network+ (N10-009)

| Exam Objective | Lab Component | Where to Find It |
|---------------|--------------|-----------------|
| 1.1 - OSI model | NOC runbook procedures | Repo #2 runbook |
| 1.2 - Network protocols | pfSense firewall rules (ports, protocols) | 04-pfsense-firewall-and-vlans.md |
| 1.4 - IP addressing and subnetting | VLAN subnets, DHCP scope design | 04-pfsense-firewall-and-vlans.md |
| 1.5 - Routing concepts | pfSense inter-VLAN routing | 04-pfsense-firewall-and-vlans.md |
| 1.6 - Ethernet and switching | VLAN 802.1Q tagging in Proxmox | 02-proxmox-hypervisor-setup.md |
| 2.1 - Network hardware devices | pfSense as firewall/router | 04-pfsense-firewall-and-vlans.md |
| 2.3 - Virtual networking | Proxmox Linux bridges, VLANs | 02-proxmox-hypervisor-setup.md |
| 3.1 - Network services | AD-integrated DNS, DHCP | 03-windows-server-ad-dns-dhcp.md |
| 3.4 - VPN technologies | WireGuard VPN on pfSense | 04-pfsense-firewall-and-vlans.md |
| 4.1 - Network troubleshooting | NOC runbook, OSI-layered triage | Repo #2 runbook |
| 4.3 - Network connectivity troubleshooting | ping, tracert, nslookup, Test-NetConnection | Repo #6 scripts |

---

