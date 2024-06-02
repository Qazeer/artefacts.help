---
title: VMware ESXi
summary: 'VMware ESXi run a Syslog service that logs messages from the kernel and other system components. Logs notably include information on authentication, commands entered in the ESXi Shell, and events on virtual machines life-cycle operations (configuration changes, web console access, snapshot operations, etc.). \n\n The ESXi logs location is defined through the "/etc/vmsyslog.conf" configuration file. By default, logs are placed in "/scratch/log/" / "/var/run/log/" (which are symlinks pointing to the same "/vmfs/volumes/<UID>/log/" directory).\n\nThe collection of ESXi logs can be automated through the creation of a "support bundle". A support bundle can be manually generated through the ESXi Host Client web interface or using the DFIR4vSphere PowerShell module (for ESXi attached to a running and reachable vCenter instance).'
keywords: Vmware, vSphere, ESX, ESXi, vCenter, VMkernel, logdir, scratch, vmsyslog.conf, support bundle, shell.log, auth.log, hostd.log, vmware.log, vpxa.log, vmware-hostd, Vimsvc, WebMKS, snapshot, MKSScreenShotMgr, DFIR4vSphere, Start-ESXi_Investigation, Start-VC_Investigation, ESXBundle, VCBundle
tags:
  - hypervisors
default_location: 'Authentication logs:\n/var/run/log/auth.log\n\nESXi host agent logs:\n/var/run/log/hostd.log\n\nESXi Shell logs:\n/var/run/log/shell.log\n\n vCenter Server agent logs:\n/var/run/log/vpxa.log\n\nPer virtual machines logs:\n/vmfs/volumes/<DATASTORE_GUID>/<VM>/vmware.log'
last_updated: 2024-06-02
sidebar: sidebar
permalink: vmware_esxi.html
folder: others
---

### VMware ESXi logs

#### Overview

`VMware ESXi` (5.0 and higher) run a `Syslog` service (`vmsyslogd`), that logs
messages from the `VMkernel` and other system components.

The log location is defined through the `logdir` variable in the `vmsyslogd`
service's configuration file `/etc/vmsyslog.conf`. By default, the logs are
placed on a local scratch volume, or a ramdisk, mapped to `/scratch/log/` /
`/var/run/log/`. The folders `/scratch/log/` and `/var/run/log/` are symlinks
pointing to the same `/vmfs/volumes/<UID>/log/` directory. Some minor log
files, and symlinks to log files under `/vmfs/volumes/<UID>/log/`, are also
placed in the `/var/log/` folder.

*The `<LOG_DIR>` below can be mapped to `/scratch/log/`, `/var/run/log/`,
`/vmfs/volumes/<UID>/log/`, or a custom directory depending on how the logs are
configured and accessed / collected.*

| Component | Location | Details |
|-----------|----------|---------|
| Authentication | `<LOG_DIR>/auth.log` | Events related to authentication to the local `ESXi` host. <br><br> **Includes events from the `sshd` daemon for remote access.** |
| ESXi host agent | `<LOG_DIR>/hostd.log` | Events from the `vmware-hostd` host / management daemon, that is responsible for most of the operations on the `ESXi` host and its associated virtual machines. <br><br> **Include events on VM creation, power on / off, deletion, snapshot operations, etc.** |
| ESXi Shell | `<LOG_DIR>/shell.log` | Events for shell related activity. <br><br> **Include events for every commands entered in the `ESXi Shell`, shell session lifecycle, enabling / disabling of SSH access, etc.** |
| Virtual machines | `/vmfs/volumes/<DATASTORE_GUID>/<VM>/vmware.log` | Per virtual machine events on the VM life-cycle operations. <br><br> **Include events for VM configuration changes, `Vmware WebMKS` console access, snapshot operations, etc.** |
| vCenter Server agent | `<LOG_DIR>/vpxa.log` | Events related to the ESXi agent that communicates with vCenter Servers (if the ESXi host is managed by vCenter). |
| Install and updates | `<LOG_DIR>/esxupdate.log` | Install and updates related events. |

Additionally, for environments with `vCenter`, `VI events` can also be a great
source of information (on every `vSphere API` calls made through `vCenter`).

