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

## Procedure E - Investigate Sign-In Failures

When a user cannot sign in and the account is confirmed enabled and not locked,
the Sign-In Logs reveal exactly why authentication is failing.

```
Azure Portal → Entra ID → Users → [User Name] →
  Sign-In Logs

  OR

Azure Portal → Entra ID → Monitoring & Health → Sign-in Logs →
  Filter by: User = [username] and Status = Failure
```

**Key fields to read in the sign-in log entry:**

| Field | What to Check |
|-------|--------------|
| Status | Success / Failure |
| Failure reason | The human-readable reason for failure |
| Error code | The numeric code - reference against error code list |
| Conditional Access | Which CA policies were evaluated and their result |
| Client app | What application attempted the sign-in |
| IP address | Geographic source - unusual location? |
| Device | Which device attempted sign-in - known or unknown? |

**Common sign-in failure reasons and L1 actions:**

| Failure Reason | Code | L1 Action |
|---------------|------|-----------|
| Invalid password | 50126 | Password reset |
| Account disabled | 50057 | Check if intentional - escalate for re-enable |
| Account locked | 50053 | Unlock (on-prem via Unlock-ADUserAccount.ps1, or cloud via Portal) |
| MFA required, not satisfied | 50076 | Guide user through MFA setup or reset MFA methods |
| Conditional Access policy blocked | 53003 | Check which CA policy blocked - escalate if policy change needed |
| User not found | 50034 | Wrong UPN - verify username, check for typo |
| Token expired | 70008 | User needs to sign in again - usually auto-resolves |
| Application not permitted | 65004 | App consent issue - escalate to admin |
| AADSTS50105 | 50105 | User not assigned to the application - admin must assign user |

---


## Procedure F - Bulk User Report for Audit

When management or compliance requires a list of all users, their status, and last
sign-in date for an access review:

```
Azure Portal → Entra ID → Users →
  Download Users →
  Download all users as CSV

This exports:
  - Display name, UPN, account status (enabled/disabled)
  - Last sign-in date, creation date
  - Assigned licences
  - Department, job title
  - Sync source (cloud vs on-prem AD)
```

**Query via Log Analytics for deeper audit data:**
```kql
// Users who have not signed in for 90+ days
SigninLogs
| where TimeGenerated > ago(90d)
| summarize LastSignIn = max(TimeGenerated) by UserPrincipalName
| where LastSignIn < ago(90d) OR isnull(LastSignIn)
| project UserPrincipalName, LastSignIn
| order by LastSignIn asc

// Sign-in activity by country in the last 30 days
SigninLogs
| where TimeGenerated > ago(30d)
| where ResultType == 0
| summarize SignInCount = count() by
    UserPrincipalName,
    Location = tostring(LocationDetails.countryOrRegion)
| order by SignInCount desc
```

---

## Procedure G - Assign a Licence to a User

```
Azure Portal → Entra ID → Users → [User Name] →
  Licences → Assignments → + Assign
  Select the licence (e.g., Microsoft 365 E3)
  Review service plans and disable any not needed
  → Save
```

**Or via group-based licensing (preferred at scale):**
```
Azure Portal → Entra ID → Groups →
  [Group name] → Licences → Assign
```
Group-based licensing automatically assigns the licence to all members of the group
and removes it when they leave. This is more scalable and less error-prone than
per-user licence assignment.

---

## Procedure H - Guest User Management (B2B)

When an external user (from another organisation) needs access to your Azure
resources or M365 applications:

**Invite a guest user:**
```
Azure Portal → Entra ID → Users → New User →
  Invite External User →
  Email address   : external.user@partner.com
  Display name    : [First Last]
  Message         : [Optional welcome message]
  → Invite
```

The external user receives an invitation email. They must accept the invitation
and authenticate with their own organisation's identity (or a Microsoft account).

**Check guest user access:**
```
Azure Portal → Entra ID → Users →
  Filter: User type = Guest
```

**Guest user security best practices:**
- Review guest access regularly (quarterly at minimum)
- Remove guest accounts when the collaboration ends
- Use Access Reviews (Entra ID P2) to automate the review process
- Restrict what guests can see by default:
  ```
  Entra ID → External Identities → External Collaboration Settings →
    Guest user access : "Guest users have limited access..."
  ```


---



