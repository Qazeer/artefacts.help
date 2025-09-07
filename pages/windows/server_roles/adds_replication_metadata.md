---
title: Active Directory Domain Services (Domain Controllers) replication metadata
summary: 'The Active Directory replication metadata hold information about change made on an Active Directory object. The replication metadata is used by the Domain Controllers to replicate modifications and, as so, only attributes that are replicated will be logged in the replication metadata.\n Every object within Active Directory stores replication metadata, in their "msDS-ReplAttributeMetaData" (for regular attributes) and "msDS-ReplValueMetaData" (for linked attributes) attributes.'
keywords: Active Directory, Active Directory Domain Services, Domain Controllers, DC, replication metadata, msDS-ReplAttributeMetaData, msDS-ReplValueMetaData, pszAttributeName, dwVersion, ftimeLastOriginatingChange, pszLastOriginatingDsaDN, uuidLastOriginatingDsaInvocationID, ftimeCreated, ftimeDeleted, lastLogonTimestamp, lastLogon, repadmin, Get-ADReplicationAttributeMetadata, ADTimeline, FarsightAD, esentutl, dsamain.exe
tags:
  - windows_active_directory
location: 'Replicated attributes of interest: - adminCount\n - lastLogonTimestamp\n - member\n - msDS-AllowedToDelegateTo\n - nTSecurityDescriptor\n - primaryGroupID\n - scriptPath\n - servicePrincipalName\n - sIDHistory\n - userPrincipalName'
last_updated: 2024-09-25
sidebar: sidebar
permalink: adds_replication_metadata.html
folder: windows
---

### Overview

The Active Directory replication metadata hold information about change made
on an Active Directory object. The replication metadata is used by the Domain
Controllers to replicate modifications and, as so, only attributes that are
replicated will be logged in the replication metadata. As stated in the
Microsoft documentation: "every attribute in [the Active Directory] `Schema`
has a flag called `FLAG_ATTR_NOT_REPLICATED`. If this flag value is `True (1)`,
that attribute is not included in AD Replication. If the attribute value is
`False (0)`, then that attribute is replicated". Upon creation of an object,
all replicated attributes that are automatically populated will be logged in
the replication metadata.

Every object within Active Directory stores replication metadata, in their
`msDS-ReplAttributeMetaData` and `msDS-ReplValueMetaData` attributes.
The `msDS-ReplAttributeMetaData` attribute stores replication metadata for
regular replicated attributes, while the `msDS-ReplValueMetaData` attribute
stores replication metadata for `linked attributes`.

