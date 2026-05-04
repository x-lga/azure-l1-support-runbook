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

## Procedure A - Diagnose Connectivity Issues with IP Flow Verify

IP Flow Verify is the fastest diagnostic for NSG-related connectivity issues.
It answers the question: "Is Azure's NSG allowing or blocking this specific traffic?"
without needing to send actual traffic.

```
Azure Portal → Network Watcher → IP Flow Verify

  Subscription        : [your subscription]
  Resource group      : [resource group containing the VM]
  Virtual machine     : [target VM]
  Network interface   : [auto-populated from VM selection]
  Protocol            : TCP or UDP
  Direction           : Inbound or Outbound
  Local IP address    : [VM's private IP - e.g., 10.20.1.4]
  Local port          : [target port - e.g., 3389 for RDP, 443 for HTTPS]
  Remote IP address   : [source IP trying to connect - e.g., your public IP]
  Remote port         : [source port - enter any random port, e.g., 54321]

→ Click Check

Result: "Access allowed" or "Access denied"
If denied: the tool names the specific NSG rule causing the denial
```

**How to use the result:**
- **Access allowed** but connection still fails: the NSG is not the problem.
  Investigate the guest OS firewall (Windows Defender Firewall / UFW on Linux),
  whether the service is running, or routing issues.
- **Access denied** by a specific rule: review that rule and update if appropriate.
  Document the rule name and NSG for the change record.

---

## Procedure B - Check Effective Security Rules

Effective Security Rules shows the complete, merged view of all NSG rules applying
to a specific VM's NIC - combining both the subnet-level NSG and the NIC-level NSG
with all default rules included. This is the definitive view of what traffic is
actually allowed or denied to a VM.

```
Azure Portal → Virtual Machines → [VM Name] →
  Networking → [NIC name] → Effective Security Rules

Shows two sections:
  Inbound security rules  : All inbound rules, ordered by priority
  Outbound security rules : All outbound rules, ordered by priority

Each rule shows:
  Priority | Name | Port | Protocol | Source | Destination | Action | Source NSG
```

**Key difference from viewing the NSG directly:**
When you view the NSG directly, you only see the rules you created. Effective Security
Rules shows ALL rules including the three default rules Azure adds to every NSG:
- `AllowVnetInBound` (priority 65000) - allows all VNet-to-VNet traffic
- `AllowAzureLoadBalancerInBound` (priority 65001) - allows Azure LB health probes
- `DenyAllInBound` (priority 65500) - denies everything else not explicitly allowed

And for outbound:
- `AllowVnetOutBound` (priority 65000) - allows all VNet traffic outbound
- `AllowInternetOutBound` (priority 65001) - allows all internet outbound
- `DenyAllOutBound` (priority 65500) - denies all other outbound traffic

---

## Procedure C - Add or Modify NSG Rules

When IP Flow Verify confirms an NSG is blocking required traffic, update the rule.

**Always follow these steps before modifying an NSG rule:**
1. Confirm the change is authorised (log a Change ticket)
2. Document the current state (screenshot the existing rules)
3. Make the change
4. Verify with IP Flow Verify that the result changed as expected
5. Document the change made in the ticket

**Add an inbound allow rule:**
```
Azure Portal → Network Security Groups → [NSG Name] →
  Inbound Security Rules → + Add

  Source               : [IP Addresses / Service Tag / Any]
  Source IP ranges     : [specific IP or CIDR if IP Addresses selected]
  Source port ranges   : * (leave as wildcard for source port)
  Destination          : Any (or specific IP)
  Service              : [Custom, or select from predefined like RDP, SSH, HTTPS]
  Destination port     : [e.g., 443]
  Protocol             : TCP
  Action               : Allow
  Priority             : [choose a number not already in use - lower = higher priority]
  Name                 : [descriptive name - e.g., Allow-HTTPS-CorpNetwork]
  Description          : [optional but recommended for change history]
```

**Priority guidance:**
- Use priority 100-999 for explicit allow rules
- Use priority 1000-3999 for explicit deny rules
- Use priority 4000-4096 for catch-all deny rules
- Leave gaps between priorities (e.g., 100, 200, 300) so new rules can be inserted
- Do not use priorities 65000-65500 - these are reserved for Azure default rules

---

## Procedure D - Trace a Packet Path with Connection Troubleshoot

Connection Troubleshoot tests end-to-end connectivity between two Azure resources
or between an Azure resource and an external endpoint, identifying where in the
path the issue is.

```
Azure Portal → Network Watcher → Connection Troubleshoot

  Source:
    Resource type      : Virtual Machine
    Virtual machine    : vm-win-server

  Destination:
    Destination type   : IP address or FQDN
    Destination        : 8.8.8.8 (or the target hostname/IP)

  Protocol            : TCP
  Destination port    : 443

→ Check

Results show:
  Status       : Reachable / Unreachable
  Hop count    : Number of hops in the path
  Hops detail  : Each hop with latency and any issues detected
```

**Connection Troubleshoot vs IP Flow Verify:**
- IP Flow Verify only checks NSG rules - it does not test actual connectivity
- Connection Troubleshoot sends real test packets through the network path
- Use IP Flow Verify for NSG-specific investigation
- Use Connection Troubleshoot when you need to test full end-to-end path
  including routing, firewalls, and the destination's responsiveness

---


