# Azure Portal Navigation Guide

Quick reference for finding key blades in the Azure Portal. For L1 engineers
new to Azure, the Portal can be disorienting - resources are accessible through
multiple paths and the layout changes with updates.

---

## Most-Used Navigation Paths

### Virtual Machines
```
Azure Portal → Virtual Machines → [VM Name]
  ├── Overview          : Status, start/stop/restart, connect, metrics summary
  ├── Activity Log      : All operations - failed operations show here
  ├── Networking        : NIC, NSG rules, IP addresses
  ├── Monitoring        : Metrics charts, diagnostic settings
  ├── Help              : Resource Health, Boot Diagnostics, Serial Console
  └── Connect           : Bastion, RDP, SSH connection options
```
### Entra ID
```
Azure Portal → Entra ID (Microsoft Entra ID)
  ├── Users             : User accounts — manage, reset password, disable
  ├── Groups            : Group membership and licensing
  ├── Enterprise Apps   : Application access and permissions
  ├── Security          : Conditional Access, MFA, Risky users
  └── Monitoring        : Sign-in Logs, Audit Logs
```

### Network Security Groups
```
Azure Portal → Network Security Groups → [NSG Name]
  ├── Overview          : Associated subnets and NICs
  ├── Inbound rules     : All inbound allow/deny rules
  ├── Outbound rules    : All outbound allow/deny rules
  └── Diagnostic        : NSG Flow Logs
```

### Network Watcher
```
Azure Portal → Network Watcher
  ├── IP Flow Verify    : "Is this traffic allowed by NSG rules?"
  ├── Connection Troubleshoot : "Can this VM reach this destination?"
  ├── Packet Capture    : Capture raw network packets from a VM NIC
  ├── NSG Diagnostics   : Check NSG rule evaluation for specific traffic
  └── Topology          : Visual network topology map
```
### Log Analytics / Monitoring
```
Azure Portal → Log Analytics Workspaces → [Workspace] → Logs
  → Write and run KQL queries

Azure Portal → Monitor
  ├── Metrics           : Real-time metric charts for any Azure resource
  ├── Alerts            : Alert rules, fired alerts, action groups
  ├── Activity Log      : Subscription-wide activity log
  └── Workbooks         : Pre-built monitoring dashboards
```

### Cost Management
```
Azure Portal → Cost Management + Billing
  ├── Cost Management → Cost Analysis : Spending by service/resource/time
  ├── Cost Management → Budgets       : Budget alerts and thresholds
  └── Billing → Invoices              : Monthly invoice history
```

---

## Azure Portal Search - Fastest Navigation Method

Press `/` or click the search bar at the top of the Azure Portal.
Type a resource name, resource type, or Azure service name to jump directly to it.

Examples:
- Type `vm-win-server` → jumps to the VM resource
- Type `Virtual Machines` → shows all VMs in the subscription
- Type `Network Watcher` → opens Network Watcher directly
- Type `Cost Analysis` → opens the Cost Analysis blade

The portal search also finds documentation and marketplace items - use it first
before navigating through the left menu.

---
