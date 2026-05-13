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





