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

