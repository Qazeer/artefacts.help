---
title: Microsoft 365 - Exchange workload (Mailbox and ExchangeAdmin audit logs)
summary: 'The Exchange workload groups events from the Mailbox and ExchangeAdmin audit logs.\n\nThe Mailbox audit logs include events on mailbox activity, such as MailboxLogin, MailItemsAccessed (for E5 users), SendAs, SendOnBehalf, MoveToDeletedItems, SoftDelete/HardDelete, etc.\n\nThe ExchangeAdmin audit logs include events on administrative actions, such as Set-Mailbox, New-InboxRule, Add-MailboxPermission, Add-RecipientPermission, etc.'
keywords: Exchange, Unified Audit Logs, UAL, Mailbox, Mailbox Audit Log, MailboxLogin, MailItemsAccessed, FolderBind, MessageBind, Create, Send, SendAs, SendOnBehalf, MoveToDeletedItems, SoftDelete, HardDelete, Set-Mailbox, New-InboxRule, Set-InboxRule, UpdateInboxRules, New-TransportRule, Remove-InboxRule, Disable-InboxRule, Add-MailboxPermission, Add-RecipientPermission, Set-OwaMailboxPolicy
tags:
  - azure_logs
last_updated: 2024-01-28
sidebar: sidebar
permalink: azure_m365_exchange_workload_logs.html
folder: azure
---

The following operations are notable for the `Exchange` workload:

| Source | Operation | Description | Default Scope | Default |
|--------|-----------|-------------|---------------|---------|
| Mailbox audit logs | `MailboxLogin` | The user signed in to their mailbox. | `Owner` | Yes, for `POP3`, `IMAP4`, or `OAuth` logins (and not `NTLM` or `Kerberos` logins to the mailbox). |
| Mailbox audit logs | `MailItemsAccessed` | Access to mail items in the mailbox. <br><br> `MailItemsAccessed` covers both types of accessing mail: `sync`, where clients download sets of messages from the server, and `bind`, where a user or client accesses an individual message. <br><br> `Sync` operations are only recorded when a mailbox is accessed by a desktop version of the Outlook client for Windows or Mac. During the sync operation, the client typically downloads a large set of mail items to the local computer. An audit record is generated only for the mail folder containing items that were synced. All mail items in the synced folder are assumed to be compromised. <br><br> `Bind` operations are individual accesses to email messages. For bind access, the `InternetMessageId` of individual messages is recorded in the audit record. The `MailItemsAccessed` audit action records bind operations and then aggregates them into a single audit record. All bind operations that occur within a 2-minute interval are aggregated in a single audit record in the `Folders` field within the `AuditData` property. Each message that was accessed is identified by its `InternetMessageId`. The number of bind operations that were aggregated in the record is displayed in the `OperationCount` field in the `AuditData` property. It is possible to fully transfer/exfiltrate email messages through `Bind` operations alone. | `Owner`, `Delegate`, `Admin` | Yes, for users with an `E5` license. |
| Mailbox audit logs | `FolderBind` | Access to a mailbox folder. <br><br> Only one audit record is generated for individual folder access within a 24-hour period. | `Delegate`, `Admin` | No |
| Mailbox audit logs | `MessageBind` | Access to a mailbox item. | `Admin` | No |
| Mailbox audit logs | `Create` | Creation of an item in the Calendar, Contacts, Draft, Notes, or Tasks folder. Email creation (sending or receiving) and mailbox folder creation are not audited. | `Owner`, `Delegate`, `Admin` | Yes (for `Owner` since 2026, previously only for `Delegate` and `Admin`). |
| Mailbox audit logs | `Send` | Sending of an email. | `Owner`, `Admin` | Yes, for users with an `E5` license. |
| Mailbox audit logs | `SendAs` | Sending of an email using the `SendAs` permission. | `Delegate`, `Admin` | Yes |
| Mailbox audit logs | `SendOnBehalf` | Sending of an email using the `SendOnBehalf` permission. | `Delegate`, `Admin` | Yes |
| Mailbox audit logs | `MoveToDeletedItems` | Deletion of a message (moved to the `Deleted Items` folder). | `Owner`, `Delegate`, `Admin` | Yes |
| Mailbox audit logs | `SoftDelete` | Soft deletion of a message (deletion from the `Deleted Items` folder, but potentially recoverable from the `Recoverable Items` folder). | `Owner`, `Delegate`, `Admin` | Yes |
| Mailbox audit logs | `HardDelete` | Permanent deletion of a message (message won't be placed in the `Deleted Items` folder or recoverable from the `Recoverable Items` folder). | `Owner`, `Delegate`, `Admin` | Yes |
| Mailbox audit logs | `UpdateInboxRules` | Creation, modification, or removal of mailbox inbox rules from the `Outlook` client (or through the `Exchange Web Services (EWS)` API's `UpdateInboxRules` operation). <br><br> Inbox rules created or modified from `Outlook on the web` (or with `Exchange Online PowerShell`) are logged as `New-InboxRule` / `Set-InboxRule` events in the `ExchangeAdmin` audit logs. | `Owner`, `Delegate`, `Admin` | Yes |
| ExchangeAdmin audit logs | `Set-Mailbox` | Change to the mailbox parameters. Can notably be used to forward emails using the `ForwardingSmtpAddress` parameter. |
| ExchangeAdmin audit logs | `New-InboxRule` | Creation of a new inbox rule in the mailbox. |
| ExchangeAdmin audit logs | `Set-InboxRule` | Modification of an existing inbox rule in the mailbox. |
| ExchangeAdmin audit logs | `New-TransportRule` <br> `Set-TransportRule` <br><br> With the `BlindCopyTo` parameter. | Creation of a Transport / Mail Flow rule to send a copy of the emails to the defined address. |
| ExchangeAdmin audit logs | `Remove-InboxRule` | Removal of a mailbox inbox rule. |
| ExchangeAdmin audit logs | `Disable-InboxRule` | Disabling of a mailbox inbox rule. |
| ExchangeAdmin audit logs | `Add-MailboxPermission` | Update of the permissions associated with the mailbox, such as `FullAccess` or `ChangePermission` permissions (in the `AccessRights` field). |
| ExchangeAdmin audit logs | `Add-RecipientPermission` | Addition of the `SendAs` permission to user(s) for the mailbox. |
| ExchangeAdmin audit logs | `Set-OwaMailboxPolicy` | Update to the OWA mailbox policies. |

The list of actions logged (depending on the logon type), and whether each
action is logged by default, can be found in the
[official Exchange documentation](https://learn.microsoft.com/en-us/microsoft-365/compliance/audit-mailboxes).
