# Runbook 06 - Cost and Subscription Management

**Category:** Governance - Cost Management, Subscription, and Resource Organisation
**ITIL 4 Priority:** P3 (cost query) | P2 (unexpected charges) | P1 (quota exceeded blocking production)
**Cert alignment:** AZ-104 - Manage Azure Identities and Governance
**Last reviewed:** 2026-07

---

## Procedure A - Investigating Unexpected Azure Charges

When a user reports unexpected Azure charges or asks "why is my Azure bill higher
than expected?", this is the systematic investigation path.

### Step 1 - Open Cost Analysis

```
Azure Portal → Cost Management + Billing →
  Cost Management → Cost Analysis

Filter:
  Scope       : [Your subscription]
  Granularity : Daily
  Date range  : Current month or last month

Group by:
  Service name    : Shows which Azure service is incurring the most cost
  Resource        : Shows which specific resource is most expensive
  Resource group  : Shows which resource group is most costly
  Location        : Shows geographic distribution of cost
```

### Step 2 - Identify the Top Cost Drivers

Sort by cost descending. Common high-cost resources:

| Resource Type | Common Cost Driver | Mitigation |
|-------------|-------------------|-----------|
| Virtual Machines | Running 24/7 when only needed 8/5 | Configure auto-shutdown schedule |
| Azure Bastion | Running continuously | Stop Bastion when not in use; auto-shutdown |
| Azure SQL Database | Incorrect tier | Downsize if DTUs/vCores are consistently low |
| Bandwidth / Data Transfer | Large outbound data transfer | Investigate what data is being transferred and to where |
| Azure Monitor / Log Analytics | High data ingestion | Review what is being forwarded to Log Analytics; set data caps |
| Load Balancer | Unnecessary data processing rules | Review and remove unused rules |
| Public IP Addresses | Idle public IPs (even unattached IPs incur charges) | Delete unattached public IPs |


### Step 3 - Check for Orphaned Resources

Orphaned resources are Azure resources that continue to incur charges even though
they are no longer needed - typically created as dependencies that were not deleted
when the primary resource was removed.

Common orphaned resources:
```
Azure Portal → Virtual Networks → [VNet] → Connected Devices
  → Look for NICs not attached to any VM (NIC without an associated VM)

Azure Portal → Public IP Addresses
  → Filter: Status = Unassociated
  → Delete all unassociated public IPs

Azure Portal → Managed Disks
  → Filter: Disk State = Unattached
  → Review and delete unneeded unattached disks (these are billed at full rate)

Azure Portal → Network Security Groups
  → Check for NSGs not associated with any subnet or NIC

Azure Portal → Snapshots
  → Disk snapshots accumulate and are rarely cleaned up automatically
  → Review and delete old snapshots
```

---

## Procedure B - Set Up Budget Alerts

Budget alerts notify you before costs exceed a threshold — preventing unexpected
large bills.

```
Azure Portal → Cost Management + Billing →
  Cost Management → Budgets → + Add

  Scope         : [Subscription or Resource Group]
  Name          : budget-monthly-lab
  Reset period  : Monthly
  Budget amount : [e.g., $50 — your expected monthly spend]

  Alert conditions:
    Alert 1: Actual spend > 80% of budget → email alert@domain.com
    Alert 2: Actual spend > 100% of budget → email alert@domain.com
    Alert 3: Forecasted spend > 110% of budget → email alert@domain.com

  → Create
```

Budget alerts do NOT stop resources from running - they only send notifications.
To automatically act on budget alerts (stop VMs when budget is exceeded), use
Azure Automation runbooks or Azure Logic Apps triggered by Cost Management alerts.

---
