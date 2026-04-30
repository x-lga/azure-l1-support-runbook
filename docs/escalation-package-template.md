# Escalation Package Template - Azure L1 to L2

Use this template for every L2 escalation. Copy and fill in all fields before
assigning the ticket to L2. An incomplete escalation package is the primary cause
of L2 needing to re-gather information that L1 already had, doubling resolution time.

---

## Standard Azure Escalation Package

```
══════════════════════════════════════════════════════════════════════
  AZURE L2 ESCALATION PACKAGE
══════════════════════════════════════════════════════════════════════
TICKET ID        : [INC-YYYYMMDD-XXXX]
PRIORITY         : [P1 / P2 / P3]
ESCALATED BY     : [Your name]
ESCALATED AT     : [HH:MM DD/MM/YYYY]
──────────────────────────────────────────────────────────────────────
AFFECTED RESOURCE:
  Resource name  : [e.g., vm-win-server]
  Resource type  : [Virtual Machine / NSG / App Service / Storage Account]
  Resource ID    : [Full Azure resource ID from Portal → Properties]
  Resource Group : [rg-hybrid-lab]
  Region         : [UK South / East US / etc.]
  Subscription   : [Subscription name or ID]
──────────────────────────────────────────────────────────────────────
ISSUE DESCRIPTION:
  [One clear sentence describing the symptom as observed or reported]
──────────────────────────────────────────────────────────────────────
ACTIVITY LOG - MOST RECENT FAILURE:
  Operation name    :
  Operation time    :
  Caller            :
  Error code        :
  Full error message:
──────────────────────────────────────────────────────────────────────
RESOURCE HEALTH (if applicable):
  Status    : [Available / Unavailable - Platform / Unavailable - User / Degraded]
  Message   : [Any health message shown in Portal]
──────────────────────────────────────────────────────────────────────
STEPS TAKEN AND RESULTS:
  [HH:MM] [Step 1 taken] - [Result]
  [HH:MM] [Step 2 taken] - [Result]
  [HH:MM] [Step 3 taken] - [Result]
──────────────────────────────────────────────────────────────────────
CURRENT STATE:
  [Describe exactly what is happening right now - not the original complaint,
  the current state after L1 investigation and steps]
──────────────────────────────────────────────────────────────────────
REASON FOR ESCALATION:
  [Specific reason this cannot be resolved at L1 - not just "unresolved"]
──────────────────────────────────────────────────────────────────────
BUSINESS IMPACT:
  [What service or user is affected and since when]
══════════════════════════════════════════════════════════════════════
```

---

## Azure Support Request - When to Open with Microsoft

Open a support request directly with Microsoft when:
- Azure Resource Health shows platform-initiated unavailability persisting > 30 minutes
- Error is `InternalServerError` and persists despite Stop (Deallocate) + Start
- A quota increase is required and cannot wait for the standard request process
- Data loss or corruption has occurred and recovery support is needed

```
Azure Portal → Help + Support → + Create a support request

Support Type    : Technical
Service         : [Select the affected Azure service]
Issue type      : [Select the issue category]
Subscription    : [Affected subscription]
Problem type    : [Most specific match available]
Problem subtype : [Most specific match available]
Severity        :
  A - Critical  : Production service completely down, no workaround
  B - High      : Production degraded, workaround exists
  C - Minimal   : Non-production or minor issue

Include in the description:
  - Resource ID of the affected resource
  - Full Activity Log error message
  - Steps already attempted
  - Timeline of when the issue started
  - Business impact
```

Microsoft support response times (paid support plans):
- Severity A: < 1 hour initial response
- Severity B: < 4 hours initial response
- Severity C: < 8 business hours initial response


---
