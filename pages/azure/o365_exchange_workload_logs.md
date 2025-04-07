---
title: Office365 - Exchange workload (Mailbox and ExchangeAdmin audit logs)
summary: 'The Exchange workload regroup events from the Mailbox and ExchangeAdmin audit logs.\n\nThe Mailbox audit logs include events on mailbox activity, such as MailboxLogin, MailItemsAccessed (for E5 users), SendAs, SendOnBehalf, MoveToDeletedItems, SoftDelete/HardDelete, etc.\n\nThe ExchangeAdmin audit logs include events on administrative actions, such as Set-Mailbox, New-InboxRule, Add-MailboxPermission, Add-RecipientPermission, etc.'
keywords: Exchange, Unified Audit Logs, UAL, Mailbox, Mailbox Audit Log, MailboxLogin, MailItemsAccessed, FolderBind, MessageBind, Create, Send, SendAs, SendOnBehalf, MoveToDeletedItems, SoftDelete, HardDelete, Set-Mailbox, New-InboxRule, Set-InboxRule, UpdateInboxRules, New-TransportRule, Remove-InboxRule, Disable-InboxRule, Add-MailboxPermission, Add-RecipientPermission, Set-OwaMailboxPolicy
tags:
  - azure_logs
last_updated: 2024-01-28
sidebar: sidebar
permalink: azure_o365_exchange_workload_logs.html
folder: azure
---

The following operations are notable for the `Exchange` workload:

| Source | Operation | Description | Default Scope | Default |
|--------|-----------|-------------|---------------|---------|
| Mailbox audit logs | `MailboxLogin` | The user signed in to their mailbox. | `Owner` | Yes, for `POP3`, `IMAP4`, or `OAuth` logins (and not `NTLM` or `Kerberos` logins to the mailbox). |
| Mailbox audit logs | `MailItemsAccessed` | Access to mails in the mailbox. | `Owner`, `Delegate`, `Admin` | Yes, for user with an `E5` license. |
| Mailbox audit logs | `FolderBind` | Access to a mailbox folder. <br><br> Only One audit record is generated for individual folder access within a 24-hour period. | `Delegate`, `Admin` | No |
| Mailbox audit logs | `MessageBind` | Access to a mailbox item. | `Admin` | No |
| Mailbox audit logs | `Create` | Creation of an item in Calendar, Contacts, Notes, or Tasks folder. Email creation is not audited. | `Owner`, `Delegate`, `Admin` | Yes, for `Delegate` and `Admin`. |
| Mailbox audit logs | `Send` | Sending of an email. | `Owner`, `Admin` | Yes, for user with an `E5` license. |
| Mailbox audit logs | `SendAs` | Sending of an email using the `SendAs` permission. | `Delegate`, `Admin` | Yes |
| Mailbox audit logs | `SendOnBehalf` | Sending of an email using the `SendOnBehalf` permission. | `Delegate`, `Admin` | Yes |
| Mailbox audit logs | `MoveToDeletedItems` | Deletion of a message (moved to the `Deleted Items` folder). | `Owner`, `Delegate`, `Admin` | Yes |
| Mailbox audit logs | `SoftDelete` | Soft deletion of a message (deletion from the `Deleted Items` folder, but potentially recoverable from the `Recoverable Items` folder). | `Owner`, `Delegate`, `Admin` | Yes |
| Mailbox audit logs | `HardDelete` | Permanent deletion of a message (message won't be placed in the `Deleted Items` folder or recoverable from the `Recoverable Items` folder). | `Owner`, `Delegate`, `Admin` | Yes |
| ExchangeAdmin audit logs | `Set-Mailbox` | Change to the mailbox parameters. Can notably be used to forward emails using the `ForwardingSmtpAddress` parameter. |
| ExchangeAdmin audit logs | `New-InboxRule` | Creation of a new inbox rule in the mailbox. |
| ExchangeAdmin audit logs | `Set-InboxRule` | Modification of an existing inbox rule in the mailbox. |
| ExchangeAdmin audit logs | `UpdateInboxRules` | Creation or modification of a mailbox inbox rules, typically with the `Outlook Desktop` client using the `Exchange Web Services (EWS)` API. | `Owner`, `Delegate`, `Admin` | Yes |
| ExchangeAdmin audit logs | `New-TransportRule` <br> `Set-TransportRule` <br><br> With the `BlindCopyTo` parameter. | Creation of a Transport / Mail Flow rule to send a copy of the mail to the defined address. |
| ExchangeAdmin audit logs | `Remove-InboxRule` | Removal of a mailbox inbox rule. |
| ExchangeAdmin audit logs | `Disable-InboxRule` | Disabling of a mailbox inbox rule. |
| ExchangeAdmin audit logs | `Add-MailboxPermission` | Update of the permissions associated to the mailbox, such as `FullAccess` or `ChangePermission` permissions (in the `AccessRights` field). |
| ExchangeAdmin audit logs | `Add-RecipientPermission` | Adding of the `SendAs` permission to user(s) for the mailbox. |
| ExchangeAdmin audit logs | `Set-OwaMailboxPolicy` | Update to the OWA mailbox policies. |

The list of actions logged (depending on the logon type) and information on
whether the action is logged by default, can be found in the
[official Exchange documentation](https://learn.microsoft.com/en-us/microsoft-365/compliance/audit-mailboxes).
