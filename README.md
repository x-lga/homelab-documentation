# homelab-lab-documentation

Complete documentation of an enterprise-grade home lab used to develop and validate IT and cloud support skills. Built on Proxmox with 6 VMs simulating an enterprise environment with AD domain, VLAN-segmented network, Azure hybrid connectivity, SIEM monitoring, vulnerability scanning, and endpoint management.

---

## Lab Components Documented

| Component | Document |
|-----------|---------|
| Lab overview and VM inventory | `01-lab-overview.md` |
| Proxmox hypervisor setup | `02-proxmox-setup.md` |
| Windows Server 2022 - AD/DNS/DHCP/GPO | `03-windows-server-ad-dns-dhcp.md` |
| pfSense firewall - VLAN/WireGuard VPN | `04-pfsense-firewall-vlan.md` |
| Azure hybrid - AD Connect to Entra ID | `05-azure-hybrid-connection.md` |
| Splunk Free SIEM deployment | `06-splunk-siem-deployment.md` |
| Nessus Essentials vulnerability scanning | `07-nessus-vulnerability-scanning.md` |
| Microsoft Intune endpoint management | `08-intune-endpoint-management.md` |
| Full network topology diagram | `diagrams/lab-topology.md` |

---

## Skills Validated

- **Virtualisation:** Proxmox VM lifecycle, networking, storage
- **AD/DNS/DHCP:** Domain promotion, OU structure, GPO configuration, DHCP scopes
- **Network Security:** pfSense VLAN, inter-VLAN firewall rules, WireGuard VPN, 802.1Q
- **Azure Hybrid:** AD Connect, Entra ID sync, hybrid SSO
- **SIEM:** Splunk deployment, Windows event log forwarding, SPL queries, dashboards
- **Vulnerability Management:** Nessus scan configuration, CVSS review, remediation documentation
- **Endpoint Management:** Intune MDM enrollment, compliance policies, app deployment

---

## Outcome

This lab was built from scratch using free and trial-tier tools to simulate the exact environments found in MSP, corporate IT, NOC, and cloud support roles. Every competency covered in CompTIA A+, Network+, Security+, AZ-900, AZ-104, and ITIL 4 Foundation has a corresponding lab exercise documented here.