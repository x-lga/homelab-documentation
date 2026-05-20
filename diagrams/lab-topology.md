# Lab Network Topology - Complete Architecture Diagram

---

## Full Lab Architecture

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                    HOME NETWORK / ISP                                        ║
║   ISP Router: 192.168.1.1/24 — Proxmox host at 192.168.1.200                 ║
╚════════════════════════════════╤═════════════════════════════════════════════╝
                                 │ Physical NIC (enp2s0 → vmbr0)
╔════════════════════════════════▼═════════════════════════════════════════════╗
║                    PROXMOX VE HOST (192.168.1.200)                           ║
║                    vmbr0 — VLAN-aware Linux Bridge                           ║
║                                                                              ║
║  ┌──────────────────────────────────────────────────────────────────────┐    ║
║  │                    pfSense VM (pfsense)                              │    ║
║  │           Attached to vmbr0 (no VLAN tag — receives trunk)           │    ║
║  │                                                                      │    ║
║  │  WAN: 192.168.1.x (DHCP from home router)                            │    ║
║  │  VLAN 10 (WORK):  10.10.10.1/24                                      │    ║
║  │  VLAN 20 (GUEST): 10.10.20.1/24                                      │    ║
║  │  VLAN 99 (MGMT):  10.10.99.1/24                                      │    ║
║  │  WireGuard:       10.10.200.1/24                                     │    ║
║  └────────────┬─────────────────┬──────────────────┬────────────────────┘    ║
║               │ VLAN 10 trunk   │ VLAN 20 trunk    │ VLAN 99 trunk           ║
║               │                 │                  │                         ║
║  ┌────────────▼──────────┐ ┌────▼───────────┐ ┌────▼──────────────────────┐  ║
║  │  VLAN 10 — WORK       │ │ VLAN 20 — GUEST│ │  VLAN 99 — MANAGEMENT     │  ║
║  │  10.10.10.0/24        │ │ 10.10.20.0/24  │ │  10.10.99.0/24            │  ║
║  │                       │ │                │ │                           │  ║
║  │ dc01 (10.10.10.10)    │ │ kali           │ │ splunk-server             │  ║
║  │  Windows Server 2022  │ │ (10.10.20.10   │ │  (10.10.99.10)            │  ║
║  │  AD DS, DNS, DHCP     │ │  DHCP)         │ │  Splunk Free 9.x          │  ║
║  │  AD Connect           │ │  Kali 2024.x   │ │  Log aggregation          │  ║
║  │  Splunk UF            │ │  Pen testing   │ │  SPL dashboards           │  ║
║  │  GPO policy source    │ │  Burp Suite    │ │                           │  ║
║  │                       │ │  Nmap          │ │                           │  ║
║  │ ubuntu-admin          │ │                │ └───────────────────────────┘  ║
║  │ (10.10.10.20)         │ └────────────────┘                                ║
║  │  Ubuntu 22.04         │                                                   ║
║  │  Bash scripting       │                                                   ║
║  │  Nessus scanner src   │                                                   ║
║  │  Splunk UF            │                                                   ║ 
║  │                       │                                                   ║
║  │ win10-client          │                                                   ║
║  │ (10.10.10.50 DHCP)    │                                                   ║
║  │  Windows 10 22H2      │                                                   ║
║  │  Domain-joined        │                                                   ║
║  │  Intune enrolled      │                                                   ║
║  └───────────────────────┘                                                   ║ 
╚══════════════════════════════════════════════════════════════════════════════╝
                         │
                         │ AD Connect Sync (HTTPS/443 to Azure)
                         │ Azure Monitor Agent (Arc-connected)
                         ▼
╔═══════════════════════════════════════════════════════════════════════════════╗
║                    MICROSOFT AZURE                                            ║
║  Entra ID Tenant: contosodemo.onmicrosoft.com                                 ║
║  Users synced from contoso.local: jane.mwangi, brian.otieno, testuser1        ║
║                                                                               ║
║  ┌─────────────────────────────────────────────────────────────────────────┐  ║
║  │  Resource Group: rg-hybrid-lab (UK South region)                        │  ║
║  │                                                                         │  ║
║  │  VNet: 10.20.0.0/16                                                     │  ║
║  │    subnet-servers: 10.20.1.0/24 → vm-win-server (10.20.1.4, no PIP)     │  ║
║  │    subnet-linux:   10.20.2.0/24 → vm-ubuntu (10.20.2.4, no PIP)         │  ║
║  │    AzureBastionSubnet: 10.20.254.0/27 → Azure Bastion (PIP: bastion-pip)│  ║
║  │                                                                         │  ║
║  │  NSG: nsg-subnet-servers (deny-all + explicit allows)                   │  ║
║  │  RBAC: junioradmin=Contributor@RG, auditor=Reader@Sub, vmop=VMContrib   │  ║
║  │  Log Analytics Workspace: law-hybrid-lab                                │  ║
║  │  Alert Rule: CPU>80% → ag-it-alerts (email)                             │  ║
║  │  Microsoft Intune (M365 Dev) → device compliance → win10-client         │  ║
║  └─────────────────────────────────────────────────────────────────────────┘  ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

---
