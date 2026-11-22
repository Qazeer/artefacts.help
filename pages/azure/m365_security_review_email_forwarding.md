---
title: Security review - Email forwarding
summary: 'Emails can be forwarded to external or internal recipients using different mechanisms:\n\n - Mailbox Email Forwarding.\n\n - Mailbox Inbox rules.\n\n - Mailbox Mail Flow / Transport rules (requires Exchange Admin privileges).'
keywords: Email Forwarding, ForwardingSmtpAddress, ForwardingAddress, DeliverToMailboxAndForward, Inbox rules, Get-InboxRule, Mail Flow, Transport rules, Get-TransportRule
tags:
  - azure_security_review
last_updated: 2024-01-28
sidebar: sidebar
permalink: azure_m365_security_review_email_forwarding.html
folder: azure
---

### Mailbox Email Forwarding

Mailbox Email Forwarding can be configured through two mailbox properties:

  - `ForwardingSmtpAddress`: forwarding to any SMTP email address (external or
    internal). This property can be configured by any user for their mailbox
    (from `Outlook on the web`: `Settings` -> `Mail` -> `Forwarding`), or by
    administrators (from the `Exchange admin center`, the
    `Microsoft 365 admin center`, or with the `Set-Mailbox` cmdlet).

  - `ForwardingAddress`: forwarding to a recipient object defined in the
    directory (another mailbox, a mail user, or a mail contact, which can
    reference an external email address). This property can only be configured
    by administrators (from the `Exchange admin center` or with the
    `Set-Mailbox` cmdlet).

If both properties are configured, the `ForwardingSmtpAddress` property is
ignored and the emails are forwarded to the `ForwardingAddress` recipient. The
`DeliverToMailboxAndForward` property defines whether the emails are delivered
to the mailbox and forwarded (`True`), or only forwarded (`False`, by default).

```bash
# Lists all mailboxes with a forwarding address configured from Mailbox Settings.
Get-Mailbox -ResultSize Unlimited | Where-Object { ($Null -ne $_.ForwardingSmtpAddress) -or ($Null -ne $_.ForwardingAddress) } | Select-Object Identity,Name,PrimarySmtpAddress,ForwardingSmtpAddress,ForwardingAddress,DeliverToMailboxAndForward
```

The email forwarding configured on a given mailbox can also be visualized
from the [Microsoft 365 admin center](https://portal.office.com/Adminportal/Home/?#/users) (https://portal.office.com/Adminportal/Home/?#/users).

### Mailbox Inbox rules

Mailbox Inbox rules can be configured by any user for their mailbox, allowing
forwarding to external or internal email addresses as well as deletion of
received emails.

```bash
# Lists the Inbox rules of the given mailbox.

Get-InboxRule -Mailbox <EMAIL> | Select-Object Identity,Name,Enabled,Description | Format-List

# Lists the Inbox rules of the given mailbox with ForwardTo, RedirectTo, ForwardAsAttachmentTo, or DeleteMessage actions.

$Mailboxes = Get-Mailbox -ResultSize Unlimited
Foreach ($Mailbox in $Mailboxes) {
    Get-InboxRule -Mailbox $Mailbox.Name |
    Where-Object {($Null -ne $_.ForwardTo) -or ($Null -ne $_.RedirectTo) -or ($Null -ne $_.ForwardAsAttachmentTo) -or ($True -eq $_.DeleteMessage) } |
    Select-Object Identity,Name,PrimarySmtpAddress,Enabled,ForwardAsAttachmentTo,ForwardTo,RedirectTo,DeleteMessage
}
```

[`Microsoft-Extractor-Suite`](./logs_collection_tools.md#azure-ad-microsoft-365--azure-microsoft-extractor-suite)'s
[`Show-MailboxRules`](https://microsoft-365-extractor-suite.readthedocs.io/en/latest/functionality/M365/InboxRules.html)
PowerShell cmdlet can be used to enumerate the Mailbox Inbox rules:

```bash
Show-MailboxRules

Show-MailboxRules -UserIds "<EMAIL | EMAILS_LIST>"
```

### Mailbox Mail Flow / Transport rules

`Mailbox Mail Flow` / `Transport rules` can only be configured by
`Exchange Admin` users and allow actions to be taken on in-transit emails.

For instance, a blind copy (i.e. without disclosure to the sender or
recipients) can be sent to external or internal email addresses.

```bash
# Lists all Mail Flow / Transport rules.
Get-TransportRule | Select-Object *

# Lists all Mail Flow / Transport rules that send a copy of received emails.
Get-TransportRule | Where-Object { ($Null -ne $_.BlindCopyTo) } | Select-Object *
```

[`Microsoft-Extractor-Suite`](./logs_collection_tools.md#azure-ad-microsoft-365--azure-microsoft-extractor-suite)'s
[`Show-TransportRules`](https://microsoft-365-extractor-suite.readthedocs.io/en/latest/functionality/M365/TransportRules.html)
PowerShell cmdlet can be used to enumerate the `Transport rules` with
output formatting:

```bash
Show-TransportRules
```
