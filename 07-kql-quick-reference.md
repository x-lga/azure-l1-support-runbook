# KQL Quick Reference - Azure Log Analytics

KQL (Kusto Query Language) is the query language used in Azure Log Analytics,
Microsoft Sentinel, and Azure Monitor. These queries are tested and ready to
paste directly into a Log Analytics Workspace query window.

**Where to run these queries:**
```
Azure Portal → Log Analytics Workspaces → [Workspace Name] → Logs →
  Paste the query → Run
```

All queries assume a Log Analytics Workspace with Diagnostic Settings configured
to collect logs from Azure VMs, NSGs, App Services, and Activity Logs.

---
