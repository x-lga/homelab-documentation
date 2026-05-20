# Azure AD Connect Sync Flow Diagram

---

## Identity Sync Architecture

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                    ON-PREMISES (contoso.local)                               │
│                                                                              │
│  Active Directory Domain Services (dc01)                                     │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │  Domain: contoso.local                                                 │  │
│  │                                                                        │  │
│  │  OU=Azure-Sync                                                         │  │
│  │    ├── testuser1 (UPN: testuser1@contoso.local)                        │  │
│  │    ├── testuser2 (UPN: testuser2@contoso.local)                        │  │
│  │                                                                        │  │
│  │  OU=Staff\OU=IT                                                        │  │
│  │    ├── brian.otieno (UPN: brian.otieno@contoso.local)                  │  │
│  │    ├── david.mutua  (UPN: david.mutua@contoso.local)                   │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
│                           │                                                  │
│  Azure AD Connect         │ Reads objects from AD                            │
│  ┌────────────────────────▼───────────────────────────────────────────────┐  │
│  │  Sync Engine                                                           │  │
│  │  ├── Connector: AD (contoso.local)        — reads from AD              │  │
│  │  ├── Connector: Entra ID (contosodemo..)  — writes to Entra ID         │  │
│  │  ├── Metaverse: in-memory join space       — deduplication             │  │
│  │  ├── Sync mode: Password Hash Sync (PHS)                               │  │
│  │  └── Sync interval: Every 30 minutes                                   │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
│                           │                                                  │
│                           │ HTTPS/443 outbound to login.microsoftonline.com  │
└───────────────────────────┼──────────────────────────────────────────────────┘
                            │
                            ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                    MICROSOFT ENTRA ID (Azure AD)                             │
│                    Tenant: contosodemo.onmicrosoft.com                       │
│                                                                              │
│  Synced objects from contoso.local:                                          │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │  User: testuser1                                                       │  │
│  │    UPN       : testuser1@contosodemo.onmicrosoft.com                   │  │
│  │    Source    : Windows Server AD (synced)                              │  │
│  │    Password  : Hash synced (authenticates against cloud-stored hash)   │  │
│  │    On-prem DN: CN=testuser1,OU=Azure-Sync,DC=contoso,DC=local          │  │
│  │                                                                        │  │
│  │  User: brian.otieno                                                    │  │
│  │    UPN       : brian.otieno@contosodemo.onmicrosoft.com                │  │
│  │    Source    : Windows Server AD (synced)                              │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  Cloud-only objects (not synced from on-prem):                               │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │  admin@contosodemo.onmicrosoft.com  — Global Admin (created in cloud)  │  │
│  │  junioradmin@contosodemo...         — RBAC Contributor (cloud-only)    │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## What Happens During Each Sync Cycle

```
Every 30 minutes (or on manual trigger):

1. IMPORT from AD Connector
   AD Connect reads all objects in scope from contoso.local AD
   Changes since last import are identified (delta sync)

2. SYNCHRONISE in Metaverse
   Changed objects are evaluated against sync rules
   Attribute mapping is applied (e.g., sAMAccountName → userPrincipalName)
   Objects are joined or projected into the metaverse

3. EXPORT to Entra ID Connector
   Metaverse changes are applied to Entra ID
   New users → created in Entra ID
   Changed attributes → updated in Entra ID
   Deleted objects → disabled or deleted in Entra ID

4. PASSWORD HASH SYNC (runs separately, near real-time)
   When a password change is detected in AD:
   AD Connect reads the NT hash, derives a new hash, and syncs to Entra ID
   Happens within 2 minutes of password change - not waiting for 30-min cycle

Total sync latency for attribute changes: up to 30 minutes
Total sync latency for password changes: 2–5 minutes
```

---

## Sync Scope in This Lab

Only the `OU=Azure-Sync` OU is in scope by default (configured during AD Connect setup).
Users in other OUs (Staff, Contractors) are synced in lab builds where full-scope
sync has been enabled. For testing purposes, the Azure-Sync OU provides a controlled
set of accounts for cloud authentication testing without syncing all lab accounts.


---
