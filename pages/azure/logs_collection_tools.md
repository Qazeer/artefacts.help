---
title: Procedure - Logs search and collection tools
summary: 'The Azure logs (Entra ID sign-ins and audit logs, Office 365 Unified and Mailbox Audit Logs, Azure Activity logs, etc.) can be collected using Microsoft PowerShell modules and third-party tools, such as DFIR-O365RC or Microsoft-Extractor-Suite.'
keywords: Search-UnifiedAuditLog, DFIR-O365RC, Microsoft-Extractor-Suite, Log Analytics workspace, Diagnostic settings
tags:
  - azure_logs
  - procedures
last_updated: 2024-01-28
sidebar: sidebar
permalink: azure_logs_collection_tools.html
folder: azure
---

Remark: if only a day is specified for the `StartDate` and `EndDate` parameters,
PowerShell will initialize the corresponding `DateTime` objects at 12:00 AM
(midnight) in the local system timezone (whereas records in the UAL are stored
in `UTC`). The results retrieved (by `Search-UnifiedAuditLog`, for example)
will thus be bound by the local system timezone. Timestamps should therefore be
provided directly in `UTC` or with timezone information.

### [Microsoft 365] Unified audit and Mailbox Audit logs manual search

The `Search-UnifiedAuditLog` cmdlet of the `ExchangeOnlineManagement` module
can be used to search and export the Microsoft 365 `Unified Audit Logs`. The
cmdlet returns a maximum of 5000 results for direct queries, and 50,000
(unsorted) results for paged queries. Requests that would return a large number
of events should thus be automated (for instance with `DFIR-O365RC` or
`Microsoft-Extractor-Suite`).

```bash
# If necessary, installs and/or imports the ExchangeOnlineManagement module.
Install-Module ExchangeOnlineManagement
Import-Module ExchangeOnlineManagement

# Connects to Microsoft 365.
Connect-ExchangeOnline

# Retrieves events for the specified timeframe and accounts.
Search-UnifiedAuditLog -ResultSize 5000 -StartDate <YYYY-MM-DDT00:00:00Z> -EndDate <YYYY-MM-DDT00:00:00Z> -UserIds '<EMAIL | EMAIL_1,...,EMAIL_N>'

# Retrieves events for the specified record type.
# Record types: https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-schema#auditlogrecordtype
# Record type examples for the Exchange workload: "ExchangeItem","ExchangeAdmin","ExchangeItemGroup","ExchangeSearch","ExchangeAggregatedOperation","ExchangeItemAggregated","ComplianceDLPExchange","ComplianceSupervisionExchange","MipAutoLabelExchangeItem"
# Record type examples for Entra ID: "AzureActiveDirectory","AzureActiveDirectoryAccountLogon","AzureActiveDirectoryStsLogon"
Search-UnifiedAuditLog -ResultSize 5000 -RecordType <RECORD_TYPES>

# Retrieves events for the specified operation(s).
# Operation examples for the Exchange workload: "MailboxLogin","MailItemsAccessed","FolderBind","Send","SendAs","SendOnBehalf","Set-Mailbox","New-InboxRule","Set-InboxRule","UpdateInboxRules","New-TransportRule","Set-TransportRule","Remove-InboxRule","Disable-InboxRule","Add-MailboxPermission","AddFolderPermissions","Add-RecipientPermission","Remove-MailboxPermission","RemoveFolderPermissions","Remove-RecipientPermission","Set-OwaMailboxPolicy","MoveToDeletedItems","SoftDelete","HardDelete","Hard Delete user","Set-CASMailbox","SearchCreated","SearchExported"
Search-UnifiedAuditLog -ResultSize 5000 -Operations <OPERATIONS>
```

### [Entra ID, Microsoft 365, & Azure] Microsoft-Extractor-Suite

