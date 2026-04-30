# Azure Support Tiers Reference

Understanding Azure support tiers helps set correct expectations when communicating
with end users about Microsoft support timelines, and helps determine when to open
a support case vs handle internally.

---

## Microsoft Azure Support Plans

| Plan | Cost | Severity A Response | Severity B Response | Best For |
|------|------|--------------------|--------------------|---------|
| Basic | Free (included with subscription) | No technical support | No technical support | Billing questions only |
| Developer | ~$29/month | No 24/7 | 1 business day | Non-production environments |
| Standard | ~$100/month | < 1 hour | < 4 hours | Production workloads |
| Professional Direct | ~$1,000/month | < 1 hour | < 2 hours | Business-critical workloads |
| Premier / Unified | Custom (enterprise) | < 15 minutes | < 1 hour | Large enterprise with dedicated TAM |

---

## Azure Status Pages - Quick Reference

| Resource | URL | Purpose |
|---------|-----|---------|
| Azure Status (global) | azure.status.microsoft.com | Current and historical platform incidents by region and service |
| Azure Service Health | Portal → Service Health | Incidents, planned maintenance, and health advisories specific to YOUR subscription |
| Microsoft 365 Status | status.office365.com | M365 service health (separate from Azure) |
| Microsoft Defender | security.microsoft.com → Service health | Defender for Endpoint/Cloud service status |

---

## Escalation Decision Tree for Azure Issues

```
Is the issue causing complete production outage?
  YES → P1. Attempt L1 steps simultaneously with L2 notification. Phone call.
  NO  ↓

Is the issue affecting multiple users or resources?
  YES → P2 minimum. Escalate if L1 cannot resolve in 30 minutes.
  NO  ↓

Does Azure Resource Health show Platform Initiated Unavailability?
  YES → This is Azure's issue. Open Microsoft support ticket at Severity A or B.
        Do not spend time on internal L2 escalation.
  NO  ↓

Is the error code in the Azure Error Codes reference?
  YES → Follow the L1 action. If action fails or error repeats: escalate to L2.
  NO  ↓

Is the issue in the Activity Log with a clear error message?
  YES → Research the error message. Attempt L1 resolution. Escalate if needed.
  NO  ↓

Escalate to L2 with as much information as available.
Use the Escalation Package Template.
```


---
