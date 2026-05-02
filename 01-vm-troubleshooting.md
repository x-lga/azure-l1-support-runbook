# Runbook 01 - Azure Virtual Machine Troubleshooting

**Category:** Compute - Virtual Machine Availability
**ITIL 4 Priority:** P2 (single non-critical VM) | P1 (production VM with service impact)
**SLA - P1 Response:** 15 minutes | **P1 Resolution:** 4 hours
**SLA - P2 Response:** 1 hour | **P2 Resolution:** 8 hours
**Cert alignment:** AZ-104 - Deploy and Manage Azure Compute Resources
**Last reviewed:** 2026-07

---

## Understanding the Azure VM Lifecycle Before You Start

Before running a single troubleshooting step, understand the four VM power states
and what each means for how you respond:

| Power State | What It Means | Billing | Notes |
|------------|--------------|---------|-------|
| Running | VM is powered on, OS is booted | Yes - full compute charge | Normal operational state |
| Stopped (OS-level) | Shutdown within the OS (shutdown /s) | Yes - still allocated on a host | Dangerous - billed but not usable |
| Stopped (Deallocated) | VM released from host via Azure Portal Stop | No compute charge | Correct way to stop a VM to save cost |
| Starting | VM is being allocated and booted | Yes | Transitional state |

**The most common billing mistake:** Users power off the VM from within Windows
(shut down) rather than using the Azure Portal Stop button. The VM shows as "Stopped"
but is NOT deallocated - it is still sitting on a physical Azure host and still
being billed. The Portal will show the power state as "Stopped" and you may see
this in a support ticket as "I stopped the VM but I'm still being charged."

---

## Procedure A - VM Will Not Start

This is the most common VM support ticket. The user cannot connect to the VM or the
VM shows as Failed, Stopped, or in an error state.

### Step 1 - Read the Portal Status and Error Message

```
Azure Portal → Virtual Machines → [VM Name] → Overview

Check:
  Status field         : Should show "Running"
  Any banner messages  : Read these carefully — they often contain the specific error
  Provisioning state   : Should show "Succeeded"
  Availability zone    : Note this for your escalation package
```

The status field and banner messages are the first and fastest diagnostic. Do not
skip this step and go straight to Activity Log - the banner often tells you exactly
what is wrong.

**Common status messages and what they mean:**

| Status Message | Meaning | L1 Action |
|---------------|---------|-----------|
| Running | VM is operational | Investigate connectivity, not VM state |
| Stopped (deallocated) | VM was shut down correctly | Start the VM via Portal → Overview → Start |
| Stopped | VM shut down from OS - NOT deallocated | Portal → Stop first, then Start (to fully deallocate and reallocate) |
| Failed | Last operation failed | Check Activity Log for the specific error |
| Creating | VM is being deployed | Wait and recheck in 5 minutes |
| Updating | A management operation is in progress | Wait - do not attempt other operations |

---

### Step 2 - Check the Activity Log

The Activity Log records every control-plane operation on the VM — starts, stops,
restarts, configuration changes, and failures. It is the most reliable source of
the exact error that caused the failure.

```
Azure Portal → Virtual Machines → [VM Name] → Activity Log

Filter:
  Time range : Last 1 hour (extend to 24 hours if issue is not recent)
  Level      : Error (to find failures immediately)

Click any failed operation (shown with a red X) to expand:
  → Status    : The top-level result (Failed)
  → Status message : The detailed error including the specific error code
```

**Example Activity Log entry for AllocationFailed:**
```json
{
  "status": "Failed",
  "error": {
    "code": "AllocationFailed",
    "message": "Allocation failed. We do not have sufficient capacity
                for the requested VM size in this region."
  }
}
```

This tells you exactly what is wrong and points you to the correct resolution
(deallocate and restart to move to a different host cluster).

---

### Step 3 - Check Resource Health

Resource Health tells you whether Azure's own infrastructure is causing the issue —
separate from any configuration problem on your VM.

```
Azure Portal → Virtual Machines → [VM Name] → Help → Resource Health
```

| Resource Health Status | Meaning | L1 Action |
|-----------------------|---------|-----------|
| Available | No platform issue - investigate VM configuration | Continue triage |
| Unavailable - Platform initiated | Azure infrastructure issue affecting this VM | Document, do not escalate to L2 - open Azure support ticket if persistent |
| Unavailable - User initiated | A user action caused the unavailability | Check Activity Log for the user action |
| Degraded | Performance issue - VM is running but underperforming | Check diagnostics, possibly escalate |
| Unknown | Health status cannot be determined | Check again in 5 minutes; may be a transient state |

If Resource Health shows **Platform initiated unavailability**: this is Azure's
problem, not yours. Document the Resource Health status and timestamp, and wait.
If it persists beyond 30 minutes, open an Azure support request with Microsoft.

---


