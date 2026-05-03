# Runbook 02 - Entra ID User Management

**Category:** Identity and Access - Microsoft Entra ID (formerly Azure Active Directory)
**ITIL 4 Priority:** P3 (single user access issue) | P2 (executive or critical role) | P1 (organisation-wide auth failure)
**Cert alignment:** AZ-104 - Manage Azure Identities and Governance
**Last reviewed:** 2026-07

---

## Important Context: Cloud-Only vs Synced Accounts

Before making any change to a user account in Entra ID, determine whether the
account is cloud-only or synced from on-premises Active Directory.

```
Azure Portal → Entra ID → Users → [User] → Properties
  Check: "Identity" section
  Source: "Microsoft account" or "Azure Active Directory"  = Cloud-only account
  Source: "Windows Server AD"                               = Synced from on-prem AD
```

**Why this matters:**
- **Cloud-only accounts:** Changes made in Entra ID Portal take effect immediately
- **Synced accounts:** Many properties are managed on-premises. Changes made in
  Entra ID for synced accounts will be **overwritten** by the next AD Connect sync
  cycle (every 30 minutes). Password resets and most attribute changes must be done
  on the on-premises Active Directory DC, not in Entra ID.

This is one of the most common mistakes made by junior cloud admins: making a change
in Entra ID for a synced user and finding it reverted 30 minutes later.

---
