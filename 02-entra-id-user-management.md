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

## Procedure A - Reset a User Password

### For Cloud-Only Accounts:
```
Azure Portal → Entra ID → Users → [User Name] →
  Reset Password →
  Auto-generate password (recommended) or Set custom password →
  User must change password on next sign-in: Yes →
  Reset
```

Copy the temporary password. Communicate to the user via phone or secure channel.

### For Synced Accounts:
Password must be reset on the **on-premises Active Directory** domain controller.
```powershell
# Run on the on-premises DC
.\Reset-ADUserPassword.ps1 -Username <samaccountname>
# Or manually:
Set-ADAccountPassword -Identity <username> `
    -NewPassword (Read-Host "New password" -AsSecureString) -Reset
Set-ADUser -Identity <username> -ChangePasswordAtLogon $true
```

After the on-premises reset, the password hash is synced to Entra ID within
30 minutes (or force a sync: `Start-ADSyncSyncCycle -PolicyType Delta`).

If the user needs access immediately:
```
Azure Portal → Entra ID → Users → [User] →
  Reset Password → use "Auto-generate password"
```
This resets the cloud password directly and bypasses the sync for immediate access.
Note that the next sync will not overwrite this — it only writes the on-prem hash
if the on-prem password changes.

---
