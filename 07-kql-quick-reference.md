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

## Virtual Machine Queries

```kql
// ── All activity on a specific VM in the last 1 hour ───────────────────
AzureActivity
| where TimeGenerated > ago(1h)
| where _ResourceId has "vm-win-server"
| project TimeGenerated, Caller, OperationNameValue,
    ActivityStatusValue, Properties
| order by TimeGenerated desc
```

```kql
// ── All failed Azure operations in a resource group ─────────────────────
AzureActivity
| where TimeGenerated > ago(24h)
| where ResourceGroup == "rg-hybrid-lab"
| where ActivityStatusValue == "Failure"
| project TimeGenerated, Caller, OperationNameValue, Properties
| order by TimeGenerated desc
```

```kql
// ── VM CPU utilisation over time ────────────────────────────────────────
Perf
| where TimeGenerated > ago(1h)
| where ObjectName == "Processor"
    and CounterName == "% Processor Time"
    and InstanceName == "_Total"
| summarize AvgCPU = avg(CounterValue), MaxCPU = max(CounterValue)
    by Computer, bin(TimeGenerated, 5m)
| render timechart
```

```kql
// ── VM memory — available megabytes over time ────────────────────────────
Perf
| where TimeGenerated > ago(1h)
| where ObjectName == "Memory"
    and CounterName == "Available MBytes"
| summarize AvgAvailMB = avg(CounterValue)
    by Computer, bin(TimeGenerated, 5m)
| render timechart
```

```kql
// ── Disk free space below 20% ─────────────────────────────────────────
Perf
| where ObjectName == "LogicalDisk"
    and CounterName == "% Free Space"
    and InstanceName != "_Total"
    and InstanceName != "HarddiskVolume1"
| where CounterValue < 20
| project TimeGenerated, Computer,
    Drive = InstanceName,
    FreeSpacePct = round(CounterValue, 1)
| order by FreeSpacePct asc
```

```kql
// ── Disk I/O — queue depth (sustained > 5 = disk bottleneck) ────────────
Perf
| where TimeGenerated > ago(1h)
| where ObjectName == "LogicalDisk"
    and CounterName == "Current Disk Queue Length"
    and InstanceName != "_Total"
| where CounterValue > 3
| summarize AvgQueue = avg(CounterValue)
    by Computer, InstanceName, bin(TimeGenerated, 5m)
| render timechart
```

---
