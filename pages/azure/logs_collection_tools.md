---
title: Logs search and collection tools
summary: 'The Azure logs (Azure AD sign-ins and audit logs, Office 365 Unified and Mailbox Audit Audit Logs, Azure Activity logs, etc.) can be collected using Microsoft PowerShell modules and third-party tools, such as DFIR-O365RC or Microsoft-Extractor-Suite.'
keywords: Search-UnifiedAuditLog, DFIR-O365RC, Microsoft-Extractor-Suite, Log Analytics workspace, Diagnostic settings
tags:
  - azure_logs
last_updated: 2024-01-28
sidebar: sidebar
permalink: azure_logs_collection_tools.html
folder: azure
---

Remark: if only a day is specified, as `StartDate` and `EndDate` for some
cmdlets for instance, PowerShell will initialize the corresponding `DateTime`
objects at 12:00 AM (midnight) in the system local timezone (whereas records in
the UAL are stored in `UTC`). The results retrieved, by
`Search-UnifiedAuditLog` for example, will thus be bound by the local system
timezone. Timestamps should thus be provided directly in `UTC` or with timezone
information.

### [Office365] Unified audit and Mailbox Audit logs manual search

The `Search-UnifiedAuditLog` cmdlet of the `ExchangeOnlineManagement` module
can be used to search and export the Office365 `Unified Audit Logs`. The cmdlet
returns a maximum of 5000 results for direct queries, 50 000 (unsorted) results
for paged queries. Requests that would return a large number of events should
thus automated (for instance with `DFIR-O365RC` or `Microsoft-Extractor-Suite`).

```bash
# If necessary, install and/or import the ExchangeOnlineManagement module.
Install-Module ExchangeOnlineManagement
Import-Module ExchangeOnlineManagement

# Connect to Office365.
Connect-ExchangeOnline

# Retrieves events for the specified timeframe and accounts.
Search-UnifiedAuditLog -ResultSize 5000 -StartDate <YYYY-MM-DDT00:00:00Z> -EndDate <YYYY-MM-DDT00:00:00Z> -UserIds '<EMAIL | EMAIL_1,...,EMAIL_N>'

# Retrieves events for the specified record type.
# Record types: https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-schema#auditlogrecordtype
# Record type examples for the the Exchange workload: "ExchangeItem","ExchangeAdmin","ExchangeItemGroup","ExchangeSearch","ExchangeAggregatedOperation","ExchangeItemAggregated","ComplianceDLPExchange","ComplianceSupervisionExchange","MipAutoLabelExchangeItem"
# Record type examples for Azure AD: "AzureActiveDirectory","AzureActiveDirectoryAccountLogon","AzureActiveDirectoryStsLogon"
Search-UnifiedAuditLog -ResultSize 5000 -RecordType <RECORD_TYPES>

# Retrieves events for the specified operation(s).
# Operation examples for the Exchange workload: "MailboxLogin","MailItemsAccessed","FolderBind","Send","SendAs","SendOnBehalf","Set-Mailbox","New-InboxRule","Set-InboxRule","UpdateInboxRules","New-TransportRule","Set-TransportRule","Remove-InboxRule","Disable-InboxRule","Add-MailboxPermission","AddFolderPermissions","Add-RecipientPermission","Remove-MailboxPermission","RemoveFolderPermissions","Remove-RecipientPermission","Set-OwaMailboxPolicy","MoveToDeletedItems","SoftDelete","HardDelete","Hard Delete user","Set-CASMailbox","SearchCreated","SearchExported"
Search-UnifiedAuditLog -ResultSize 5000 -Operations <OPERATIONS>
```

### [Azure AD, Office365, & Azure] Microsoft-Extractor-Suite

