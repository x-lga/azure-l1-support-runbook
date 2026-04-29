# azure-l1-support-runbook

Azure Portal L1 support runbook covering the six most common Azure support ticket
types encountered in cloud operations roles - VM troubleshooting, Entra ID user
management, NSG and network diagnostics, App Service recovery, storage access issues,
and cost management - along with a KQL quick reference for log-based investigation,
a comprehensive Azure error code guide, and supporting documentation for portal
navigation and escalation packaging.

Built from AZ-104 training and hands-on Azure hybrid lab experience. Every procedure
is designed to be followed in real time during an incident - not just read as theory.

---

## What this repo contains

| File | Purpose | Who uses it |
|------|---------|-------------|
| `01-vm-troubleshooting.md` | VM start failures, power states (Restart vs Stop/Start distinction), Activity Log, Resource Health, Boot Diagnostics, Serial Console, NSG/JIT access, performance degradation with KQL | Azure cloud support L1, operations engineers |
| `02-entra-id-user-management.md` | Cloud-only vs synced account distinction, password reset for both types, enable/disable, session revoke, MFA reset, sign-in failure investigation with error codes, bulk user report, guest user B2B management | IT support, IAM teams, cloud admins |
| `03-nsg-and-network-diagnostics.md` | IP Flow Verify, Effective Security Rules, NSG rule addition with priority guidance, Connection Troubleshoot, NSG Flow Log queries, Azure DNS troubleshooting for hybrid environments | Network-aware L1, cloud support engineers |
| `04-app-service-recovery.md` | App Service vs App Service Plan distinction, HTTP 502/503/504 triage, restart vs stop/start, application logs, Log Analytics app log queries, deployment slot swap for rollback, SSL certificate issues, Kudu console | Cloud support, web application operations |
| `05-storage-and-access-issues.md` | 403 AuthorizationFailure diagnosis (RBAC vs firewall vs access keys vs SAS), 404 BlobNotFound, Azure Files mounting errors (port 445, SMB persistence), Archive rehydration, Lifecycle Management, storage cost optimisation | Cloud support, application support engineers |
| `06-cost-and-subscription-management.md` | Cost Analysis investigation, orphaned resource cleanup checklist, budget alert setup, quota errors and increase requests, resource tagging for cost attribution, Azure Advisor cost recommendations | Cloud admins, FinOps practitioners, anyone managing Azure spend |
| `07-kql-quick-reference.md` | 25+ tested KQL queries covering VM performance, security events (4625/4624/4740/4720/4672), Azure Activity Log, Entra ID sign-in analysis, App Service HTTP errors, Storage authentication failures, plus KQL operator reference | Any Azure engineer using Log Analytics |
| `08-common-azure-error-codes.md` | 20 error codes across Compute, Networking, Identity, Storage, and App Service - each with plain-language explanation, root cause, and L1 action | L1 support engineers, AZ-104 exam prep |
| `docs/azure-portal-navigation-guide.md` | Navigation paths for every common portal blade, Azure search usage, Cloud Shell commands | L1 engineers new to Azure Portal |
| `docs/escalation-package-template.md` | Complete L2 escalation template, guidance on when to open Microsoft support tickets directly, severity levels and response times | All L1 engineers |
| `docs/azure-support-tiers-reference.md` | Microsoft support plan comparison, Azure Status page URLs, escalation decision tree | L1 engineers communicating support timelines |

---

## AZ-104 Exam Domains Mapped to This Runbook

| AZ-104 Domain | Coverage in This Runbook |
|--------------|------------------------|
| Manage Azure Identities (20-25%) | Entra ID user management (Runbook 02): cloud vs synced accounts, password reset, session revoke, MFA reset, sign-in log investigation |
| Implement and Manage Virtual Networking (15-20%) | NSG diagnostics (Runbook 03): IP Flow Verify, Effective Security Rules, Connection Troubleshoot, NSG Flow Logs, DNS troubleshooting |
| Deploy and Manage Azure Compute (20-25%) | VM troubleshooting (Runbook 01): power states, Activity Log, Resource Health, Boot Diagnostics, Serial Console; App Service (Runbook 04) |
| Implement and Manage Azure Storage (10-15%) | Storage access issues (Runbook 05): RBAC vs firewall vs keys vs SAS, 404 diagnosis, Azure Files mounting, Archive rehydration |
| Monitor and Maintain Azure Resources (10-15%) | KQL reference (Runbook 07): 25+ queries across all resource types; performance investigation (Runbooks 01, 04) |
| Manage Azure Governance (15-20%) | Cost management (Runbook 06): Cost Analysis, orphaned resources, budgets, quotas, tagging; Error codes (Runbook 08): policy and RBAC errors |

---

## Skills demonstrated

**Azure troubleshooting methodology:**
Systematic triage starting with portal status → Activity Log → Resource Health →
specific diagnostic tools (Boot Diagnostics, Serial Console, IP Flow Verify) →
escalation with a complete package. Every procedure follows this structure.

**Critical Azure distinctions:**
- Restart vs Stop (Deallocate) + Start - why they are different and when each matters
- Cloud-only vs synced Entra ID accounts - why changes revert for synced accounts
- Storage account RBAC roles vs data plane roles - why Contributor does not grant blob access
- NSG subnet vs NIC level - why both are checked in Effective Security Rules
- App Service vs App Service Plan - why the plan health affects all apps on it

**KQL investigation:**
25+ tested queries covering every runbook scenario - analysts can investigate any
incident in the log layer without needing to be redirected to a SIEM team.

**Error code fluency:**
20 error codes with plain-language explanations and L1 actions - eliminates the
"what does this mean?" delay and communicates clearly to non-technical stakeholders.

**ITIL 4 alignment:**
Every runbook has P1-P3 priority classifications with SLA targets, escalation
criteria, and a structured escalation package template that L2 can act on immediately.


---

## How to use this runbook

```bash
# Clone
git clone https://github.com/YOUR-USERNAME/azure-l1-support-runbook.git
cd azure-l1-support-runbook

# Start with the most common ticket type
cat 01-vm-troubleshooting.md

# Reference error codes during any investigation
cat 08-common-azure-error-codes.md

# Copy KQL queries directly into Azure Log Analytics
cat 07-kql-quick-reference.md

# Use the escalation template before every L2 escalation
cat docs/escalation-package-template.md
```

---

## Impact

This runbook operationalises AZ-104 knowledge into real support procedures. The
distinction between Restart and Stop/Deallocate+Start is not just an exam question —
it is the difference between a resolved AllocationFailed ticket and 30 wasted minutes
of trial-and-error. The Entra ID synced vs cloud-only account distinction prevents the
frustrating cycle of making a change in the portal and watching it revert 30 minutes
later. The KQL reference turns every investigation from "wait for the SIEM team" into
"run this query now."

These are the operational details that textbooks do not emphasise but that every
cloud support engineer encounters in their first week.


---
