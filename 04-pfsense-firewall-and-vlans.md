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

## DHCP Server Configuration

pfSense serves DHCP for VLAN 20 and VLAN 99. VLAN 10 uses the Windows DC.

```
pfSense Web UI → Services → DHCP Server

GUEST interface (VLAN 20):
  Enable        : Checked
  Range         : 10.10.20.100 – 10.10.20.200
  DNS servers   : 8.8.8.8 (public DNS - no access to internal DNS by design)
  Default gateway: 10.10.20.1

MGMT interface (VLAN 99):
  Enable        : Checked
  Range         : 10.10.99.50 – 10.10.99.100
  DNS servers   : 10.10.10.10 (internal DNS - management segment needs AD resolution)
  Default gateway: 10.10.99.1
```

---

## Firewall Rules

Firewall rules in pfSense are configured per interface (the source interface
of the traffic). Rules are evaluated top-to-bottom - first match wins.

### VLAN 10 (Work) - Interface WORK

| Priority | Action | Protocol | Source | Destination | Port | Description |
|---------|--------|---------|--------|------------|------|-------------|
| 1 | Allow | TCP/UDP | WORK net | 10.10.10.10 | 53 | DNS to DC |
| 2 | Allow | Any | WORK net | 10.10.10.0/24 | Any | Intra-VLAN communication |
| 3 | Allow | TCP | 10.10.10.10, 10.10.10.20 | 10.10.99.0/24 | Any | Admin IPs to management VLAN |
| 4 | Allow | Any | WORK net | WAN | Any | Internet access |
| 5 | Block | Any | WORK net | 10.10.20.0/24 | Any | Block corporate → isolated |
| 6 | Block | Any | WORK net | Any | Any | Default deny |


### VLAN 20 (Guest / Isolated) - Interface GUEST

| Priority | Action | Protocol | Source | Destination | Port | Description |
|---------|--------|---------|--------|------------|------|-------------|
| 1 | Allow | TCP | GUEST net | WAN | 80, 443 | Internet web access only |
| 2 | Allow | TCP/UDP | GUEST net | 8.8.8.8, 1.1.1.1 | 53 | DNS to public resolvers |
| 3 | Block | Any | GUEST net | 10.10.10.0/24 | Any | No access to corporate |
| 4 | Block | Any | GUEST net | 10.10.99.0/24 | Any | No access to management |
| 5 | Block | Any | GUEST net | GUEST net | Any | No intra-VLAN (IoT isolation) |
| 6 | Block | Any | GUEST net | Any | Any | Default deny |


### VLAN 99 (Management) - Interface MGMT

| Priority | Action | Protocol | Source | Destination | Port | Description |
|---------|--------|---------|--------|------------|------|-------------|
| 1 | Allow | Any | MGMT net | Any | Any | Management can reach everything |
| 2 | Block | Any | Any | MGMT net | Any | All other traffic blocked into MGMT |

**Note on management VLAN access:** The rule above (Block Any → MGMT) applies on
interfaces other than MGMT. On the MGMT interface itself, Rule 1 allows full egress.
Access into MGMT from WORK is controlled by the WORK rule (Allow admin IPs only).

---

## WireGuard VPN Configuration

WireGuard provides secure remote admin access to the lab without exposing any
management interface to the public internet.

```
pfSense Web UI → VPN → WireGuard → Add Tunnel

Tunnel configuration:
  Listen Port    : 51820
  Interface Keys : Generate (click Generate to create server key pair)
  Interface Addresses: 10.10.200.1/24

Peer (remote admin machine):
  Description    : Remote-Admin-Laptop
  Public Key     : [generate on client machine and paste here]
  Allowed IPs    : 10.10.200.2/32
  Endpoint       : [leave blank if client is behind NAT]
```

**Generate client key pair (on the admin machine):**
```bash
# Install WireGuard on Linux client
sudo apt install wireguard

# Generate key pair
wg genkey | tee privatekey | wg pubkey > publickey
cat publickey    # paste this into pfSense as the peer public key
cat privatekey   # use this in the client config file
```

**Client configuration file (`/etc/wireguard/wg0.conf`):**
```ini
[Interface]
PrivateKey = [client private key]
Address    = 10.10.200.2/32
DNS        = 10.10.10.10

[Peer]
PublicKey  = [pfSense server public key - from pfSense WireGuard settings]
AllowedIPs = 10.10.10.0/24, 10.10.99.0/24
Endpoint   = [your home public IP]:51820
PersistentKeepalive = 25
```

**Start the VPN:**
```bash
sudo wg-quick up wg0
# Verify connection
sudo wg show
```

**pfSense firewall rule for WireGuard interface:**
```
pfSense → Firewall → Rules → WireGuard:
  Allow: Source 10.10.200.0/24 → Destination 10.10.10.0/24 (VLAN 10 resources)
  Allow: Source 10.10.200.0/24 → Destination 10.10.99.0/24 (Management)
  Block: Source 10.10.200.0/24 → Destination 10.10.20.0/24 (No access to isolated)
```

---

## Verifying VLAN Isolation

Test that VLANs are correctly isolated after configuration:

**Test 1: VLAN 20 cannot reach VLAN 10**
```bash
# From kali (VLAN 20 — 10.10.20.10):
ping 10.10.10.10        # Should FAIL (pfSense blocks VLAN20 → VLAN10)
ping 10.10.10.1         # Should FAIL (gateway unreachable from VLAN 20 to VLAN 10)
ping 8.8.8.8            # Should SUCCEED (internet allowed from VLAN 20)
```

**Test 2: VLAN 10 non-admin cannot reach VLAN 99**
```powershell
# From win10-client (VLAN 10 — 10.10.10.50, non-admin IP):
Test-NetConnection -ComputerName 10.10.99.10 -Port 8000  # Should FAIL
Test-NetConnection -ComputerName 10.10.10.10 -Port 53    # Should SUCCEED (DNS)
```

**Test 3: Admin IP can reach VLAN 99**
```powershell
# From dc01 (10.10.10.10 - admin IP in VLAN 10):
Test-NetConnection -ComputerName 10.10.99.10 -Port 8000  # Should SUCCEED (Splunk)
```

**Test 4: VLAN 20 devices cannot communicate with each other**
```bash
# From kali, try to reach another VLAN 20 device:
# (Add a second device to VLAN 20, e.g., a second Ubuntu VM for testing)
ping [second VLAN 20 device IP]    # Should FAIL (VLAN 20 → VLAN 20 blocked)
```


---

