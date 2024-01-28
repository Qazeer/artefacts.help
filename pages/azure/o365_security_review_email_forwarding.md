---
title: Security review - Emails forwarding
summary: 'Emails can be forwarded to external or internal recipients using different mechanisms:\n\n - Mailbox Email Forwarding.\n\n - Mailbox Inbox rules.\n\n - Mailbox Mail Flow / Transport rules (requires Exchange Admin privileges).'
keywords: Email Forwarding, ForwardingSmtpAddress, ForwardingAddress, Inbox rules, Get-InboxRule, Mail Flow, Transport rules, Get-TransportRule
tags:
  - azure_security_review
last_updated: 2024-01-28
sidebar: sidebar
permalink: azure_o365_security_review_email_forwarding.html
folder: azure
---

### Mailbox Email Forwarding

Mailbox Email Forwarding can be configured by any user for their mailbox,
allowing forwarding to external (`ForwardingSmtpAddress`) or internal
(`ForwardingAddress`) email addresses.

```bash
# Lists all mailboxes with a forwarding address configured from Mailbox Settings.
Get-Mailbox -ResultSize Unlimited | Where-Object { ($Null -ne $_.ForwardingSmtpAddress) } | Select Identity,Name,PrimarySmtpAddress,ForwardingSmtpAddress
```

The email forwarding configured on a given mailbox can also be visualized
from the [Microsoft 365 admin center](https://portal.office.com/Adminportal/Home/?#/users) (https://portal.office.com/Adminportal/Home/?#/users).

### Mailbox Inbox rules

Mailbox Inbox rules can be configured by any user for their mailbox, allowing
forwarding to external or internal email addresses as well as deletion of
received emails.

```bash
# List all mailboxes's Inbox rules for the given mailbox.

Get-InboxRule -Mailbox <EMAIL> | Select-Object Identity,Name,Enabled,Description | Format-List

# List all mailboxes's Inbox rules with ForwardTo, RedirectTo, ForwardAsAttachmentTo, or DeleteMessage actions.

$Mailboxes = Get-Mailbox
Foreach ($Mailbox in $Mailboxes) {
    Get-InboxRule -Mailbox $Mailbox.Name |
    Where-Object {($Null -ne $_.ForwardTo) -or ($Null -ne $_.RedirectTo) -or ($Null -ne $_.ForwardAsAttachmentTo) -or ($True -eq $_.DeleteMessage) } |
    Select-Object Identity,Name,PrimarySmtpAddress,Enabled,ForwardAsAttachmentTo,ForwardTo,RedirectTo
}
```

### Mailbox Mail Flow / Transport rules

`Mailbox Mail Flow` / `Transport rules` can only be configured by
`Exchange Admin` users, and allow actions to be taken on in transit emails.

For instance, a blind copy (i.e with out disclosure to the sender or
recipients) can be send to external or internal email addresses.

```bash
# List all Mail Flow / Transport rules.
Get-TransportRule | Select-Object *

# List all Mail Flow / Transport rules that send a copy of received emails.
Get-TransportRule | Where-Object { ($Null -ne $_.BlindCopyTo) } | Select-Object *
```
