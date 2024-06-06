---
title: Security review - Mailbox delegations
summary: 'The following level / scope of mailbox delegations can be configured:\n\n - Mailbox permissions: to allow items viewing at the mails box level (but not the right to send emails).\n\n - Recipient SendAs permissions: to delegate the right to send emails from the mailbox (that transparently appear to come from the specified mailbox to the recipients).\n\n - Recipient SendOnBehalf permissions: to delegate the right to send emails on behalf of the mailbox (and will appear as such to the receiving recipients).\n\n - Folder-level permissions: to delegate the rights to interact with items at the mailboxes folder level.'
keywords: Mailbox permissions, Recipient permissions, SendAs, SendOnBehalf, Folder-level permissions, ChangeOwner, ChangePermission, DeleteItem, ExternalAccount, FullAccess, ReadPermission
tags:
  - azure_security_review
last_updated: 2024-01-28
sidebar: sidebar
permalink: azure_o365_security_review_mailbox_delegations.html
folder: azure
---

### Overview

The following level / scope of mailbox delegations can be configured:

  - Mailbox permissions: to allow items viewing at the mails
    box level (but not the right to send emails).

  - Recipient `SendAs` permissions: to delegate the right to send emails from
    the mailbox (that transparently appear to come from the specified mailbox
    to the recipients).

  - Recipient `SendOnBehalf` permissions: to delegate the right to send emails
    on behalf of the mailbox (and will appear as such to the receiving
    recipients).

  - Folder-level permissions: to delegate the rights to interact with items at
    the mailbox's folder level.

### Mailbox access rights / permissions

Mailboxes are securable objects with a set of possible access rights /
permissions.

The available access rights are:

  - `ChangeOwner`: change the owner of the mailbox.

  - `ChangePermission`: change the permissions on the mailbox.

  - `DeleteItem`: delete the mailbox.

  - `ExternalAccount`: indicates the account isn't in the same domain.

  - `FullAccess`: open the mailbox, access its contents, but can't send mail.

  - `ReadPermission`: read the permissions on the mailbox.

The permissions defined on that level allow for emails viewing at the mailbox
scope but do not allow sending emails (which is defined through the recipient
`SendAs` and `SendOnBehalf` permissions).

```bash
# Retrieves the access rights defined on the given mailbox.
Get-MailboxPermission -Identity <EMAIL>

# Lists the mailbox permissions with Full Access, ChangeOwner, ChangePermission, or ExternalAccount access rights.
Get-Mailbox -Resultsize Unlimited | Get-MailboxPermission | Where-Object { ($_.Accessrights -like "FullAccess" -or $_.Accessrights -like "ChangeOwner" -or $_.Accessrights -like "ChangePermission" -or $_.Accessrights -like "ExternalAccount") } | Format-List
```

### Recipient (or `SendAs`) access right / permission

Recipient, or `SendAs`, permission does not allow for emails viewing but
allow a user, or group members, to send messages that appear to come from the
specified mailbox. The email received from the mailbox owner or through a
`SendAs` delegation are indistinguishable by the receiving end-user.

Note that the `Get-EXORecipientPermission` / `Get-RecipientPermission` is not
included by default in the cmdlets allowed for the
`View-Only Organization Management` role group.

```bash
# Lists the mailboxes with the SendAs reciptient permission.
Get-Mailbox -Resultsize Unlimited | Get-EXORecipientPermission | Where-Object { ($_.Accessrights -like "SendAs") }
```

### Folder-level permissions

Permissions / access rights can also be defined at the folder-level in
mailboxes, to grant delegate the rights to interact with items at the
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

The following roles, that group individual permissions, are available:

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

```bash
# Retrieves the folder-level permission for the specified mailbox.
Get-Mailbox -Identity <EMAIL> | Get-MailboxFolderPermission | Select-Object *

# Enumerates, for all the mailboxes, the folder-level permission allowing access to Anonymous or Default.
$MailBoxes = Get-Mailbox -Resultsize Unlimited
ForEach ($MailBox in $MailBoxes) {
  $Permissions = Get-MailboxFolderPermission $MailBox |
  Where-Object { (($_.User -like 'Anonymous') -or ($_.User -like 'Default')) -and $_.AccessRights -ne 'None' }

  ForEach ($Permission in $Permissions) {
    [PSCustomObject]@{
      MailBoxIdentity = $MailBox.Identity
      MailBoxPrimarySmtpAddress = $MailBox.PrimarySmtpAddress
      FolderName = $Permission.FolderName
      DelegateUser = $Permission.User
      DelegateRights = $Permission.AccessRights
      DelegateIsValid = $Permission.IsValid
    }
  }
}
```
