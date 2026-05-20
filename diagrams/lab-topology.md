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

## Traffic Flow Examples

**Example 1: win10-client resolves dc01.contoso.local**
```
win10-client → DNS query to 10.10.10.10 (VLAN 10 → VLAN 10 — allowed by pfSense)
dc01 DNS responds with 10.10.10.10
win10-client connects to dc01 on port 389 (LDAP — VLAN 10 → VLAN 10 — allowed)
```

**Example 2: win10-client tries to reach kali (VLAN 20) - should be blocked**
```
win10-client sends packet to 10.10.20.10 (VLAN 10 source)
pfSense WORK interface rules: Allow WORK → WAN, Block WORK → 10.10.20.0/24
pfSense evaluates: source VLAN 10, destination VLAN 20 → Rule 5 (Block) matches
Packet dropped - firewall log records the blocked flow
win10-client receives no response
```

**Example 3: dc01 forwards Windows Security events to Splunk**
```
SplunkForwarder on dc01 (10.10.10.10) opens TCP connection to 10.10.99.10:9997
pfSense WORK interface rules: Allow admin IPs to VLAN 99 (Rule 3 - dc01 is admin IP)
Connection established - Windows Security events stream to Splunk
```

**Example 4: Kali accesses internet for package updates**
```
kali (10.10.20.10) sends HTTP request to external IP
pfSense GUEST interface rules: Rule 1 — Allow GUEST → WAN TCP 80,443 — matches
Outbound NAT translates 10.10.20.10 → pfSense WAN IP
Packet reaches internet — response returns via NAT
```

---

## Component Connectivity Matrix

| From → To | dc01 | win10-client | ubuntu-admin | kali | splunk-server | Internet |
|-----------|:----:|:------------:|:------------:|:----:|:-------------:|:--------:|
| dc01 | ✓ | ✓ | ✓ | ✗ | ✓ (admin) | ✓ |
| win10-client | ✓ | ✓ | ✓ | ✗ | ✗ | ✓ |
| ubuntu-admin | ✓ | ✓ | ✓ | ✗ | ✓ (admin) | ✓ |
| kali | ✗ | ✗ | ✗ | ✓ (same VLAN)* | ✗ | ✓ (web only) |
| splunk-server | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

*kali → kali (intra-VLAN): blocked by the VLAN 20 rule that prevents IoT-style lateral movement


---