The replication metadata is stored in the `msDS-ReplAttributeMetaData`
attribute as a XML with the following notable fields:
  - `pszAttributeName`: the name of the attribute replicated.
  - `dwVersion`: a replication counter, incremented upon each modification of
    the associated attribute.
  - `ftimeLastOriginatingChange`: last change timestamp.
  - `pszLastOriginatingDsaDN`: Domain Controller from which originated the last
    change. More precisely, the distinguished name of the `NTDS Settings`
    (type `Domain Controller Settings`) of the Domain Controller.
  - `uuidLastOriginatingDsaInvocationID`: `invocationId` of the Domain
    Controller (stored in the Domain Controller 's `NTDS Settings`).

As stated, the `msDS-ReplValueMetaData` attribute hold replication metadata for
the `linked attributes`, which were introduced in Windows Server 2003
functional level to reduce replication data. `Linked attributes` are pairs of
two attributes, with the values of one attribute, denominated the `back link`,
being based on the values set on the other attribute, denominated the `forward
link. Only the `forward link` attributes are replicated, which help reduce
replication metadata volume.

For instance, the groups's `member` attribute is the forward link for the
user and computer accounts' `memberOf` attribute. The `member` attribute of
group objects holds, in addition to the fields introduced above, information on
when the principal was added (`ftimeCreated` field) or deleted (`ftimeDeleted`
field) from the group.

### Replication metadata of interest

The following replicated attribute (in a default configuration) could be of
interest for digital forensics and incident response purposes.

| Attribute | Object(s) | Description |
|-----------|-----------|-------------|
| `adminCount` | User accounts or groups. | If `adminCount` equal `1`, successful elevation of privileges, that is adding of the object in a group protected by the `AdminSDHolder` mechanism. <br><br> If `adminCount` is set to `0`, potential manual concealing of previously obtained privileges, after removal of the object from a privileged group protected by the `AdminSDHolder` mechanism. |
| `lastLogonTimestamp` | Security principals. | Refer to `lastLogon v. lastLogonTimestamp` below for more information. |
| `member` | Groups. | Information on when a given principal was added or removed from the group. <br><br> While the list of present principals is exhaustive, the replication metadata of removed principal will only persist for the `tombstone lifetime` of the Active Directory domain. <br><br> An odd `dwVersion` value means that the user is still a member of the group while an even `dwVersion` value means that the user was removed from the group. |
| `msDS-AllowedToDelegateTo` <br><br> `msDS-AllowedToActOnBehalfOfOtherIdentity` | User or computer accounts. | Setting or modification of Kerberos `constrained delegation` or `resource-based constrained delegation (RBCD)`. |
| `nTSecurityDescriptor` | Any securable objects: <br> user or computer accounts, groups, `Organizational Unit (OU)`, `AdminSDHolder` container, etc. | Modification of the security descriptor of a securable object, linked to a potential change of ownership or `DACL` modification. |
| `primaryGroupID` | User or computer accounts. | Modification of an account `primaryGroupID` attribute, potentially linked to persistence purposes. |
| `scriptPath` | User accounts. | Modification of an user's logon script. |
| `servicePrincipalName` | User or computer accounts. | Setting or modification of an user or computer account's `Service Principal Name (SPN)`. |
| `sIDHistory` | Security principals. | Setting or modification of a security principal `SID History`, potentially for persistence or trusts hopping purposes. |
| `userPrincipalName` | User or computer accounts. | Setting or modification of an user or computer account's `userPrincipalName`, potentially related to certificate-authentication attacks.  |
| ... | ... | ... |

#### lastLogon v. lastLogonTimestamp

Every account in Active Directory, be it an user or computer account, has both
a `lastLogon` and a `lastLogonTimestamp` attributes.

The `lastLogon` attribute of an account is immediately updated upon a
successful authentication of the account. The `lastLogon` attribute is however
(by default) not replicated, and is thus only updated on the Domain Controller
that provided the authentication.

On the other hand, the `lastLogonTimestamp` attribute of an account is
replicated by default but will only be updated if the difference between the
previous value and new value is greater than the default naming context's
`ms-DS-Logon-Time-Sync-Interval` attribute. By default the value for this
attribute is not set, and takes default to 14 days.

Knowing precisely when the account last connected thus requires to enumerate
the account's `lastLogon` attribute on all the Domain Controllers of the
forest.

Both the `lastLogon` and a `lastLogonTimestamp` attributes are stored as `UNIX
time`: 32-bit value containing the number of seconds elapsed since 1/1/1970.
Both attributes store time values in `Greenwich Mean Time (GMT)`.

### Replication metadata enumeration and timelining

The attributes that are / are not replicated can be listed using the following
PowerShell queries (that make use of the `Remote Server Administration Tools
(RSAT)`'s PowerShell `ActiveDirectory` module):

```bash
# DOMAIN_ROOT = "DC=LAB,DC=AD" for example

# Lists the attributes which are replicated.
Get-ADObject -SearchBase 'CN=schema,CN=configuration,<DOMAIN_ROOT>' -LDAPFilter '(&(objectClass=attributeSchema)(!systemFlags:1.2.840.113556.1.4.803:=1))' | Select-Object -Expand name

# Lists the attributes which are NOT replicated.
Get-ADObject -SearchBase 'CN=schema,CN=configuration,<DOMAIN_ROOT>' -LDAPFilter '(&(objectClass=attributeSchema)(systemFlags:1.2.840.113556.1.4.803:=1))' | Select-Object -Expand name
```

The `repadmin` utility and the `Get-ADReplicationAttributeMetadata` PowerShell
cmdlet (introduced in `Windows Server 2012`) can be used to enumerate the
replication metadata of a specified object:

```bash
repadmin /showobjmeta /Linked <. | DC_HOSTNAME> "<DISTINGUISHED_NAME>"

Get-ADReplicationAttributeMetadata -Server <DC> -IncludeDeletedObjects –ShowAllLinkedValues "<DISTINGUISHED_NAME>"
```

The [ADTimeline](https://github.com/ANSSI-FR/ADTimeline) PowerShell script can
be used to automate the enumeration of the replication metadata and
consolidates the modifications in a timeline. Only the objects "considered of
interest" are listed (more details on the project GitHub repository).

In order to access the objects in the tombstone, the account used to execute
the script must be able to read the object placed the
`Deleted Objects Container`. It is thus advised to execute the script with
`Domain Admins` privileges. Otherwise, only the replication metadata of
existing objects will be enumerated.

```bash
.\AD-timeline.ps1 -Server <GLOBAL_CATALOG_FQDN>
```

The [`FarsightAD`](https://github.com/Qazeer/FarsightAD) cmdlets uses
replication metadata to enrich the persistence review with timestamps of
last modification for the attributes enumerated.

Note: a NTDS database can be mounted offline to pull the AD replication
metadata:
[Active Directory Domain Services NTDS offline mounting](./adds_ntds_offline_mounting.md).

### References

  - [ANSSI - Leonard Savina - Investigation numérique sur l'annuaire Active Directory avec les métadonnées de réplication](https://cyber.gouv.fr/sites/default/files/2019/01/anssi-coriin_2019-ad_timeline.pdf)

  - [Microsoft - TechNet Wiki Archive - Understanding the AD Account attributes - LastLogon, LastLogonTimeStamp and LastLogonDate](https://social.technet.microsoft.com/wiki/contents/articles/22461.understanding-the-ad-account-attributes-lastlogon-lastlogontimestamp-and-lastlogondate)aspx