The [`Microsoft-Extractor-Suite`](https://github.com/invictus-ir/Microsoft-Extractor-Suite)
PowerShell module can be used to extract logs from Entra ID and Microsoft 365.

The `AzureAD` / `AzureADPreview` based cmdlets (such as `Get-EntraSignInLogs`
and `Get-EntraAuditLogs`) were removed from `Microsoft-Extractor-Suite` in
version `4.0.0` (July 2025), following the retirement of the `AzureAD` module,
and the `Entra ID` logs are now retrieved through the `Microsoft Graph` API.

```bash
# If necessary, installs the Microsoft-Extractor-Suite and the required PowerShell modules.
Install-Module -Name Microsoft-Extractor-Suite
Install-Module -Name ExchangeOnlineManagement
# Required for the cmdlets using the Microsoft Graph API, such as Get-GraphEntraSignInLogs or Get-GraphEntraAuditLogs.
Install-Module -Name Microsoft.Graph
# Required for the Azure Activity logs collection (Get-ActivityLogs).
Install-Module -Name Az

Import-Module Microsoft-Extractor-Suite

# Connects to Microsoft 365, Microsoft Graph, and/or Azure (depending on the collection targets).
Connect-M365
Connect-MgGraph -Scopes "AuditLog.Read.All","Directory.Read.All"
Connect-AzureAZ

# Retrieves the Entra ID sign-in logs (interactive, non-interactive, service principal, and managed identity sign-ins).
# By default, retrieves data for the last 30 days.
Get-GraphEntraSignInLogs
# Retrieves Entra ID sign-in logs before and/or after the specified date(s).
Get-GraphEntraSignInLogs -startDate <YYYY-MM-DD> -endDate <YYYY-MM-DD>

# Retrieves the Entra ID audit logs.
# By default, retrieves data for the last 30 days.
Get-GraphEntraAuditLogs
# Retrieves Entra ID audit logs before and/or after the specified date(s).
Get-GraphEntraAuditLogs -startDate <YYYY-MM-DD> -endDate <YYYY-MM-DD>

# Retrieves all UAL data.
# By default, retrieves data for the last 90 days for all users.
Get-UAL [-Output JSON]
# For the specified user(s) and/or in the given timeframe.
Get-UAL [-Output JSON] -UserIds "<EMAIL | EMAILS_LIST>" -StartDate <YYYY-MM-DDT00:00:00Z> -EndDate <YYYY-MM-DDT00:00:00Z>

# Retrieves Mailbox audit logs for the specified or all mailboxes.
# Search-MailboxAuditLog being deprecated, Get-MailboxAuditLog relies on Search-UnifiedAuditLog (with the ExchangeItem record type).
Get-MailboxAuditLog [-StartDate <YYYY-MM-DDT00:00:00Z>] [-EndDate <YYYY-MM-DDT00:00:00Z>]
Get-MailboxAuditLog -UserIds "<EMAIL | EMAILS_LIST>"

# Retrieves Message Trace logs for the specified or all mailboxes.
# Uses Get-MessageTraceV2 to retrieve 90 days of message trace data by default, with pagination (5000 records per page) and the 10-day query window allowed by the cmdlet.
Get-MessageTraceLog [-StartDate <YYYY-MM-DDT00:00:00Z>] [-EndDate <YYYY-MM-DDT00:00:00Z>]
Get-MessageTraceLog -UserIds "<EMAIL | EMAILS_LIST>"
```

### [Entra ID, Microsoft 365, & Azure] DFIR-O365RC collector

[`DFIR-O365RC`](https://github.com/ANSSI-FR/DFIR-O365RC) is a PowerShell module
that implements a number of cmdlets to retrieve Microsoft 365, Entra ID, and
Azure logs. As `DFIR-O365RC` supports PowerShell Core, it can be used on both
Windows and Linux endpoints.

The logs are retrieved in `JSON` from the following sources of information:

  - `Office 365 Unified Audit Logs`

  - `Mailbox Audit Log`

  - `Entra ID sign-in logs`

  - `Entra ID audit logs`

  - `Azure Activity logs`

  - `Azure DevOps Activity logs`

`DFIR-O365RC` can also retrieve the `Entra ID` applications and service
principals, devices, and users (with their authentication methods).

The version `2.0.0` of `DFIR-O365RC` (August 2024) introduced breaking changes:
the user authentication (including the device code authentication flow on
PowerShell Core) was replaced by an application-only authentication, with
`DFIR-O365RC` running in the context of a dedicated `Entra ID` application
(service principal) authenticated with a certificate.

#### Installation

`DFIR-O365RC` can be installed from the `PowerShell Gallery` (which also
installs its dependencies, such as the `ExchangeOnlineManagement`,
`Microsoft Graph`, and `Az` modules) or used through its `Docker` image (the
recommended method according to the `DFIR-O365RC` documentation).

```bash
Install-Module -Name DFIR-O365RC

docker pull anssi/dfir-o365rc:latest
docker run --rm -v .:/mnt/host -it anssi/dfir-o365rc:latest
```

The module can also be installed manually, by placing the `DFIR-O365RC`
directory of the `DFIR-O365RC` project in one of the system module paths
(retrievable using `$env:PSModulePath`), after installing the dependencies
listed in the `DFIR-O365RC.psd1` file.

#### Application creation

The `DFIR-O365RC` application must be created, with the `New-Application`
cmdlet, before the logs collection. A privileged account is required to create
(and later remove) the application.

```bash
# Creates a self-signed certificate (in PFX format) and retrieves its base64-encoded public part (on Windows).
$certificate = New-SelfSignedCertificate -Subject "CN=DFIRO365RC" -KeySpec KeyExchange -NotBefore (Get-Date) -NotAfter (Get-Date).AddDays(365)
$certificateSecurePassword = Read-Host -AsSecureString -Prompt "Certificate password"
Export-PfxCertificate -Cert $certificate -FilePath DFIRO365RC.pfx -Password $certificateSecurePassword
$certificateb64 = [System.Convert]::ToBase64String($certificate.GetRawCertData())

# Creates the application, and outputs its application identifier (AppID) and the tenant name.
# The -subscriptions and -organizations switches are respectively required to retrieve the Azure Activity logs and Azure DevOps Activity logs, for the selected subscription(s) / organization(s).
New-Application -certificateb64 $certificateb64 [-subscriptions] [-organizations]

# Removes the application once the logs collection is done.
Remove-Application [-subscriptions] [-organizations]
```

#### DFIR-O365RC cmdlets

All the `DFIR-O365RC` cmdlets take as parameters the application identifier
(`-appId`), the tenant name (`-tenant`), and the path to the certificate in
`PFX` format (`-certificatePath`).

The `Unified Audit Logs` are retrieved by default using
`Exchange Online PowerShell` (`Search-UnifiedAuditLog`), or using the
`Microsoft Purview` audit search API if the `-purview` switch is specified.

```bash
$appId = "<APP_ID>"
$tenant = "<TENANT>.onmicrosoft.com"
$certificatePath = "<PFX_CERTIFICATE_PATH>"

$EndDate = (Get-Date).ToUniversalTime()
$StartDate30 = $EndDate.AddDays(-30)
$StartDate90 = $EndDate.AddDays(-90)

# Get a subset of Office 365 Unified audit logs (selection of operations deemed of interest).
# Operations sets: "all" (default), "allButAzureAD", "ExchangeOnly", "OneDrive_Sharepoint_Teams_YammerOnly", "AzureADOnly", "SecurityAlertsOnly", "Devices".
# The -mailboxLogin switch can be specified to also retrieve the MailboxLogin operations.
# Files produced:
# - Get-O365Light.log
# - O365_unified_audit_logs\YYYY-MM-DD\UnifiedAuditLog_<TENANT>_<YYYY-MM-DD>_<HH>-00-00.json
Get-O365Light -startDate $StartDate90 -endDate $EndDate -appId $appId -tenant $tenant -certificatePath $certificatePath [-operationsSet "allButAzureAD"] [-mailboxLogin]

# Get all Office 365 Unified audit logs, or the events of the specified record types.
# As performance is poor, usage should be limited to a small time period or to small tenants.
$StartDateLimited = $EndDate.AddDays(-<DAYS>)
Get-O365Full -startDate [$StartDate90 | $StartDateLimited] -endDate $EndDate -appId $appId -tenant $tenant -certificatePath $certificatePath [-recordTypes <RECORD_TYPES>]

# Get Defender for Office 365 logs, from Office 365 Unified audit logs.
# Defender logs require at least one Office 365 E5 license or a license plan with Microsoft Defender for Office 365.
Get-O365Defender -startDate $StartDate90 -endDate $EndDate -appId $appId -tenant $tenant -certificatePath $certificatePath

# Search for activity related to a particular user, IP address, or free-text query in the Office 365 Unified audit logs.

# To retrieve the default time zone of a given user's mailbox, the ExchangeOnlineManagement PowerShell module can be used (in order to correlate the Mailbox logs with UTC+0).
# Install-Module ExchangeOnlineManagement
Connect-ExchangeOnline
Get-MailboxRegionalConfiguration -Identity <USER_ID>

# If a user is specified, the Mailbox Audit Log will also be retrieved for the given user.
# User IDs example: "user.name1@domain.com", "user.name2@domain.com"
Search-O365 -startDate $StartDate90 -endDate $EndDate -appId $appId -tenant $tenant -certificatePath $certificatePath -userIds <USER_ID | USER_IDS_COMMA_LIST>
Search-O365 -startDate $StartDate90 -endDate $EndDate -appId $appId -tenant $tenant -certificatePath $certificatePath -IPAddresses <IP_ADDRESS | IP_ADDRESSES_COMMA_LIST>
Search-O365 -startDate $StartDate90 -endDate $EndDate -appId $appId -tenant $tenant -certificatePath $certificatePath -freeTexts "<TEXT>"

# Get tenant general information, plus all Entra ID sign-in and audit logs.
Get-AADLogs -startDate $StartDate30 -endDate $EndDate -appId $appId -tenant $tenant -certificatePath $certificatePath

# Get Entra ID service principals and their applications, OAuth2 permission grants, and app role assignments.
Get-AADApps -appId $appId -tenant $tenant -certificatePath $certificatePath

# Get Entra ID devices and their owners / users.
Get-AADDevices -appId $appId -tenant $tenant -certificatePath $certificatePath

# Get Entra ID users, and optionally their authentication methods.
Get-AADUsers -appId $appId -tenant $tenant -certificatePath $certificatePath [-authenticationMethods]

# Get all Azure activity logs for all the subscriptions the application has access to (subscription(s) selected at the application creation).
Get-AzRMActivityLogs -startDate $StartDate90 -endDate $EndDate -appId $appId -tenant $tenant -certificatePath $certificatePath

# Get all Azure DevOps activity logs for all the DevOps organization(s) the application has access to (organization(s) selected at the application creation).
Get-AzDevOpsActivityLogs -startDate $StartDate90 -endDate $EndDate -appId $appId -tenant $tenant -certificatePath $certificatePath
```

### [Entra ID & Azure] Log Analytics workspace or storage account with Diagnostic settings

Through the `Diagnostic settings`, Azure logs at tenant, subscription(s), or
resource(s) level can be either:

  - Exported to JSON-formatted files in a `storage account blob`. Logs exported
    to a blob will be in `PT1H.json` files and can be downloaded using the
    `Azure Storage Explorer` utility (among others).

  - Sent to a `Log Analytics workspace` to be processed directly in the cloud
    with `KQL` queries.

Once a `storage account` or `Log Analytics workspace` has been created, the
procedure to export logs from different sources is as follows:

  - `Entra ID` tenant logs (sign-in and audit logs) - P1/P2 license required:

    ```
    Microsoft Entra admin center
       => Entra ID => Monitoring & health => Diagnostic settings (left menu)
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

  - Resource logs:

    ```
    The given resource portal
       => Diagnostic settings
          => Add diagnostic setting
             => Check all or the relevant categories
             => Archive to a storage account / Send to Log Analytics workspace
    ```

If exported to a `storage account blob`, logs will be available in the
following folders:

  - Entra ID audit logs: `insights-logs-auditlogs`

  - Entra ID sign-in logs:

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