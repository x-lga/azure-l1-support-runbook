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
