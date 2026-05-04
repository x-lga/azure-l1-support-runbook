# Runbook 05 - Azure Storage and Access Issues

**Category:** Storage - Blob, File Shares, Access Keys, SAS Tokens
**ITIL 4 Priority:** P2 (application cannot access data) | P1 (data unavailability with service impact)
**Cert alignment:** AZ-104 - Implement and Manage Storage
**Last reviewed:** 2026-07

---

## Azure Storage Types - Quick Reference

| Storage Type | Use Case | Access Method |
|-------------|---------|--------------|
| Blob Storage | Unstructured data (images, backups, logs, videos) | REST API, Storage Explorer, AzCopy |
| File Share | Shared network drive - mountable on VMs and on-prem | SMB protocol, NFS (Premium tier) |
| Queue Storage | Message queuing between application components | REST API |
| Table Storage | NoSQL key-value store for structured data | REST API |
| Azure Data Lake Storage Gen2 | Big data analytics - hierarchical namespace on Blob | REST API, Hadoop, Spark |

---

## Procedure A - Storage Account Access Denied (403 Error)

A 403 AuthorizationFailure or AuthorizationPermissionMismatch error means the
request reached Azure Storage but was rejected due to insufficient permissions.

### Check 1 - RBAC Assignments

For the application, service principal, or user account trying to access storage:
```
Azure Portal → Storage Accounts → [Account Name] →
  Access Control (IAM) → Role Assignments

Verify the identity has one of:
  Storage Blob Data Reader    : Read-only access to blob data
  Storage Blob Data Contributor: Read/write/delete blob data
  Storage Blob Data Owner     : Full control including setting ACLs
  Storage Account Contributor : Manage the storage account (not the data)
```

**Common mistake:** Assigning `Storage Account Contributor` or `Contributor` at
the resource group level does NOT grant access to read or write blob data.
These roles manage the storage account configuration, not the data plane.
For blob data access, you must use the Storage Blob Data roles.


### Check 2 - Storage Account Firewall

If the storage account has a firewall configured, only allowed IPs or VNets can
access it. Applications accessing from an unexpected IP will get 403.

```
Azure Portal → Storage Accounts → [Account Name] →
  Security + Networking → Networking

Check:
  Public network access:
    "Enabled from all networks" : No firewall - anyone with credentials can access
    "Enabled from selected virtual networks and IP addresses": Firewall active
    "Disabled": All public access blocked - only Azure Private Endpoints

If firewall is enabled:
  Check that the application's source IP or VNet is in the allowed list
  Check that "Allow Azure services on the trusted services list" is enabled
  if an Azure service (VM, Function App, App Service) needs access
```

**Add a VNet to the storage firewall:**
```
Azure Portal → Storage Account → Networking → Firewalls and virtual networks →
  Add existing virtual network →
  [Select vnet-hybrid-lab and subnet-servers]
  → Save
```
