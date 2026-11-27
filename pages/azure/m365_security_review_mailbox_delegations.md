---
title: Security review - Mailbox delegations
summary: 'The following levels/scopes of mailbox delegation can be configured:\n\n - Mailbox permissions: to allow viewing items at the mailbox level (but not the right to send emails).\n\n - Recipient SendAs permissions: to delegate the right to send emails from the mailbox (which transparently appear to the recipients to come from the specified mailbox).\n\n - SendOnBehalf permissions (GrantSendOnBehalfTo mailbox property): to delegate the right to send emails on behalf of the mailbox (which will appear as such to the recipients).\n\n - Folder-level permissions: to delegate the rights to interact with items at the mailbox''s folder level.'
keywords: Mailbox permissions, Recipient permissions, SendAs, SendOnBehalf, GrantSendOnBehalfTo, Folder-level permissions, ChangeOwner, ChangePermission, DeleteItem, ExternalAccount, FullAccess, ReadPermission
tags:
  - azure_security_review
  - guides
last_updated: 2024-01-28
sidebar: sidebar
permalink: azure_m365_security_review_mailbox_delegations.html
folder: azure
---

### Overview

The following levels/scopes of mailbox delegation can be configured:

  - Mailbox permissions: to allow viewing items at the
    mailbox level (but not the right to send emails).

  - Recipient `SendAs` permissions: to delegate the right to send emails from
    the mailbox (which transparently appear to the recipients to come from the
    specified mailbox).

  - `SendOnBehalf` permissions (`GrantSendOnBehalfTo` mailbox property): to
    delegate the right to send emails on behalf of the mailbox (which will
    appear as such to the recipients).

  - Folder-level permissions: to delegate the rights to interact with items at
    the mailbox's folder level.

### Mailbox access rights/permissions

Mailboxes are securable objects with a set of possible access rights /
permissions.

The available access rights are:

  - `ChangeOwner`: change the owner of the mailbox.

  - `ChangePermission`: change the permissions on the mailbox.

  - `DeleteItem`: delete the mailbox.

  - `ExternalAccount`: indicates the account isn't in the same domain.

  - `FullAccess`: open the mailbox, access its contents, but can't send mail.

  - `ReadPermission`: read the permissions on the mailbox.

The permissions defined at that level allow viewing emails at the mailbox
scope but do not allow sending emails (which is defined through the recipient
`SendAs` permission and the `SendOnBehalf` permission / `GrantSendOnBehalfTo`
mailbox property).

```bash
# Retrieves the access rights defined on the given mailbox.
Get-MailboxPermission -Identity <EMAIL>

# Lists the mailbox permissions with FullAccess, ChangeOwner, ChangePermission, or ExternalAccount access rights.
Get-Mailbox -Resultsize Unlimited | Get-MailboxPermission | Where-Object { ($_.Accessrights -like "FullAccess" -or $_.Accessrights -like "ChangeOwner" -or $_.Accessrights -like "ChangePermission" -or $_.Accessrights -like "ExternalAccount") } | Format-List
```

### Recipient (or `SendAs`) access right/permission

The Recipient, or `SendAs`, permission does not allow viewing emails, but
allows a user, or the members of a group, to send messages that appear to come
from the specified mailbox. Emails received from the mailbox owner or through a
`SendAs` delegation are indistinguishable to the receiving end user.

Note that the `Get-EXORecipientPermission`/`Get-RecipientPermission` cmdlets
are not included by default in the cmdlets allowed for the
`View-Only Organization Management` role group.

```bash
# Lists the mailboxes with the SendAs recipient permission.
Get-Mailbox -Resultsize Unlimited | Get-EXORecipientPermission | Where-Object { ($_.Accessrights -like "SendAs") }
```

### SendOnBehalf permission

The `SendOnBehalf` permission is not a recipient permission, but is defined
through the `GrantSendOnBehalfTo` property of the mailbox (configured with the
`Set-Mailbox` cmdlet). The `SendOnBehalf` permission allows a user, or the
members of a mail-enabled security group, to send messages on behalf of the
mailbox. Contrary to the `SendAs` permission, the emails sent clearly show the
sender in the `From` field (`<SENDER> on behalf of <MAILBOX>`).

```bash
# Lists the mailboxes with the SendOnBehalf permission granted to user(s) or group(s).
Get-Mailbox -ResultSize Unlimited | Where-Object { ($_.GrantSendOnBehalfTo.Count -gt 0) } | Select-Object Identity,PrimarySmtpAddress,GrantSendOnBehalfTo
```

### Folder-level permissions

Permissions/access rights can also be defined at the folder level in
mailboxes, to delegate the rights to interact with items at the
mailbox's folder level.

