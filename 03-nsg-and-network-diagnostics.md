# Runbook 03 - NSG and Network Diagnostics

**Category:** Networking - Connectivity and Security Rules
**ITIL 4 Priority:** P2 (single resource affected) | P1 (multiple resources or production service)
**Cert alignment:** AZ-104 - Implement and Manage Virtual Networking
**Last reviewed:** 2026-07

---

## Understanding NSG Rule Evaluation

NSGs are stateful packet filters applied at the subnet or NIC level. Rules are
evaluated in priority order - lowest number first. The first matching rule wins
and no further rules are evaluated.

**Two levels of NSG application:**
- **Subnet NSG:** Applies to all traffic entering/leaving any VM in the subnet
- **NIC NSG:** Applies to traffic to/from a specific VM's network interface

If both subnet and NIC NSGs exist, both are evaluated. For inbound traffic:
subnet NSG is evaluated first, then NIC NSG. For outbound: NIC NSG first, then subnet.

This means even if the subnet NSG allows traffic, a NIC-level NSG can still block it.

---
