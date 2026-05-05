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

## Security and Authentication Queries

```kql
// ── Failed Windows login attempts (EventID 4625) - last 24 hours ────────
SecurityEvent
| where TimeGenerated > ago(24h)
| where EventID == 4625
| summarize FailureCount = count()
    by Account, IpAddress, Computer,
    bin(TimeGenerated, 1h)
| order by FailureCount desc
```

```kql
// ── Successful logins outside business hours ─────────────────────────────
SecurityEvent
| where TimeGenerated > ago(7d)
| where EventID == 4624
| extend Hour = hourofday(TimeGenerated)
| where Hour < 7 or Hour > 19
| summarize Count = count()
    by Account, IpAddress, Computer, Hour
| order by Count desc
```

```kql
// ── Account lockouts (EventID 4740) ─────────────────────────────────────
SecurityEvent
| where TimeGenerated > ago(24h)
| where EventID == 4740
| project TimeGenerated, TargetAccount, Computer, SubjectUserName
| order by TimeGenerated desc
```

```kql
// ── New user account created (EventID 4720) ─────────────────────────────
SecurityEvent
| where TimeGenerated > ago(7d)
| where EventID == 4720
| project TimeGenerated, TargetUserName,
    SubjectUserName, Computer
| order by TimeGenerated desc
```

```kql
// ── Privilege escalation — special logon (EventID 4672) ──────────────────
SecurityEvent
| where TimeGenerated > ago(24h)
| where EventID == 4672
| project TimeGenerated, Account, PrivilegeList, Computer
| where PrivilegeList has "SeDebugPrivilege"
    or PrivilegeList has "SeTakeOwnershipPrivilege"
| order by TimeGenerated desc
```

---

## Azure Activity Log Queries

```kql
// ── All operations in the last 7 days by a specific user ────────────────
AzureActivity
| where TimeGenerated > ago(7d)
| where Caller == "admin@contosodemo.onmicrosoft.com"
| project TimeGenerated, OperationNameValue,
    ActivityStatusValue, ResourceGroup, _ResourceId
| order by TimeGenerated desc
```

```kql
// ── Resource deletions in the last 7 days ───────────────────────────────
AzureActivity
| where TimeGenerated > ago(7d)
| where OperationNameValue endswith "delete"
    and ActivityStatusValue == "Success"
| project TimeGenerated, Caller, OperationNameValue,
    ResourceGroup, _ResourceId
| order by TimeGenerated desc
```

```kql
// ── NSG rule changes in the last 7 days ─────────────────────────────────
AzureActivity
| where TimeGenerated > ago(7d)
| where OperationNameValue has "networkSecurityGroups"
    and ActivityStatusValue == "Success"
| project TimeGenerated, Caller, OperationNameValue, Properties
| order by TimeGenerated desc
```

```kql
// ── VM start and stop operations ────────────────────────────────────────
AzureActivity
| where TimeGenerated > ago(7d)
| where OperationNameValue in (
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/deallocate/action",
    "Microsoft.Compute/virtualMachines/restart/action"
  )
| project TimeGenerated, Caller, OperationNameValue,
    ActivityStatusValue, _ResourceId
| order by TimeGenerated desc
```

---
