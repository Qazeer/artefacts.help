---
title: Azure activity / subscription logs
summary: 'The Azure activity / subscription logs record activity in an Azure subscription, such as resource modification, virtual machine creation and start, etc.'
keywords: activity, subscription, identity.claims, UPN, callerIpAddress, resourceId, operationName, correlationId
tags:
  - azure_logs
last_updated: 2024-01-28
sidebar: sidebar
permalink: azure_activity_logs.html
folder: azure
---

### Activity log key fields

The key fields in the subscription / activity log schema are:

  - `identity.claims`: nested JSON with information about the identity that
    performed the action and its authentication method.

     - `identity.claims.http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn`:
       the `UPN` of the identity that performed the action.

     - `identity.claims.groups`: the AzureAD groups of which the identity is a
       member.

    - `identity.claims.ipaddr`: the IP address the identity authenticated from.

  - `callerIpAddress`: the IP address the action was performed from.

  - `resourceId`: the unique resource identifier of the resource. The
    `resourceId` follows the format:
    `/SUBSCRIPTIONS/<SUBSCRIPTION_ID>/RESOURCEGROUPS/<RESOURCEGROUP_NAME>/PROVIDERS/<PROVIDER>/<RESOURCE_NAME>`.

    The provider can for example be `/MICROSOFT.COMPUTE/VIRTUALMACHINES` or
    `MICROSOFT.STORAGE/STORAGEACCOUNTS`.

  - `operationName`: the name of the operation.

    Examples:
      - `MICROSOFT.AUTHORIZATION/ROLEASSIGNMENTS/WRITE`
      - `MICROSOFT.COMPUTE/VIRTUALMACHINES/WRITE`
      - `MICROSOFT.COMPUTE/VIRTUALMACHINES/START/ACTION`
      - `MICROSOFT.COMPUTE/VIRTUALMACHINES/DELETE`
      - `MICROSOFT.COMPUTE/DISKS/WRITE`
      - `MICROSOFT.STORAGE/STORAGEACCOUNTS/LISTKEYS/ACTION`
      - ...

  - `resultType` and `resultSignature` (more verbose): the result of the operation.

  - `correlationId`: an unique identifier that can be used to map the different
    events associated with a single operation.
