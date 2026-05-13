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


