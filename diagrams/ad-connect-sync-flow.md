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
