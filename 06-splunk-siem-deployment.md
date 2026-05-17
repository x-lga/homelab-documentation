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