#### Events related to various operations

{% include note.html content="SSH access (as root)." %}

```
Source: auth.log

2023-12-05T17:35:43.281Z In(38) sshd[1053654]: Connection from 192.168.127.10 port 58470
2023-12-05T17:35:43.320Z In(38) sshd[1053654]: Accepted keyboard-interactive/pam for root from 192.168.127.10 port 58470 ssh2
[...]
2023-12-05T17:59:31.981Z In(38) sshd[1053654]: Session closed for 'root' on /dev/char/pty/t0
```

{% include note.html content="Access through `ESXi` web management interface `ESXi Host Client`." %}

```
Source: hostd.log

2023-12-05T14:31:48.557Z In(166) Hostd[1050748]: [Originator@6876 sub=Vimsvc.HaSessionManager opID=esxui-8c02-9b39 sid=52f0e556] Accepted password for user root from 192.168.127.10 - session=52f0e556-5be4-83d4-8d71-8f9d8a43791c
2023-12-05T14:31:48.557Z In(166) Hostd[1050748]: [Originator@6876 sub=Vimsvc opID=esxui-8c02-9b39 sid=52f0e556] [Auth]: User root
2023-12-05T14:31:48.558Z In(166) Hostd[1050748]: [Originator@6876 sub=Vimsvc.ha-eventmgr opID=esxui-8c02-9b39 sid=52f0e556] Event 99 : User root@192.168.127.10 logged in as Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36 Edg/119.0.0.0
```

{% include note.html content="Enabling of SSH on the `ESXi` host." %}

```
Source: auth.log

2023-12-05T17:35:29.040Z No(13) SSH[1053644]: SSH login enabled
2023-12-05T17:35:29.106Z No(13) SSH[1053649]: SSH sandbox is not enabled, SSH will run in superdom
```

```
Source: hostd.log

2023-12-05T17:35:29.045Z In(166) Hostd[1050749]: [Originator@6876 sub=Vimsvc.ha-eventmgr] Event 129 : SSH access has been enabled.
2023-12-05T17:35:29.163Z In(166) Hostd[1050753]: [Originator@6876 sub=Hostsvc.ServiceSystem opID=esxui-42e-a241 sid=52f0e556 user=root] TSM-SSH running status is true
2023-12-05T17:35:29.163Z In(166) Hostd[1050753]: [Originator@6876 sub=Vimsvc.ha-eventmgr opID=esxui-42e-a241 sid=52f0e556 user=root] Event 130 : SSH for the host nc-ass-vip.sdv.fr has been enabled
```


{% include note.html content="Shell command execution." %}

```
Source: shell.log

2023-12-05T17:36:17.835Z In(14) shell[1053659]: [root]: vim-cmd vmsvc/getallvms
2023-12-05T17:37:35.560Z In(14) shell[1053659]: [root]: esxcli system version get
2023-12-05T17:37:45.836Z In(14) shell[1053659]: [root]: esxcli system account list
2023-12-05T17:38:21.798Z In(14) shell[1053659]: [root]: esxcli vm process list
2023-12-05T17:42:32.948Z In(14) shell[1053659]: [root]: vim-cmd vmsvc/snapshot.create $vm_id
```

{% include note.html content="Snapshot of a virtual machine." %}

```
Source: shell.log (if conducted through ESXi shell)

2023-12-05T17:42:32.948Z In(14) shell[1053659]: [root]: vim-cmd vmsvc/snapshot.create $vm_id
```

```
Source: vmfs/volumes/<DATASTORE_GUID>/<VM>/vmware.log (of the VM that has been snapshot)

2023-12-05T17:42:40.666Z In(05) vmx vim-cmd-09-a45d SnapshotVMX_TakeSnapshot start: 'snapshot_exfill', deviceState=1, lazy=1, quiesced=0, forceNative=0, tryNative=1, saveAllocMaps=0
2023-12-05T17:42:40.680Z In(05) vcpu-0 vim-cmd-09-a45d MainMem: Writing full memory image, '/vmfs/volumes/643a85c2-3920a51f-8175-48210b361288/Base-WindowsServer2019/Base-WindowsServer2019-Snapshot2.vmem'
2023-12-05T17:42:40.741Z In(05) vcpu-0 - SnapshotVMXTakeSnapshotWork: Initiated lazy snapshot 'snapshot_exfill': 2
2023-12-05T17:42:41.585Z In(05) vmx - SnapshotVMXTakeSnapshotComplete: Done with snapshot 'snapshot_exfill': 2
```

