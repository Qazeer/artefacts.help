---
title: Security review - Mailbox auditing configuration
summary: 'Mailbox auditing is on by default for the entire Office 365 tenant, but can be turned off.\n\nWhile mailbox auditing cannot be disabled for a specific mailbox if mailbox auditing is enabled tenant-wide, mailbox audit logging can still be bypassed by defined users. In such circumstances, mailbox access and actions, to any mailbox, made by the bypassing account are not logged.'
keywords: Owner, Delegate, Admin, Get-OrganizationConfig, Get-MailboxAuditBypassAssociation
tags:
  - azure_security_review
last_updated: 2024-01-28
sidebar: sidebar
permalink: azure_o365_security_review_mailbox_auditing_configuration.html
folder: azure
---

Mailbox auditing is on by default for the entire `Office 365` tenant, but can
be turned off. Turning off mailbox auditing will result mailbox actions no
longer being audited (even if auditing is enabled on at a mailbox level).
Existing mailbox audit records will however be retained until the audit log age
limit for the record expires.

The following logon types are used to classify the audited actions on a
mailbox:

  - `Owner`: The account that's associated with the mailbox.

  - `Delegate`: A user who's been assigned the `SendAs`, `SendOnBehalf`, or
    `FullAccess` permission to another mailbox.

  - `Admin`: The mailbox is searched with a `Microsoft eDiscovery` tool or
    is accessed with the `Microsoft Exchange Server MAPI Editor`.

While mailbox auditing cannot be disabled for a specific mailbox if mailbox
auditing is enabled tenant-wide, mailbox audit logging can still be bypassed by
defined users.

In such circumstances, mailbox `Owner`, `Delegate`, or `Admin`
access and actions, to any mailbox, made by the bypassing user or computer
account aren't logged.

```bash
Connect-ExchangeOnline

# Retrieves the mailbox auditing status at the Office365 tenant level.
Get-OrganizationConfig | Select-Object Identity,Name,AuditDisabled

# Retrieves the mailbox auditing bypass status for the specified mailbox.
Get-MailboxAuditBypassAssociation -Identity <EMAIL> | Select-Object Id,DistinguishedName,AuditBypassEnabled

# Retrieves mailbox auditing settings, including the operations logged for the specified mailbox.
Get-Mailbox -Identity <EMAIL> | Select-Object Identity,Name,AuditEnabled,DefaultAuditSet,AuditLogAgeLimit,AuditOwner,AuditDelegate,AuditAdmin | ConvertTo-Json
```

[`Microsoft-Extractor-Suite`](./logs_collection_tools.md#azure-ad-office365--azure-microsoft-extractor-suite)'s
[`Get-MailboxAuditStatus`](https://microsoft-365-extractor-suite.readthedocs.io/en/latest/functionality/M365/MailboxAuditStatus.html)
PowerShell cmdlet can be used to automate the retrieval of the mailboxes audit
settings:

```bash
Get-MailboxAuditStatus

Get-MailboxAuditStatus -UserIds "<EMAIL | EMAILS_LIST>"
```
