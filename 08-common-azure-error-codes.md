# Common Azure Error Codes - L1 Reference

This reference covers the 20 most frequently encountered Azure error codes in L1
cloud support, including the root cause, what it means in plain language, and the
L1 action to take. For any error not listed here, the Azure Activity Log entry
contains the full error details and the Azure documentation provides resolution steps.

**How to find the full error in Activity Log:**
```
Azure Portal → [Resource] → Activity Log →
  Click the failed operation → Status message field
```

---

## Compute Errors (Virtual Machines)

| Error Code | Plain Language | Root Cause | L1 Action |
|-----------|---------------|-----------|-----------|
| `AllocationFailed` | Azure cannot place your VM on a physical host | No capacity for the requested VM size in this region or zone | Stop (Deallocate) the VM, then Start - this moves it to a different host cluster. If it persists, try a different VM size or region. |
| `SkuNotAvailable` | The VM size you requested does not exist in this region | Not all VM sizes are available in all regions or zones | Choose a different VM size, or choose a different region. Check available sizes: `az vm list-skus --location uksouth --query "[].name"` |
| `QuotaExceeded` | Your subscription has reached the resource limit | Default subscription vCPU or resource quotas have been hit | Delete unused resources, or request a quota increase via Portal → Subscriptions → Usage + Quotas |
| `OperationNotAllowed` | The operation was blocked by a policy | Azure Policy, resource lock, or RBAC restriction is preventing the action | Check Azure Policy assignments for the subscription/RG. Check for resource locks. Check RBAC role - you may not have the required permission. |
| `Conflict` | Another operation is already in progress | Azure cannot run two operations on the same resource simultaneously | Wait 5–10 minutes and retry. If persistent, escalate - may indicate a stuck operation. |
| `BadRequest` | The request had invalid parameters | Malformed deployment template, invalid resource name, or invalid configuration value | Check Activity Log for the specific parameter that was rejected. Fix the parameter and retry. |
| `ResourceNotFound` | The resource specified does not exist | Resource was deleted, wrong name, wrong subscription, or wrong resource group | Verify the resource name and resource group. Check whether the resource exists using Portal search. |
| `RequestDisallowed` | A policy rule blocked this deployment | Azure Policy explicitly prevents this action or configuration | Review Policy assignments in Azure Policy → Compliance. Identify which policy is blocking the action. Escalate to admin to evaluate policy exception. |
| `StorageOperationError` | A VM operation failed due to a storage issue | The managed disk or storage account backing the VM is unhealthy | Check Boot Diagnostics. Escalate to L2 - may require disk repair or snapshot restore. |
| `InternalServerError` | An unexpected Azure platform error | Azure-side failure - transient in most cases | Wait 5 minutes and retry. Check azure.status.microsoft.com for platform incidents. If persistent > 30 minutes, escalate. |

---

## Networking Errors

| Error Code | Plain Language | Root Cause | L1 Action |
|-----------|---------------|-----------|-----------|
| `AuthorizationFailed` | Access denied - RBAC permissions insufficient | The account attempting the action does not have the required role at the required scope | Check RBAC role assignments for the account. Confirm the role has the required action permission. For example, `Contributor` cannot assign roles - `Owner` or `User Access Administrator` is needed. |
| `SubnetIsFull` | No IP addresses available in the subnet | The subnet's address range is exhausted - all IPs are assigned | Add a new subnet to the VNet with an available address range and move resources to the new subnet, or expand the existing subnet if the VNet address space allows. |
| `VnetAddressSpaceConflict` | VNet address ranges overlap | Attempting to create a VNet or subnet with an address range already in use | Choose a non-overlapping CIDR range. Common issue with VNet peering - peered VNets cannot have overlapping address spaces. |
| `InvalidAddressPrefix` | The IP address range is not valid | CIDR notation is incorrect (e.g., 10.0.0.0/33 - /33 does not exist) | Correct the CIDR notation. Valid subnet masks are /0 through /32 for IPv4. |

---

## Identity and Access Errors

| Error Code | Plain Language | Root Cause | L1 Action |
|-----------|---------------|-----------|-----------|
| `RoleAssignmentUpdateNotPermitted` | Cannot update an existing role assignment | Attempting to update a role assignment in place instead of deleting and recreating | Delete the existing role assignment and create a new one with the updated role or scope. |
| `PrincipalNotFound` | The user, group, or service principal does not exist | Wrong object ID, deleted account, or wrong tenant | Verify the object ID in Entra ID. Check whether the account was recently deleted (Entra ID → Deleted Users - 30-day recovery window). |
| `InvalidAuthenticationTokenTenant` | Token is from the wrong Azure AD tenant | The user is authenticated to a different tenant than the one containing the resource | Sign out and sign in again ensuring the correct tenant is selected. In Portal, check the tenant switcher (top right). |

---
