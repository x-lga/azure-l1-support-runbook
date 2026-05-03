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
Note that the next sync will not overwrite this - it only writes the on-prem hash
if the on-prem password changes.

---

## Procedure B - Enable or Disable an Account

### Disable (Cloud-Only):
```
Azure Portal → Entra ID → Users → [User] →
  Edit Properties → Account Enabled: toggle to No → Save
```

### Disable via PowerShell (Az module / Microsoft Graph):
```powershell
# Using Microsoft Graph PowerShell
Connect-MgGraph -Scopes "User.ReadWrite.All"
Update-MgUser -UserId "user@domain.com" -AccountEnabled:$false
Write-Host "Account disabled: user@domain.com"
```

### Enable (requires L2 authorisation for synced accounts):
```
Azure Portal → Entra ID → Users → [User] →
  Edit Properties → Account Enabled: toggle to Yes → Save
```

**L1 rule:** Enabling a disabled account requires confirmation that the disablement
was not intentional (offboarding, security hold). Always check with your manager
or L2 before re-enabling any account. Never re-enable based solely on a user's
request without verification.

---

## Procedure C - Revoke All Active Sessions (Compromised Account Response)

When an account is suspected of compromise - phishing victim entered credentials,
unusual sign-in activity, unknown device sign-in - revoke all active sessions
immediately to kick the attacker out.

```
Azure Portal → Entra ID → Users → [User Name] →
  Revoke Sessions

This action:
  - Signs the user out of all browser sessions (M365 web apps)
  - Signs the user out of Outlook, Teams, OneDrive mobile and desktop apps
  - Invalidates all access tokens (takes effect within ~1 hour as tokens expire)
  - Does NOT disable the account (user can sign back in immediately)

If account compromise is confirmed:
  Do BOTH: Revoke Sessions AND Disable the account
```

**Important:** Revoking sessions invalidates refresh tokens but existing access
tokens with short lifetimes (typically 1 hour) may still work until they expire
naturally. For immediate lockout, disable the account.

---

## Procedure D - Reset MFA for a User

When a user has lost their phone, changed their phone number, or is reporting "MFA
not working," their MFA methods need to be cleared so they can re-register.

```
Azure Portal → Entra ID → Users → [User Name] →
  Authentication Methods →
  Require Re-register MFA

This clears all existing MFA registrations.
The user will be prompted to set up new MFA on next sign-in.
```

**Before clearing MFA:**
- Verify the user's identity through an alternative method
  (manager confirmation, HR verification, video call with ID)
- Confirm this is not a social engineering attempt to bypass MFA
- Log the reason in the ticket

**Alternative: Remove specific MFA methods:**
```
Azure Portal → Entra ID → Users → [User] →
  Authentication Methods →
  Delete the specific method (phone number, authenticator app) individually
```

---



