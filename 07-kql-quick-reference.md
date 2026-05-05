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

## Entra ID Sign-In Queries

```kql
// ── Failed sign-ins — last 24 hours ─────────────────────────────────────
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType != 0
| summarize FailureCount = count()
    by UserPrincipalName, ResultDescription, IPAddress,
    AppDisplayName
| order by FailureCount desc
```

```kql
// ── Sign-ins blocked by Conditional Access ───────────────────────────────
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType == 53003
| project TimeGenerated, UserPrincipalName,
    IPAddress, AppDisplayName,
    ConditionalAccessStatus,
    tostring(ConditionalAccessPolicies)
| order by TimeGenerated desc
```

```kql
// ── Sign-ins from new countries compared to last 30 days ─────────────────
// First run: get baseline countries
let baseline = SigninLogs
| where TimeGenerated between (ago(37d) .. ago(7d))
| where ResultType == 0
| summarize BaselineCountries = make_set(
    tostring(LocationDetails.countryOrRegion))
    by UserPrincipalName;
// Then find sign-ins from countries not in baseline
SigninLogs
| where TimeGenerated > ago(7d)
| where ResultType == 0
| extend Country = tostring(LocationDetails.countryOrRegion)
| join kind=leftouter baseline on UserPrincipalName
| where not(Country in (BaselineCountries))
| project TimeGenerated, UserPrincipalName, Country,
    IPAddress, AppDisplayName
| order by TimeGenerated desc
```

---

## App Service Queries

```kql
// ── HTTP 5xx errors in the last 1 hour ──────────────────────────────────
AppServiceHTTPLogs
| where TimeGenerated > ago(1h)
| where ScStatus >= 500
| summarize ErrorCount = count()
    by ScStatus, CsUriStem, bin(TimeGenerated, 5m)
| order by ErrorCount desc
```

```kql
// ── App Service response time — identify slow requests ──────────────────
AppServiceHTTPLogs
| where TimeGenerated > ago(1h)
| where TimeTaken > 3000
| project TimeGenerated, CsMethod, CsUriStem,
    ScStatus, TimeTaken, CIp
| order by TimeTaken desc
| take 50
```

```kql
// ── App Service restart history ─────────────────────────────────────────
AppServicePlatformLogs
| where TimeGenerated > ago(7d)
| where OperationName == "Restart"
| project TimeGenerated, OperationName, Level, Message
| order by TimeGenerated desc
```

---

## Storage Account Queries

```kql
// ── Storage account authentication failures ──────────────────────────────
StorageBlobLogs
| where TimeGenerated > ago(24h)
| where StatusCode in (403, 401)
| summarize FailureCount = count()
    by CallerIpAddress, AuthenticationType, StatusText,
    bin(TimeGenerated, 1h)
| order by FailureCount desc
```

```kql
// ── Large blob read operations (potential data exfiltration indicator) ────
StorageBlobLogs
| where TimeGenerated > ago(24h)
| where OperationName == "GetBlob"
| where ResponseBodySize > 104857600  // > 100 MB
| project TimeGenerated, CallerIpAddress,
    Uri, ResponseBodySize, AuthenticationType
| order by ResponseBodySize desc
```

---

## Useful KQL Operators and Functions

```kql
// Time filters
| where TimeGenerated > ago(1h)                    // Last hour
| where TimeGenerated > ago(24h)                   // Last 24 hours
| where TimeGenerated > ago(7d)                    // Last 7 days
| where TimeGenerated between (ago(7d) .. ago(1d)) // 7 to 1 day ago

// Aggregation
| summarize count() by FieldName                   // Count grouped by field
| summarize avg(CounterValue) by Computer          // Average grouped by Computer
| summarize max(CounterValue) by bin(TimeGenerated, 5m) // Max per 5-minute bucket

// Filtering
| where FieldName == "exact value"                 // Exact match
| where FieldName contains "partial"               // Contains (case-insensitive)
| where FieldName startswith "prefix"              // Starts with
| where FieldName endswith "suffix"                // Ends with
| where FieldName has "word"                       // Has (word-boundary match)
| where FieldName in ("a", "b", "c")               // In list
| where FieldName !in ("x", "y")                   // Not in list
| where isnotnull(FieldName)                       // Field is not null

// Projection and renaming
| project Field1, Field2, Field3                   // Select specific fields
| project-away FieldToRemove                       // Remove a field
| extend NewField = expression                     // Add a computed field

// Sorting and limiting
| order by FieldName desc                          // Sort descending
| sort by FieldName asc                            // Sort ascending
| take 50                                          // Return first 50 rows
| top 10 by CountValue desc                        // Return top 10 by value

// Rendering
| render timechart                                 // Line chart over time
| render barchart                                  // Bar chart
| render piechart                                  // Pie chart
| render table                                     // Tabular view (default)
```


---
