# VLAN Firewall Rules - Complete Reference

This document is the single source of truth for all pfSense firewall rules
in the lab. It supplements the topology diagram with the exact rule logic.

---

## Rule Processing Reminder

pfSense evaluates rules on the **ingress interface** (the interface where traffic enters pfSense).
Rules are evaluated **top to bottom - first match wins**.
Default action if no rule matches: **BLOCK** (implicit deny-all).

---
