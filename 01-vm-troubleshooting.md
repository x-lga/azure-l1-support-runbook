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

