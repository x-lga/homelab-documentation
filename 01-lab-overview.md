# Home Lab Overview

## Purpose

This lab was built to develop, practise, and validate IT and cloud skills across
every domain covered by CompTIA A+, Network+, Security+, Microsoft AZ-900, AZ-104,
and ITIL 4 Foundation - in a working environment that simulates what small and
medium enterprises actually run, rather than in isolated textbook exercises.

The design principle was: if a cert exam objective says "configure X" or
"troubleshoot Y", there should be a corresponding lab component where X is actually
configured and where breaking Y and fixing it has actually happened. Every
technology documented here has been deployed, tested, broken at least once, and
repaired from that breakage.

This is not a tutorial. It is a build record.

---

## Lab Design Philosophy

**Why simulate an enterprise rather than just install individual components:**
Individual components in isolation teach you how to configure them. Components
integrated together teach you how they interact, how they fail, and how to diagnose
which layer of a multi-component stack has broken. Every production IT environment
is an integrated stack - you almost never see pure, isolated components.

**Why pfSense with VLANs instead of a simpler network:**
Any network that does not practice segmentation is not teaching real network security.
VLAN separation between corporate, guest, and management traffic is standard in
every enterprise environment worth supporting. The pfSense firewall also forces
rule-writing practice - understanding that firewall rules have order, that blocking
wrong traffic has consequences, and that diagnosing "why can't this device reach
this resource" requires understanding the firewall rule chain.

**Why Azure hybrid rather than cloud-only:**
Cloud-only Entra ID is a simplified architecture. Most organisations have
on-premises infrastructure they cannot fully migrate, and they connect it to
Azure using AD Connect. Understanding the hybrid sync - and what happens when
it breaks - is more practically valuable than understanding only the cloud side.

**Why Splunk Free instead of just reviewing logs manually:**
Manual log review is an L1 skill. SIEM correlation - writing SPL queries,
building dashboards, detecting patterns across multiple hosts - is what
differentiates a security-aware engineer from one who only checks individual
logs. Splunk Free imposes constraints (no scheduled searches, single CPU) that
force you to learn efficient query design.

**Why Nessus instead of ignoring vulnerability management:**
Vulnerability scanning is part of every enterprise security programme. Understanding
what Nessus reports, how to interpret CVSS scores, and what to do with the findings
is a practical skill that extends Security+ theory into operational practice.

---

## Hardware and Hypervisor

| Component | Specification |
|-----------|-------------|
| Hypervisor | Proxmox VE 8.x |
| Host platform | Dedicated spare PC (alternatively: high-spec laptop) |
| Minimum RAM | 16 GB (lab runs comfortably at this level with careful VM scheduling) |
| Recommended RAM | 32 GB (allows all VMs to run simultaneously without swapping) |
| Storage | 512 GB SSD for VM images (NVMe preferred - significant performance impact) |
| Network | Single physical NIC (VLANs handled by Proxmox Linux bridges and pfSense) |

**Cost consideration:**
The entire lab was built using:
- Proxmox VE: free (open source)
- Windows Server 2022: free evaluation licence (180 days, renewable)
- pfSense: free (open source)
- Splunk Free: free tier (up to 500MB/day ingestion)
- Nessus Essentials: free tier (up to 16 hosts)
- Microsoft Intune: included in M365 Developer E5 Sandbox (free 90-day, renewable)
- Azure: free trial ($200 credit) + M365 Developer Sandbox

Total hardware cost: $0 if using existing hardware. A refurbished small-form-factor
PC with 32 GB RAM and a 512 GB SSD typically costs $150-300 and is sufficient.

---

## Complete VM Inventory

| VM Name | OS | Role | vCPUs | RAM | Disk | VLAN | IP Address |
|---------|-----|------|-------|-----|------|------|-----------|
| dc01 | Windows Server 2022 Standard | AD DS, DNS, DHCP, AD Connect, Splunk UF | 2 | 4 GB | 60 GB | VLAN 10 | 10.10.10.10 |
| win10-client | Windows 10 22H2 | Domain-joined client, Intune enrolled | 2 | 2 GB | 50 GB | VLAN 10 | 10.10.10.50 (DHCP) |
| pfsense | pfSense 2.7.x | Firewall, VLAN routing, WireGuard VPN, DHCP | 1 | 1 GB | 8 GB | All VLANs | 10.10.x.1 per VLAN |
| ubuntu-admin | Ubuntu 22.04 LTS | Bash scripting lab, Splunk Forwarder, Nessus scanner source | 2 | 2 GB | 30 GB | VLAN 10 | 10.10.10.20 |
| splunk-server | Ubuntu 22.04 LTS | Splunk Free SIEM, log aggregation and dashboards | 2 | 4 GB | 50 GB | VLAN 99 | 10.10.99.10 |
| kali | Kali Linux 2024.x | Penetration testing, Burp Suite, Nmap, vulnerability verification | 2 | 4 GB | 40 GB | VLAN 20 | 10.10.20.10 (DHCP) |

**Total resource consumption when all VMs are running simultaneously:**
- vCPUs: 11 (Proxmox host handles scheduling)
- RAM: 17 GB (tight at 16 GB host — recommend 32 GB or run subsets of VMs)
- Storage: ~238 GB allocated across all disks

**VM scheduling recommendation for 16 GB hosts:**
Run dc01 + pfsense + ubuntu-admin + splunk-server as the always-on core (11 GB RAM).
Start win10-client or kali when specifically needed.

---

## Network Architecture Summary

| VLAN | Name | Subnet | Gateway | Purpose |
|------|------|--------|---------|---------|
| VLAN 10 | Work | 10.10.10.0/24 | 10.10.10.1 | Corporate endpoints - dc01, win10-client, ubuntu-admin |
| VLAN 20 | Guest / Isolated | 10.10.20.0/24 | 10.10.20.1 | Kali Linux lab - isolated from corporate |
| VLAN 99 | Management | 10.10.99.0/24 | 10.10.99.1 | Splunk server - accessible only from admin IPs |

**Inter-VLAN connectivity rules (enforced by pfSense):**
- VLAN 10 → Internet: Allowed
- VLAN 10 → VLAN 20: Blocked (corporate cannot reach isolated lab)
- VLAN 10 → VLAN 99: Allowed from admin IPs only (10.10.10.10 and 10.10.10.20)
- VLAN 20 → Internet: Allowed (HTTP/HTTPS only)
- VLAN 20 → VLAN 10: Blocked (Kali cannot reach corporate)
- VLAN 20 → VLAN 99: Blocked (no SIEM access from isolated network)
- VLAN 99 → All: Allowed (management must reach everything for log forwarding)

---