The [`Microsoft-Extractor-Suite`](https://github.com/invictus-ir/Microsoft-Extractor-Suite)
PowerShell module can be used to extract logs from Azure AD and Office365.

```bash
# If necessary, installs the required PowerShell modules.
Install-Module -Name ExchangeOnlineManagement
Install-Module -Name AzureADPreview

# Only required for cmdlets using the GraphAPI, such as Get-EntraSignInLogsGraph or Get-EntraAuditLogsGraph
Install-Module Microsoft.Graph.Beta

# The AzureADPreview module MUST be imported (in place of the AzureAD module), as Get-AzureADAuditSignInLogs is updated to allow the retrieval of all events (instead of 1.000 entries with the AzureAD version).
Remove-Module -Name 'AzureAD' -Force
Import-Module -Name 'AzureADPreview' -Force

Import-Module .\Microsoft-Extractor-Suite.psd1

# Connects to Office 365, AzureAD, and/or Azure (depending on the collection targets).
Connect-M365
Connect-Azure
Connect-AzureAZ
Connect-MgGraph

# Retrieves all Azure AD sign-in logs.
Get-EntraSignInLogs
# Retrieves Azure AD sign-in logs before and/or after the specified date(s) (no timestamp support, date with day precision only).
Get-EntraSignInLogs -StartDate <YYYY-MM-DD> -EndDate <YYYY-MM-DD>

# Retrieves all Azure AD Audit logs.
Get-EntraAuditLogs
# Retrieves Azure AD Audit logs before and/or after the specified date(s) (no timestamp support, date with day precision only).
Get-EntraAuditLogs -StartDate <YYYY-MM-DD> -EndDate <YYYY-MM-DD>

# Retrieves the total number of records in the UAL per Record Type.
# By default retrieve data for the last 90 days for all users.
Get-UALStatistics
# For the specified user(s) and/or in the given timeframe.
Get-UALStatistics -UserIds "<EMAIL>" -StartDate <YYYY-MM-DDT00:00:00Z> -EndDate <YYYY-MM-DDT00:00:00Z>

# Retrieves all UAL data.
# By default retrieve data for the last 90 days for all users.
Get-UAL [-Output JSON]
# For the specified user(s) andor in the given timeframe.
Get-UAL [-Output JSON] -UserIds "<EMAIL | EMAILS_LIST>" -StartDate <YYYY-MM-DDT00:00:00Z> -EndDate <YYYY-MM-DDT00:00:00Z>

# Retrieves MailBox audit logs for the specified or all mailboxes.
Get-MailboxAuditLog [-StartDate <YYYY-MM-DDT00:00:00Z>] [-EndDate <YYYY-MM-DDT00:00:00Z>]
Get-MailboxAuditLog -UserIds "<EMAIL | EMAILS_LIST>"

# Retrieves Message Trace logs for the specified or all mailboxes.
# Uses Get-MessageTraceV2 to retrieve by default 90 days of message trace data, with  pagination (5000 records per page) and the 10-day query window allowed by the cmdlet.
Get-MessageTraceLog [-StartDate <YYYY-MM-DDT00:00:00Z>] [-EndDate <YYYY-MM-DDT00:00:00Z>]
Get-MessageTraceLog -UserIds "<EMAIL | EMAILS_LIST>"
```

### [Azure AD, Office365, & Azure] DFIR-O365RC collector

[`DFIR-O365RC`](https://github.com/ANSSI-FR/DFIR-O365RC) is a PowerShell module
that implement a number of cmdlets to retrieve Office 365Azure logs. As
`DFIR-O365RC` supports PowerShell Core, it can be used on both Windows or Linux
endpoints.

The logs are retrieved in `JSON` from the following sources of information:

  - `Office 365 Unified Audit Logs`

  - `Mailbox Audit Log`

  - `Azure AD sign-ins logs`

  - `Azure AD audit logs`

  - `Azure Activity logs`

  - `Azure DevOps Activity logs`

#### Manual installation

`DFIR-O365RC` depends on the [`MSAL.PS`](https://github.com/AzureAD/MSAL.PS)
and [`PoshRSJob`](https://github.com/proxb/PoshRSJob) modules, that must be
installed before usage.

```bash
Install-PackageProvider Nuget -Force
Install-Module -Name PowerShellGet -Force

Install-Module -Name MSAL.PS -RequiredVersion '4.21.0.1'
Install-Module -Name PoshRSJob -RequiredVersion '1.7.4.4'
```

On PowerShell Core, the installation of the `WSMan` client may also be
required:

```bash
Install-Module PSWSMan
Install-WSMan
```

The `DFIR-O365RC` directory of the `DFIR-O365RC` project can then be placed in
in one of the system modules path (retrievable using `$env:PSModulePath`) and
imported with `Import-Module DFIR-O365RC`.

#### DFIR-O365RC cmdlets

Note that whenever using PowerShell Core, the `-DeviceCode:$true` parameter
must be specified for all `DFIR-O365RC` cmdlets in order to authenticate to the
Azure AD tenant. The authentication should be done using a web browser at the
`https://microsoft.com/devicelogin` URL and the device code obtained passed
to the executed `DFIR-O365RC` cmdlet.

```bash
$EndDate = (Get-Date).ToUniversalTime()
$StartDate30 = $EndDate.adddays(-31)
$StartDate90 = $EndDate.adddays(-91)

# Get a subset of Office 365 Unified audit logs (selection of operations of judged of interest).
# Files produced:
# - Get-O365Light.log
# - O365_unified_audit_logs\YYYY-MM-DD\UnifiedAuditLog_<FQDN>_<YYYY-MM-DD>.json
Get-O365Light -StartDate $StartDate90 -Enddate $EndDate [-Operationsset "AllbutAzureAD"]

# Get all Office 365 Unified audit logs.
# As performance are poor, usage should be limited on a small time period or on small tenant.
$StartDateLimited = $EndDate.adddays(-<DAYS>)
Get-O365Full -StartDate [$StartDate90 | $StartDateLimited] -Enddate $EndDate

# Get Defender for Office 365 logs, from Office 365 Unified audit logs.
# Defender logs require an E5 license or a license plan with Microsoft Defender for Office 365cloud app security.
Get-DefenderforO365 -StartDate $StartDate90 -Enddate $EndDate

# Search for activity related to a particular user, IP address or freetext query in the Office 365 Unified audit logs.

# To retrieve the default time zone of a given user's mailbox the ExchangeOnlineManagement PowerShell module can be used (in order to correlate the Mailbox logs with UTC+0).
# Install-Module ExchangeOnlineManagement
Connect-ExchangeOnline
Get-MailboxRegionalConfiguration -Identity <USER_ID>

# If a user is specified, Mailbox Audit Log will also be retrieved for the given user.
# User ids example: "user.name1@domain.com", "user.name2@domain.com"
Search-O365 -StartDate $StartDate90 -Enddate $EndDate -UserIds <USER_ID | USER_IDS_COMMA_LIST>
Search-O365 -StartDate $StartDate90 -Enddate $EndDate -IPAddresses <IP_ADDRESS | IP_ADDRESSES_COMMA_LIST>
Search-O365 -StartDate $StartDate90 -Enddate $EndDate -Freetext "<TEXT>"

# Get tenant general information, plus all Azure sign-ins and audit logs.
Get-AADLogs	-StartDate $StartDate30 -Enddate $EndDate

# Get Azure audit logs related to Azure applications and service principals only.
Get-AADApps	-StartDate $StartDate30 -Enddate $EndDate

# Get Azure audit logs related to Azure AD joined or registered devices only.
Get-AADDevices -StartDate $StartDate30 -Enddate $EndDate

# Get all Azure activity logs available for the tenant or for the specified tenant.
Get-AzRMActivityLogs -StartDate $StartDate90 -Enddate $EndDate [-SelectSubscription:$true]

# Get all Azure DevOps activity logs available for all the DevOps organization(s) the account executing the cmdlet has access to or for the given DevOps organization.
Get-AzDevOpsActivityLogs -StartDate $StartDate90 -Enddate $EndDate [-SelectOrg:$true]
```

### [Azure AD & Azure] Log Analytics workspace or storage account with Diagnostic settings

Through the `Diagnostic settings`, Azure logs at tenant, subscription(s), or
resource(s) level can be either:

  - Exported to json formatted files in a `storage account blob`. Logs exported
    to a blob will be in `PT1H.json` files, and can be downloaded using the
    `Azure Storage Explorer` utility (among others).

  - Send to a `Log Analytics workspace` to be processed directly in the Cloud
    with `KQL` queries.

Once a `storage account` or `Log Analytics workspace` has been created, the
procedure to export logs from different sources is as follow:

  - `AzureAD` tenant logs (sign-ins and audit logs) - P1P2 license required:

    ```
    Azure Active Directory portal
       => Diagnostic settings (left menu)
          => Add diagnostic setting
             => Check "AuditLogs", "SignInLogs", "NonInteractiveUserSignInLogs", "ServicePrincipalSignInLogs", "ManagedIdentitySignInLogs", "ADFSSignInLogs", "RiskyUsers", "UserRiskEvents"
             => Archive to a storage account / Send to Log Analytics workspace
    ```

  - Subscription activity logs:

    ```
    Monitor portal
       => Activity log (left menu)
          => Export Activity logs (top menu)
             => Add diagnostic setting
                => Check all categories
                => Archive to a storage account / Send to Log Analytics workspace
    ```

  - Resources logs:

    ```
    The given resource portal
       => Diagnostic settings
          => Add diagnostic setting
             => Check all or the relevant categories
             => Archive to a storage account / Send to Log Analytics workspace
    ```

If exported to a `storage account blob`, logs will be available in the
following folders:

  - Azure AD audit logs: `insights-logs-auditlogs`

  - Azure AD sign-ins logs:

    - `insights-logs-signinlogs`

    - `insights-logs-noninteractiveusersigninlogs`

    - `insights-logs-managedidentitysigninlogs`

    - `insights-logs-serviceprincipalsigninlogs`

  - Subscription: `insights-activity-logs`

  - Resources:

    - Storage accounts: `insights-logs-storageread`

    - Key vaults: `insights-logs-auditevent`

    - NSG flows: `insights-logs-networksecuritygroupflowevent`

    - ...