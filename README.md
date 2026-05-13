# homelab-documentation

Complete build documentation for an enterprise-grade home lab designed to develop
and validate IT and cloud skills across CompTIA A+, Network+, Security+, Microsoft
AZ-900, AZ-104, and ITIL 4 Foundation.

The lab runs on Proxmox VE with six VMs simulating a realistic small enterprise
environment: Windows Server 2022 with Active Directory, pfSense firewall with
three-VLAN segmentation and WireGuard VPN, Azure hybrid identity via AD Connect,
Splunk Free SIEM with Windows event log forwarding, Nessus Essentials vulnerability
scanning, and Microsoft Intune endpoint management.

**This repository is the evidence that every other repo in this portfolio is backed
by real hands-on experience in a working lab, not just guided tutorials.**

---

## What makes this lab enterprise-grade

Most home labs run a single VM with a basic AD install. This lab was designed to
simulate the integrated architecture of a real small enterprise:

**Network segmentation:** Three VLANs (Work/Guest/Management) with pfSense enforcing
firewall rules between them. Guest devices cannot reach corporate resources. IoT
devices cannot communicate with each other. Management segment is accessible only
from designated admin IPs. This is not a flat network - it is a defence-in-depth
network architecture.

**Hybrid identity:** On-premises AD synced to Azure Entra ID via AD Connect with
Password Hash Sync. This is the most common enterprise Azure architecture — the one
that requires understanding both sides of the identity boundary, not just cloud-native
deployments.

**Defence in depth:** Splunk SIEM receives Windows Security event logs. Nessus
scans internal hosts for vulnerabilities. Intune enforces device compliance.
pfSense enforces network segmentation. GPOs enforce workstation security baseline.
Five independent layers of security visibility and control - each aligned to a
different Security+ or AZ-104 domain.

**Everything documented:** Every deployment step, every configuration decision,
every architectural choice, and - most importantly - every issue that went wrong
and how it was diagnosed and fixed.

---

## Repository Contents

| File | Purpose |
|------|---------|
| `01-lab-overview.md` | Purpose, design philosophy, hardware specs, VM inventory, network summary, cost breakdown |
| `02-proxmox-hypervisor-setup.md` | Proxmox installation, community repo setup, VLAN-aware bridge, VM creation guide, snapshot strategy, auto-shutdown |
| `03-windows-server-ad-dns-dhcp.md` | AD DS promotion with full PowerShell, OU structure, lab user accounts, DNS verification, DHCP with relay context, GPO configuration, domain join |
| `04-pfsense-firewall-and-vlans.md` | VLAN interface creation, DHCP per VLAN, full firewall rule tables with justification per rule, WireGuard VPN config, isolation verification tests |
| `05-azure-hybrid-connection.md` | AD Connect lab configuration, sync monitoring PowerShell, health troubleshooting steps |
| `06-splunk-siem-deployment.md` | Splunk Free install on Ubuntu, Universal Forwarder on Windows with inputs.conf and outputs.conf, VLAN 99 placement rationale, dashboard SPL queries |
| `07-nessus-vulnerability-scanning.md` | Nessus Essentials install, scan configuration, CVSS interpretation table, common lab findings, re-scan workflow, export documentation |
| `08-intune-endpoint-management.md` | Lab Intune configuration, compliance policy settings table, Conditional Access integration explanation |
| `09-what-i-broke-and-fixed.md` | Eight real issues: DNS service startup type, pfSense blocking AD Connect, DHCP relay missing, Splunk missing outputs.conf, Nessus firewall blocking scan, GPO SYSVOL permissions after snapshot rollback, hybrid DNS misunderstanding, Kali missing NAT rule |
| `10-cert-to-lab-mapping.md` | Every A+, Net+, Sec+, AZ-900, AZ-104, ITIL 4 objective mapped to a specific lab component |
| `diagrams/lab-topology.md` | Full ASCII architecture diagram showing all VMs, VLANs, IP addresses, Azure components, and traffic flow examples |
| `diagrams/vlan-firewall-rules.md` | Complete firewall rule tables for all four interfaces (WORK, GUEST, MGMT, WireGuard) with rule-by-rule justification |
| `diagrams/ad-connect-sync-flow.md` | AD Connect architecture diagram showing sync engine, connectors, metaverse, what happens each cycle, and lab sync scope |

---

## Skills validated across the full lab build

| Domain | Skills practiced |
|--------|-----------------|
| Virtualisation (A+) | Proxmox VM lifecycle, snapshots, VLAN-aware bridge, storage types |
| Networking (Net+) | VLAN design, inter-VLAN routing, DHCP relay, firewall rules, VPN, OSI-layer troubleshooting |
| Security (Sec+) | Defence-in-depth architecture, SIEM deployment, vulnerability scanning, incident response playbooks, network segmentation |
| Cloud (AZ-900/104) | Azure hybrid identity, VNet/NSG, RBAC, Azure Monitor + KQL, Intune |
| Process (ITIL 4) | Incident management, problem management, change management, event management |
| Scripting | PowerShell AD automation, Bash system administration |
| Real troubleshooting | 8 documented real issues - DNS, firewall, DHCP, SIEM, scanning, GPO, DNS, NAT |

---



