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

