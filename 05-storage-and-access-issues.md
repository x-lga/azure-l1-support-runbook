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
