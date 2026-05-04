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

### Check 3 - Storage Account Access Keys vs SAS Tokens vs Entra ID

**Access Keys:**
Storage accounts have two access keys (key1 and key2). Anyone with an access key
has full access to all data. Access keys should only be used when necessary and
should be rotated regularly.

```
Azure Portal → Storage Accounts → [Account Name] →
  Security + Networking → Access Keys

If an application is using an expired or rotated key:
  Either update the application's connection string with the new key
  OR rotate to the other key first to give time for the application update
```

**SAS (Shared Access Signature) Token Expired:**
SAS tokens grant time-limited, scoped access to storage. When a SAS token expires,
the application gets 403.

```
Symptoms:
  - Application worked yesterday, failing today with 403
  - Connection string contains "sig=" parameter (indicates SAS token)
  - Error: "Signature not valid" or "AuthenticationFailed"

Resolution:
  Azure Portal → Storage Accounts → [Account] →
    Security + Networking → Shared Access Signature →
    Generate a new SAS token with updated expiry date
    Update the application's connection string/environment variable with the new token
```

---
