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