{% include note.html content="Access to a virtual machine description page through the `ESXi` web management interface `ESXi Host Client`." %}

```
Source: vmfs/volumes/<DATASTORE_GUID>/<VM>/vmware.log (of the VM that has been snapshot)

2023-12-05T17:59:05.961Z In(05) vmx - MKSVMX: Vigor requested a screenshot
2023-12-05T17:59:05.961Z In(05) svga - MKSScreenShotMgr: Taking a screenshot
```

{% include note.html content="Access to a virtual machine login screen by launching the remote console through `ESXi` web management interface `ESXi Host Client`, bypassing possible network filtering. <br><br> Only indicates that an access was made to the guest host login screen, not necessarily that a successful login occurred (that would result on a logon `Type 2` / `Interactive` `4624` event on a Windows guest host)." %}

```
Source: vmfs/volumes/<DATASTORE_GUID>/<VM>/vmware.log (of the VM that has been snapshot)

2023-12-05T18:02:22.340Z In(05) vmx - VigorTransportProcessClientPayload: opID=e0ada929 seq=8511: Receiving MKS.IssueTicket request.
2023-12-05T18:02:22.341Z In(05) vmx e0ada929 Issuing new webmks ticket 2d0378... (120 seconds)
2023-12-05T18:02:23.807Z In(05) mks - Accepting connection for webmks ticket 2d0378...
2023-12-05T18:02:23.807Z In(05) mks - MKSRemoteMgr: AddRemoteConsole (numConsoles=0)
2023-12-05T18:02:23.807Z In(05) mks - SOCKET 7 (119) Creating VNC remote connection.
2023-12-05T18:02:23.807Z In(05) svga - MVNCEncode: Number of screens changed from 0 to 1
```

### Artifacts and logs collection

The collection of `ESXi` logs and other artifacts can be automated through the
creation of a "support bundle". The generation of a support bundle is natively
supported by `ESXi` and can be conducted through the `ESXi` web management
interface `ESXi Host Client` or in PowerShell. In addition to the
`/var/run/log/` folder, a support bundle also includes the outputs of various
commands: running processes and active network connections, virtual machines
and datastores listing, etc.

A support bundle can be manually generated through the `ESXi`
`ESXi Host Client` interface at
`https://<ESXI_IP | ESXI_HOSTNAME>/cgi-bin/vm-support.cgi` or through the
"Monitor" interface (Host -> Monitor -> Logs -> Generate a support bundle).

