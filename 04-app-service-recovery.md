# Runbook 04 - Azure App Service Recovery

**Category:** Compute - Azure App Service (Web Apps, Function Apps)
**ITIL 4 Priority:** P1 (production web application or API down) | P2 (degraded performance) | P3 (non-production environment)
**Cert alignment:** AZ-104 - Deploy and Manage Azure Compute Resources
**Last reviewed:** 2026-07

---

## App Service Architecture Overview

Understanding the App Service architecture helps interpret issues correctly:

```
App Service Plan (the underlying compute - VM equivalent)
  └── App Service / Web App (the application running on the plan)
       ├── Deployment Slots (staging, production - used for zero-downtime deploys)
       ├── Application Settings (environment variables, connection strings)
       ├── Custom domains and SSL certificates
       └── Log streaming / Diagnostic logs
```

**Key implication:** If the App Service Plan is unhealthy or at resource limits,
ALL apps running on that plan are affected. When investigating a single app issue,
always check the App Service Plan health first.

---

## Procedure A - App Service Not Responding (HTTP 502 / 503 / 504)

### Step 1 - Check App Service Status

```
Azure Portal → App Services → [App Name] → Overview

Check:
  Status        : Running / Stopped
  URL           : [your-app].azurewebsites.net - can you reach this in a browser?
  HTTP 5xx errors in the metrics summary
```

**Start a stopped app:**
```
Azure Portal → App Services → [App Name] → Overview → Start
```

### Step 2 - Check the App Service Plan

```
Azure Portal → App Services → [App Name] →
  App Service Plan → [Plan Name] → Overview

Check:
  CPU Percentage    : Sustained >80% = plan at CPU limit
  Memory Percentage : Sustained >80% = plan at memory limit
  HTTP Queue Length : Sustained >0 = requests queuing — plan cannot keep up
```

If the plan is at resource limits, the fix may be to scale up (larger instance)
or scale out (more instances), not to restart the app.

**Scale up the App Service Plan:**
```
Azure Portal → App Service Plan → Scale Up (App Service Plan)
  Choose a higher tier (e.g., B2 → B3 → P1v3)
```

**Scale out (add more instances):**
```
Azure Portal → App Service Plan → Scale Out (App Service Plan)
  Manual scale → Increase instance count
  OR configure Autoscale rules
```
