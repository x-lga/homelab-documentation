# Splunk Free SIEM Deployment

**Cert alignment:** CompTIA Security+
**Lab VM:** splunk-server (Ubuntu 22.04, VLAN 99 - 10.10.99.10)
**Forwarder:** Splunk Universal Forwarder on dc01 (VLAN 10 - 10.10.10.10)
**Last reviewed:** 2026-07

---

## Why Splunk on VLAN 99 (Management)

The Splunk server is on the Management VLAN rather than the Work VLAN for two reasons:

**Security:** The SIEM receives logs from all VLANs. If Splunk were on VLAN 10
(Work), VLAN 20 devices could not forward logs to it (VLAN 20 → VLAN 10 is blocked).
Placing Splunk on VLAN 99 (Management) and allowing all VLANs to forward to VLAN 99
means log collection is not blocked by the VLAN isolation policy.

**Access control:** The Splunk web interface (port 8000) should only be accessible
by administrators. On VLAN 99, access is restricted by pfSense rules — only the
admin IPs in VLAN 10 can reach VLAN 99. This means regular users on win10-client
cannot browse to the Splunk dashboard, but the IT admin on dc01 can.

---

## Splunk Free Installation on Ubuntu 22.04

```bash
# On splunk-server (connect via Bastion or console)

# Update system
sudo apt update && sudo apt upgrade -y

# Download Splunk Free (register at splunk.com — free download)
# Copy the download URL from your browser after accepting the licence
wget -O splunk.deb 'https://download.splunk.com/products/splunk/releases/9.x.x/linux/splunk-9.x.x-linux-2.6-amd64.deb'

# Install
sudo dpkg -i splunk.deb

# Start Splunk and accept licence
sudo /opt/splunk/bin/splunk start --accept-license --answer-yes

# Set admin password when prompted (save this — you need it to access the web UI)

# Configure Splunk to start on boot
sudo /opt/splunk/bin/splunk enable boot-start

# Open firewall for Splunk web UI (port 8000) and receiving (port 9997)
sudo ufw allow from 10.10.10.0/24 to any port 8000
sudo ufw allow from 10.10.10.0/24 to any port 9997
sudo ufw allow from 10.10.99.0/24 to any port 8000
sudo ufw allow from 10.10.99.0/24 to any port 9997
```

Access Splunk web: `http://10.10.99.10:8000`

---

## Configure Splunk to Receive Data

```
Splunk Web UI → Settings → Forwarding and Receiving →
  Configure Receiving → Add New:
  Listen on port: 9997

This tells Splunk to accept data from Universal Forwarders on port 9997.
```

---