`ESXi` support bundle, for a specific `ESXi` or all `ESXi` attached to a
`vCenter` instance, can be automatically generated and retrieved using the
PowerShell module [`DFIR4vSphere`](https://github.com/ANSSI-FR/DFIR4vSphere).
`DFIR4vSphere` also supports the collection of the `vCenter` instance's
`VI events`, and the generation of a support bundle for the `vCenter` appliance
itself. `DFIR4vSphere` requires that the targeted `ESXi` instances are attached
to a (running and reachable) `vCenter` instance.

```bash
Install-Module VMware.PowerCLI -Scope CurrentUser

Import-module DFIR4vSphere

Connect-VIServer <VCENTER_NAME>

# vCenter collection.

# Collects VI events (for the last X days) and generate a support bundle for the vCenter appliance itself.
# The support bundle generation can be omitted by removing the -VCBundle flag.
$EndDate = Get-Date
$StartDate = $EndDate.AddDays(<-X>)

Start-VC_Investigation -VCBundle -StartDate $StartDate -EndDate $EndDate

# For large environnement, the collection of VI events can be limited to predefined events of interest.
# The following events of interest are set (in the Start-VC_Investigation cmdlet):
#"ad.event.JoinDomainEvent", "VmFailedToSuspendEvent", "VmSuspendedEvent", "VmSuspendingEvent", "VmDasUpdateOkEvent", "VmReconfiguredEvent", "UserUnassignedFromGroup", "UserAssignedToGroup", "UserPasswordChanged", "AccountCreatedEvent", "AccountRemovedEvent", "AccountUpdatedEvent", "UserLoginSessionEvent", "RoleAddedEvent", "RoleRemovedEvent", "RoleUpdatedEvent", "TemplateUpgradeEvent", "TemplateUpgradedEvent", "PermissionAddedEvent", "PermissionUpdatedEvent", "PermissionRemovedEvent", "LocalTSMEnabledEvent", "DatastoreFileDownloadEvent", "DatastoreFileUploadEvent", "DatastoreFileDeletedEvent", "VmAcquiredMksTicketEvent", "com.vmware.vc.guestOperations.GuestOperationAuthFailure", "com.vmware.vc.guestOperations.GuestOperation", "esx.audit.ssh.enabled", "esx.audit.ssh.session.failed", "esx.audit.ssh.session.closed", "esx.audit.ssh.session.opened", "esx.audit.account.locked", "esx.audit.account.loginfailures", "esx.audit.dcui.login.passwd.changed", "esx.audit.dcui.enabled", "esx.audit.dcui.disabled", "esx.audit.lockdownmode.exceptions.changed", "esx.audit.shell.disabled", "esx.audit.shell.enabled", "esx.audit.lockdownmode.disabled", "esx.audit.lockdownmode.enabled", "com.vmware.sso.LoginSuccess", "com.vmware.sso.LoginFailure", "com.vmware.sso.Logout", "com.vmware.sso.PrincipalManagement", "com.vmware.sso.RoleManagement", "com.vmware.sso.IdentitySourceManagement", "com.vmware.sso.DomainManagement", "com.vmware.sso.ConfigurationManagement", "com.vmware.sso.CertificateManager", "com.vmware.trustmanagement.VcTrusts", "com.vmware.trustmanagement.VcIdentityProviders", "com.vmware.cis.CreateGlobalPermission", "com.vmware.cis.CreatePermission", "com.vmware.cis.RemoveGlobalPermission", "com.vmware.cis.RemovePermission", "com.vmware.vc.host.Crypto.Enabled", "com.vmware.vc.host.Crypto.HostCryptoDisabled", "ProfileCreatedEvent", "ProfileChangedEvent", "ProfileRemovedEvent", "ProfileAssociatedEvent", "esx.audit.esximage.vib.install.successful", "esx.audit.esximage.hostacceptance.changed", "esx.audit.esximage.vib.remove.successful"
Start-VC_Investigation -LightVIEvents -StartDate $StartDate -EndDate $EndDate

# A subset of VI events can also be retrieved specifically.
# For instance, to retrieve only the successful and unsuccessful login attempts.
# Full list of available VI Events, per vSphere version: https://github.com/lamw/vCenter-event-mapping
Start-VC_Investigation -LightVIEvents -LightVIEventTypesId "com.vmware.sso.LoginSuccess", "com.vmware.sso.LoginFailure" -StartDate $StartDate -EndDate $EndDate

# ESXi collection.

# Collect basic information (running processes, open network connections, local accounts, etc.) and generate a support bundle for every ESXi in a given cluster.
# The support bundle generation can be omitted by removing the -ESXBundle flag.
Get-VMHost -Location <CLUSTER_NAME> | Start-ESXi_Investigation -ESXBundle

# Collect basic information and generate a support bundle for the specific ESXi.
Start-ESXi_Investigation -Name <ESXI_NAME> -ESXBundle
```

### References

  - [VMware - ESXi Log File Locations](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.monitoring.doc/GUID-832A2618-6B11-4A28-9672-93296DA931D0.html)

  - [broadcom - Configuring syslog on ESXi](https://knowledge.broadcom.com/external/article/318939/configuring-syslog-on-esxi.html)

  - [SYNACKTIV - VMWARE ESXI FORENSIC WITH VELOCIRAPTOR](https://www.synacktiv.com/publications/vmware-esxi-forensic-with-velociraptor)