The [following individual permissions are available](https://learn.microsoft.com/en-us/powershell/module/exchange/add-mailboxfolderpermission):

  - `None`: The user has no access to view or interact with the folder or its
    contents.

  - `CreateItems`: The user can create items within the specified folder.

  - `CreateSubfolders`: The user can create subfolders in the specified folder.

  - `DeleteAllItems`: The user can delete all items in the specified folder.

  - `DeleteOwnedItems`: The user can only delete items that they created from
    the specified folder.

  - `EditAllItems`: The user can edit all items in the specified folder.

  - `EditOwnedItems`: The user can only edit items that they created in the
    specified folder.

  - `FolderContact`: The user is the contact for the specified public folder.

  - `FolderOwner`: The user is the owner of the specified folder. The user can
    view the folder, move the folder and create subfolders. The user can't read
    items, edit items, delete items or create items.

  - `FolderVisible`: The user can view the specified folder, but can't read or
    edit items within the specified public folder.

  - `ReadItems`: The user can read items within the specified folder.

The following roles, which group individual permissions, are available:

  - `Author`: CreateItems, DeleteOwnedItems, EditOwnedItems, FolderVisible,
    ReadItems.

  - `Contributor`: CreateItems, FolderVisible.

  - `Editor`: CreateItems, DeleteAllItems, DeleteOwnedItems, EditAllItems,
    EditOwnedItems, FolderVisible, ReadItems.

  - `NonEditingAuthor`: CreateItems, DeleteOwnedItems, FolderVisible,
    ReadItems.

  - `Owner`: CreateItems, CreateSubfolders, DeleteAllItems, DeleteOwnedItems,
    EditAllItems, EditOwnedItems, FolderContact, FolderOwner, FolderVisible,
    ReadItems.

  - `PublishingAuthor`: CreateItems, CreateSubfolders, DeleteOwnedItems,
    EditOwnedItems, FolderVisible, ReadItems.

  - `PublishingEditor`: CreateItems, CreateSubfolders, DeleteAllItems,
    DeleteOwnedItems, EditAllItems, EditOwnedItems, FolderVisible, ReadItems.

  - `Reviewer`: FolderVisible, ReadItems.

The `Get-MailboxFolderPermission` cmdlet retrieves the permissions of a single
folder, specified using the `<MAILBOX>:\<FOLDER_PATH>` syntax (for example
`<EMAIL>:\Calendar` or `<EMAIL>:\Inbox`). If only the mailbox is specified,
only the permissions of the root folder of the mailbox
(`Top of Information Store`) are returned. As the folder names are localized
(depending on the language of the mailbox), the folders can be enumerated with
the `Get-MailboxFolderStatistics` cmdlet, and their `FolderId` used to retrieve
their permissions (`<MAILBOX>:<FOLDER_ID>`).

```bash
# Retrieves the folder-level permissions of the root folder, or of the given folder, of the specified mailbox.
Get-MailboxFolderPermission -Identity <EMAIL> | Select-Object *
Get-MailboxFolderPermission -Identity <EMAIL>:\<Calendar | Inbox | FOLDER_PATH> | Select-Object *

# Retrieves the folder-level permissions of all the folders of the specified mailbox.
Get-MailboxFolderStatistics -Identity <EMAIL> | ForEach-Object { Get-MailboxFolderPermission -Identity "<EMAIL>:$($_.FolderId)" -ErrorAction SilentlyContinue } | Select-Object FolderName,User,AccessRights

# Enumerates, for all the mailboxes and all their folders, the folder-level permissions allowing access to Anonymous or Default.
# As each folder of each mailbox is queried, the enumeration may take a long time on large tenants.
$MailBoxes = Get-Mailbox -Resultsize Unlimited
ForEach ($MailBox in $MailBoxes) {
  $Folders = Get-MailboxFolderStatistics -Identity $MailBox.PrimarySmtpAddress
  ForEach ($Folder in $Folders) {
    $Permissions = Get-MailboxFolderPermission -Identity "$($MailBox.PrimarySmtpAddress):$($Folder.FolderId)" -ErrorAction SilentlyContinue |
    Where-Object { (($_.User -like 'Anonymous') -or ($_.User -like 'Default')) -and $_.AccessRights -ne 'None' }

    ForEach ($Permission in $Permissions) {
      [PSCustomObject]@{
        MailBoxIdentity = $MailBox.Identity
        MailBoxPrimarySmtpAddress = $MailBox.PrimarySmtpAddress
        FolderName = $Permission.FolderName
        FolderPath = $Folder.FolderPath
        DelegateUser = $Permission.User
        DelegateRights = $Permission.AccessRights
        DelegateIsValid = $Permission.IsValid
      }
    }
  }
}
```

### Automated collection

[`Microsoft-Extractor-Suite`](./logs_collection_tools.md#entra-id-microsoft-365--azure-microsoft-extractor-suite)'s
[`Get-MailboxPermissions`](https://microsoft-365-extractor-suite.readthedocs.io/en/latest/functionality/M365/MailboxDelegatedPermissions.html)
PowerShell cmdlet can be used to enumerate the Mailbox (`FullAccess`),
Recipient (`SendAs`), and `SendOnBehalf` access rights / permissions, as well
as the folder-level permissions of the `Calendar` and `Inbox` folders:

```bash
Get-MailboxPermissions

Get-MailboxPermissions -UserIds "<EMAIL | EMAILS_LIST>"
```
