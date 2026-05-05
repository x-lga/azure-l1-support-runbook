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

## Procedure B - Blob Container Cannot Be Found (404 Error)

A 404 BlobNotFound or ContainerNotFound error means the specified resource
does not exist at the location specified.

### Check the Container and Blob Path

```
Azure Portal → Storage Accounts → [Account] →
  Data Storage → Containers

Common causes:
  - Container name has incorrect capitalisation (container names are lowercase only)
  - Blob path has trailing or leading slashes that differ from expected
  - Blob was deleted or moved
  - Wrong storage account being queried
```

**Verify the exact blob path:**
```bash
# Using Azure CLI — list blobs in a container
az storage blob list \
    --container-name mycontainer \
    --account-name mystorageaccount \
    --output table

# Check if a specific blob exists
az storage blob exists \
    --container-name mycontainer \
    --name path/to/myfile.pdf \
    --account-name mystorageaccount
```

---

## Procedure C - Azure Files Share Cannot Be Mounted

Azure Files provides SMB-based file shares mountable on VMs and on-premises machines.

### Common Mounting Errors

**Error 53 - Network path not found:**
Port 445 (SMB) is blocked between the machine and Azure Storage.

```powershell
# Test if port 445 is reachable from the machine
Test-NetConnection -ComputerName mystorageaccount.file.core.windows.net -Port 445

# If TcpTestSucceeded = False:
# - Port 445 may be blocked by the corporate firewall
# - ISP may block port 445 (common for residential ISPs)
# Fix: Use Azure File Sync to sync a local server, or use SMB over HTTPS (REST)
```

**Error 13 - Permission denied:**
The storage account key or identity credentials are incorrect.

```powershell
# Mount the Azure File Share with explicit credentials
net use Z: \\mystorageaccount.file.core.windows.net\myshare /user:AZURE\mystorageaccount [AccessKey]

# Or use the mount command from Azure Portal:
Azure Portal → Storage Accounts → [Account] →
  File Shares → [Share Name] → Connect →
  Select OS (Windows/Linux/macOS) →
  Copy the mount script provided
```

**Error: Share not accessible after VM restart:**
The drive mapping was not persisted. Azure File mounts created with `net use` are
not persistent across restarts unless the `/persistent:yes` flag is used.

```powershell
net use Z: \\mystorageaccount.file.core.windows.net\myshare [AccessKey] /persistent:yes
```

For Linux persistence, add the mount to `/etc/fstab`:
```bash
# Linux fstab entry for Azure Files
//mystorageaccount.file.core.windows.net/myshare /mnt/myshare \
    cifs nofail,credentials=/etc/smbcredentials/mystorageaccount.cred,\
    serverino,nosharesock,actimeo=30 0 0
```

---

## Procedure D - Lifecycle Management and Tier Issues

**Blob shows as Archived - Application Cannot Read It:**
Archived blobs cannot be read directly. They must be rehydrated to Hot or Cool tier first.

```
Azure Portal → Storage Accounts → [Account] → Containers →
  [Container] → [Blob file] → Change Tier →
  Select: Hot or Cool →
  Rehydration Priority: Standard (up to 15 hours) or High (within 1 hour, higher cost)

Rehydration is NOT instant. Plan for up to 15 hours for Standard priority.
```

**Storage Costs Higher Than Expected:**
```
Azure Portal → Storage Accounts → [Account] →
  Storage browser → Blob containers →
  Check for large blobs that should have been deleted or moved to a lower tier

Azure Portal → Storage Accounts → [Account] →
  Monitoring → Insights → Overview →
  Check: Capacity used over time, transactions by tier
```

Set up Lifecycle Management to automate tier transitions:
```
Azure Portal → Storage Accounts → [Account] →
  Data Management → Lifecycle Management → Add Rule

Example rule:
  Name: move-to-cool-after-30-days
  Blob type: Block blobs
  Conditions:
    Base blobs: Last modified more than 30 days ago → Move to Cool
    Base blobs: Last modified more than 90 days ago → Move to Archive
    Base blobs: Last modified more than 365 days ago → Delete
```


---
