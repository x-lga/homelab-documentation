# Microsoft Intune Endpoint Management

**Cert alignment:** AZ-900, AZ-104
**Platform:** M365 Developer E5 Sandbox (endpoint.microsoft.com)
**Enrolled device:** win10-client (domain-joined, VLAN 10)
**Last reviewed:** 2026-07

---

## Overview

Microsoft Intune is deployed using the M365 Developer E5 Sandbox - a free 90-day
tenant that includes Intune, Entra ID P2, Defender for Endpoint, and all Microsoft
365 services. The tenant is linked to the same Entra ID where AD Connect syncs
contoso.local users, creating a unified identity and management plane.

For the full Intune configuration guide - enrollment, compliance policy, configuration
profiles, and Win32 app deployment - see the dedicated document in the azure-hybrid-lab
repo: `06-intune-device-compliance.md`.

---

## Lab-Specific Intune Configuration

**Enrolled devices:** win10-client (Windows 10 22H2)
**Compliance policy name:** compliance-windows-security-baseline
**Configuration profile:** config-security-baseline-windows

**Key compliance requirements active in the lab:**

| Setting | Value | Why |
|---------|-------|-----|
| Minimum OS version | 10.0.19041 | Enforces current baseline — machines below this version are non-compliant |
| Password required | Yes, alphanumeric, min 10 chars | Aligns with the GPO password policy for consistency |
| Windows Defender Antivirus | Required | Ensures real-time protection is not disabled |
| Defender real-time protection | Required | Catches tampering with AV settings |
| BitLocker | Not configured (hardware limitation) | Lab VM does not have virtual TPM — see troubleshooting |

---

## Monitoring Device Compliance

```
Intune Portal (endpoint.microsoft.com) →
  Devices → Monitor → Device Compliance

Shows:
  Compliant    : Devices meeting all policy requirements
  Not Compliant: Devices with at least one failing requirement
  In Grace Period: Recently enrolled — within grace period before enforcement
```

When a device shows Not Compliant:
```
Intune Portal → Devices → All Devices → [Device Name] →
  Device Compliance → [Policy Name]

This shows exactly which settings are failing and their current vs required values.
```

---
