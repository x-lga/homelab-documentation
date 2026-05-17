# Azure Hybrid Identity - AD Connect and Entra ID

**Cert alignment:** AZ-900, AZ-104
**Lab VM:** dc01 (AD Connect installed here)
**Azure:** Free trial subscription + M365 Developer E5 Tenant
**Last reviewed:** 2026-07

---

## Overview

The Azure hybrid connection links the on-premises contoso.local Active Directory
domain to Microsoft Entra ID (formerly Azure Active Directory) via Azure AD Connect.
This enables single sign-on across on-premises and cloud resources, and mirrors the
architecture used by most enterprises that have existing on-premises infrastructure.

**What is synced:**
- User accounts (with their attributes - display name, department, title, email)
- Groups
- Password hashes (with Password Hash Sync enabled)
- Computer objects (if scope is configured to include them)

**What is NOT synced by default:**
- Passwords in plaintext (only the hash, and only with PHS enabled)
- Local admin accounts or built-in accounts like Administrator
- Contacts and distribution lists (can be configured but not default)

For the full build guide including installation steps, configuration walkthrough,
sync verification, and hybrid SSO testing - see the dedicated document in the
azure-hybrid-lab repo: `03-azure-ad-connect-hybrid-sync.md`.

---

## Lab-Specific Configuration

**AD Connect installed on:** dc01 (contoso.local domain controller)
**Sync mode:** Password Hash Synchronisation (PHS)
**Sync interval:** Every 30 minutes (default)
**Scope:** OU=Azure-Sync filtered - only users in the Azure-Sync OU are synced

**Why PHS was chosen:** See `08-design-decisions-and-justifications.md` in the
azure-hybrid-lab repo for the full explanation. Short version: PHS allows cloud
authentication to work even when the Proxmox lab host is powered down, which
happens regularly in a home lab environment.

---




