# VLAN Firewall Rules - Complete Reference

This document is the single source of truth for all pfSense firewall rules
in the lab. It supplements the topology diagram with the exact rule logic.

---

## Rule Processing Reminder

pfSense evaluates rules on the **ingress interface** (the interface where traffic enters pfSense).
Rules are evaluated **top to bottom - first match wins**.
Default action if no rule matches: **BLOCK** (implicit deny-all).

---

## WORK Interface Rules (VLAN 10 - 10.10.10.0/24)

Rules applied to traffic entering pfSense from the VLAN 10 segment:

| # | Action | Protocol | Source | Destination | Port | Purpose |
|---|--------|---------|--------|------------|------|---------|
| 1 | ALLOW | TCP/UDP | WORK net | 10.10.10.10 | 53 | DNS to domain controller |
| 2 | ALLOW | Any | WORK net | 10.10.10.0/24 | Any | Intra-VLAN: all Work devices can talk to each other |
| 3 | ALLOW | Any | 10.10.10.10, 10.10.10.20 | 10.10.99.0/24 | Any | Admin IPs (dc01 + ubuntu-admin) → MGMT VLAN |
| 4 | ALLOW | Any | WORK net | WAN net | Any | Internet access for all Work devices |
| 5 | BLOCK | Any | WORK net | 10.10.20.0/24 | Any | No Corporate → Isolated VLAN access |
| 6 | BLOCK | Any | WORK net | 10.10.99.0/24 | Any | Non-admin Work → Management VLAN blocked |
| 7 | BLOCK | Any | Any | Any | Any | Explicit default deny (belt and suspenders) |

**Important order:** Rule 3 (admin IPs allowed to MGMT) is above Rule 6 (all Work blocked from MGMT).
Since dc01 (10.10.10.10) and ubuntu-admin (10.10.10.20) are specifically listed in Rule 3,
they match Rule 3 first and are allowed. All other Work IPs skip Rule 3 (no match), hit Rule 6 (match), and are blocked.

---
