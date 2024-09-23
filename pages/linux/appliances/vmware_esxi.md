---
title: VMware ESXi
summary: 'VMware ESXi run a Syslog service that logs messages from the kernel and other system components. Logs notably include information on authentication, commands entered in the ESXi Shell, and events on virtual machines life-cycle operations (configuration changes, web console access, snapshot operations, etc.). \n\n The ESXi logs location is defined through the "/etc/vmsyslog.conf" configuration file. By default, logs are placed in "/scratch/log/" / "/var/run/log/" (which are symlinks pointing to the same "/vmfs/volumes/<UID>/log/" directory).\n\nThe collection of ESXi logs can be automated through the creation of a "support bundle". A support bundle can be manually generated through the ESXi Host Client web interface or using the DFIR4vSphere PowerShell module (for ESXi attached to a running and reachable vCenter instance).'
keywords: Vmware, vSphere, ESX, ESXi, vCenter, VMkernel, logdir, scratch, vmsyslog.conf, support bundle, shell.log, auth.log, hostd.log, vmware.log, vpxa.log, vmware-hostd, Vimsvc, WebMKS, snapshot, MKSScreenShotMgr, DFIR4vSphere, Start-ESXi_Investigation, Start-VC_Investigation, ESXBundle, VCBundle
tags:
  - hypervisors
default_location: 'SSH authentication logs:\n/var/run/log/auth.log\n\nESXi host agent logs:\n/var/run/log/hostd.log\n\nESXi Shell logs:\n/var/run/log/shell.log\n\n vCenter Server agent logs:\n/var/run/log/vpxa.log\n\nPer virtual machines logs:\n/vmfs/volumes/<DATASTORE_GUID>/<VM>/vmware.log'
last_updated: 2024-06-02
sidebar: sidebar
permalink: vmware_esxi.html
folder: others
---

### VMware ESXi access

`ESXi` host can be accessed and managed using a number of methods.

| Access method | Description | Port |
|---------------|-------------|------|
| `ESXi Host Client` | Web management interface accessible over `HTTPS`. Access through the web interface are often leveraged to enable `SSH`. | `TCP` 443 |
| `Secure Shell (SSH)` | `SSH` access to execute commands locally on the `ESXi` host through the `ESXi Shell`. `SSH` is commonly leveraged by threat actors to perform operations on the `ESXi` host, including power-off and encryption of virtual machines's virtual disk `vmdk` for ransomware operators. | 22 |
| `Direct Console User Interface (DCUI)` | Text-based console accessible from the physical console attached to the `ESXi` host. | - |
| [vSphere Web Services API](https://developer.broadcom.com/xapis/vsphere-web-services-api/7.0U2/) | API exposed as a Web service to manage `vCenter` instances and `ESXi` hosts, including life-cycle operations on virtual machines. | 443 |

#### Microsoft Active Directory Domain Services integration

`ESXi` hosts can be joined to an `Active Directory` domain. If an `ESXi` host
is joined to an `AD` domain, members of the `AD` group defined in the
`Config.HostAgent.plugins.hostsvc.esxAdminsGroup` parameter will have full
administrative access to the host. By default, the `esxAdminsGroup` parameter
is set to `ESX Admins`.

The `esxAdminsGroup` parameter can be consulted and set in the
`ESXi Host Client` web management interface: "Managed" -> "Advanced Settings"
-> "Config.HostAgent.plugins.hostsvc.esxAdminsGroup". The value is stored in
the `/etc/vmware/configstore/current-store-1` configuration file.
Additionally, for offline analysis, the `esxAdminsGroup` parameter value is
retrieved in a support bundle collection .

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
| ESXi host agent | `<LOG_DIR>/hostd.log` | Events from the `vmware-hostd` host / management daemon, that is responsible for most of the operations on the `ESXi` host and its associated virtual machines. <br><br> **Include events:** <br> - **On authentications**, for SSH and other access types that are not recorded in auth.log, such as access through the web management interface ESXi Host Client, PowerCLI, programming language SDK, etc. <br> - **On virtual machine life-cycle operations**, such as VM creation, power on / off, deletion, snapshot operations. <br> - **On PowerCLI guest operations**, such as files transfer and command executions notably. <br> - etc. |
| ESXi Shell | `<LOG_DIR>/shell.log` | Events for shell related activity. <br><br> **Include events for every commands entered in the `ESXi Shell`, shell session lifecycle, enabling / disabling of SSH access, etc.** |
| Virtual machines | `/vmfs/volumes/<DATASTORE_GUID>/<VM>/vmware.log` | Per virtual machine events on the VM life-cycle operations. <br><br> **Include events for VM configuration changes, snapshot operations, `Vmware WebMKS` console access, guest operations (files transfer and command executions notably) through PowerCLI, etc.** |
| vCenter Server agent | `<LOG_DIR>/vpxa.log` | Events related to the ESXi agent that communicates with vCenter Servers (if the ESXi host is managed by vCenter). |
| Install and updates | `<LOG_DIR>/esxupdate.log` | Install and updates related events. |

Additionally, for environments with `vCenter`, `VI events` can also be a great
source of information (on every `vSphere API` calls made through `vCenter`).

#### Events related to various operations

**Authentication to the ESXi host**

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

{% include note.html content="Access through `PowerCLI`." %}

```
2024-06-03T19:58:26.435Z In(166) Hostd[1050278]: [Originator@6876 sub=Vimsvc.ha-eventmgr opID=c4c96306 sid=52018116] Event 220 : User root@192.168.127.13 logged in as PowerCLI/13.2.1.22851661
```

**Operations on the ESXi host itself**

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

{% include note.html content="Update of the \"Config.HostAgent.plugins.hostsvc.esxAdminsGroup\" parameter, setting the AD group with administrative access on the ESXI host to \"ESX Admins new\"." %}

```
Source: hostd.log

2024-06-02T21:19:01.386Z In(166) Hostd[1050738]: [Originator@6876 sub=Hostsvc.AppConfigOptionsProvider(Config.HostAgent.) opID=esxui-a003-5e51 sid=5295925c user=root] Set called with key 'Config.HostAgent.plugins.hostsvc.esxAdminsGroup' value '"ESX Admins new"'
```

**Operations on Guest virtual machines**

{% include note.html content="Snapshot of a virtual machine." %}

```
Source: shell.log (if conducted through ESXi shell)

2023-12-05T17:42:32.948Z In(14) shell[1053659]: [root]: vim-cmd vmsvc/snapshot.create $vm_id
```

```
Source: /vmfs/volumes/<DATASTORE_GUID>/<VM>/vmware.log (of the VM that has been snapshot)

2023-12-05T17:42:40.666Z In(05) vmx vim-cmd-09-a45d SnapshotVMX_TakeSnapshot start: 'snapshot_exfill', deviceState=1, lazy=1, quiesced=0, forceNative=0, tryNative=1, saveAllocMaps=0
2023-12-05T17:42:40.680Z In(05) vcpu-0 vim-cmd-09-a45d MainMem: Writing full memory image, '/vmfs/volumes/643a85c2-3920a51f-8175-48210b361288/Base-WindowsServer2019/Base-WindowsServer2019-Snapshot2.vmem'
2023-12-05T17:42:40.741Z In(05) vcpu-0 - SnapshotVMXTakeSnapshotWork: Initiated lazy snapshot 'snapshot_exfill': 2
2023-12-05T17:42:41.585Z In(05) vmx - SnapshotVMXTakeSnapshotComplete: Done with snapshot 'snapshot_exfill': 2
```

{% include note.html content="Access to a virtual machine description page through the `ESXi` web management interface `ESXi Host Client`." %}

```
Source: /vmfs/volumes/<DATASTORE_GUID>/<VM>/vmware.log (of the VM that has been snapshot)

2023-12-05T17:59:05.961Z In(05) vmx - MKSVMX: Vigor requested a screenshot
2023-12-05T17:59:05.961Z In(05) svga - MKSScreenShotMgr: Taking a screenshot
```

{% include note.html content="Access to a virtual machine login screen by launching the remote console through `ESXi` web management interface `ESXi Host Client`, bypassing possible network filtering. <br><br> Only indicates that an access was made to the guest host login screen, not necessarily that a successful login occurred (that would result on a logon `Type 2` / `Interactive` `4624` event on a Windows guest host)." %}

```
Source: /vmfs/volumes/<DATASTORE_GUID>/<VM>/vmware.log (of the VM that has been snapshot)

2023-12-05T18:02:22.340Z In(05) vmx - VigorTransportProcessClientPayload: opID=e0ada929 seq=8511: Receiving MKS.IssueTicket request.
2023-12-05T18:02:22.341Z In(05) vmx e0ada929 Issuing new webmks ticket 2d0378... (120 seconds)
2023-12-05T18:02:23.807Z In(05) mks - Accepting connection for webmks ticket 2d0378...
2023-12-05T18:02:23.807Z In(05) mks - MKSRemoteMgr: AddRemoteConsole (numConsoles=0)
2023-12-05T18:02:23.807Z In(05) mks - SOCKET 7 (119) Creating VNC remote connection.
2023-12-05T18:02:23.807Z In(05) svga - MVNCEncode: Number of screens changed from 0 to 1
```

{% include note.html content="Transfer of a file to a guest virtual machine (from a controlled system) using `PowerCLI`'s `Copy-VMGuestFile` cmdlet. <br><br> Command example: <br> `Copy-VMGuestFile -LocalToGuest -VM \"<VM_NAME>\" -GuestUser \"<USERNAME>\" -GuestPassword \"<PASSWORD>\" -Source \"<SOURCE_FILE>\" -Destination \"<DESTINATION_FULL_PATH>\"`." %}

```
Source: hostd.log

2024-06-03T22:46:09.892Z In(166) Hostd[1050233]: [Originator@6876 sub=Vimsvc.ha-eventmgr] Event 229 : Guest operation Initiate File Transfer To Guest performed on Virtual machine <VIRTUAL_MACHINE_NAME>.
2024-06-03T22:46:09.969Z In(166) Hostd[1050237]: [Originator@6876 sub=Guestsvc.GuestFileHandler] Guest File Path is <DESTINATION_FULL_PATH>
```

```
Source: /vmfs/volumes/<DATASTORE_GUID>/<VM>/vmware.log (of the VM to which the file was transferred)

2024-06-03T22:45:46.667Z In(05) vmx - VigorTransportProcessClientPayload: opID=c4c964e7 seq=845: Receiving GuestOps.InitiateFileTransferToGuest request.
2024-06-03T22:45:46.679Z In(05) vcpu-1 - VigorTransport_ServerSendResponse opID=c4c964e7 seq=845: Completed GuestOps request.
```

{% include note.html content="Download of a file from a guest virtual machine (to a local controlled system) using `PowerCLI`'s `Copy-VMGuestFile` cmdlet. <br><br> Command example: <br> `Copy-VMGuestFile -GuestToLocal -VM \"<VM_NAME>\" -GuestUser \"<USERNAME>\" -GuestPassword \"<PASSWORD>\" -Source \"<SOURCE_FULL_PATH>\" -Destination \"<DESTINATION_FOLDER | DESTINATION_FILE>\"`." %}

```
Source: hostd.log

2024-06-04T21:53:35.264Z In(166) Hostd[1050265]: [Originator@6876 sub=Vimsvc.ha-eventmgr] Event 243 : Guest operation Initiate File Transfer From Guest performed on Virtual machine <VIRTUAL_MACHINE_NAME>.
2024-06-04T21:53:35.409Z In(166) Hostd[1050754]: [Originator@6876 sub=Guestsvc.GuestFileHandler] Guest File Path is <SOURCE_FULL_PATH>
```

```
Source: /vmfs/volumes/<DATASTORE_GUID>/<VM>/vmware.log (of the VM from which the file was downloaded)

2024-06-04T21:53:35.220Z In(05) vmx - VigorTransportProcessClientPayload: opID=c4c9713c seq=3576: Receiving GuestOps.InitiateFileTransferFromGuest request.
2024-06-04T21:53:35.263Z In(05) vcpu-1 - VigorTransport_ServerSendResponse opID=c4c9713c seq=3576: Completed GuestOps request with messages.
```

{% include note.html content="Execution of a command on a guest virtual machine using `PowerCLI`'s `Invoke-VMScript` cmdlet. **The command executed is not logged in ESXi logs.** <br><br> The cmdlet first create a temporary file to store (in real-time) the script `stdout` / `stderr` output, execute the given command, retrieve the temporary file to display the script console output on the local machine, and delete the temporary file. <br><br> Command example: <br> `Invoke-VMScript -VM \"<VM_NAME>\" -GuestUser \"<USERNAME>\" -GuestPassword \"<PASSWORD>\" -ScriptText \"<COMMAND>\"`." %}

```
Source: hostd.log

2024-06-04T21:58:00.913Z In(166) Hostd[1050740]: [Originator@6876 sub=Vimsvc.ha-eventmgr] Event 245 : Guest operation Create Temporary File performed on Virtual machine <VIRTUAL_MACHINE_NAME>.
2024-06-04T21:58:00.975Z In(166) Hostd[1050232]: [Originator@6876 sub=Vimsvc.ha-eventmgr] Event 246 : Guest operation Start Program performed on Virtual machine <VIRTUAL_MACHINE_NAME>.
2024-06-04T21:58:06.170Z In(166) Hostd[1050230]: [Originator@6876 sub=Vimsvc.ha-eventmgr] Event 249 : Guest operation Initiate File Transfer From Guest performed on Virtual machine <VIRTUAL_MACHINE_NAME>.
2024-06-04T21:58:06.248Z In(166) Hostd[1050750]: [Originator@6876 sub=Guestsvc.GuestFileHandler] Guest File Path is C:\Users\vagrant\AppData\Local\Temp\powerclivmware151
2024-06-04T21:58:06.364Z In(166) Hostd[1050232]: [Originator@6876 sub=Vimsvc.ha-eventmgr] Event 250 : Guest operation Delete File performed on Virtual machine <VIRTUAL_MACHINE_NAME>.
```

```
Source: /vmfs/volumes/<DATASTORE_GUID>/<VM>/vmware.log (of the VM to which the file was transferred)

2024-06-04T21:58:00.759Z In(05) vmx - VigorTransportProcessClientPayload: opID=c4c97156 seq=3930: Receiving GuestOps.CreateTemporaryFile request.
2024-06-04T21:58:00.913Z In(05) vcpu-0 - VigorTransport_ServerSendResponse opID=c4c97156 seq=3930: Completed GuestOps request with messages.
2024-06-04T21:58:00.929Z In(05) vmx - VigorTransportProcessClientPayload: opID=c4c97157 seq=3936: Receiving GuestOps.StartProgram request.
2024-06-04T21:58:00.975Z In(05) vcpu-1 - VigorTransport_ServerSendResponse opID=c4c97157 seq=3936: Completed GuestOps request with messages.
2024-06-04T21:58:00.993Z In(05) vmx - VigorTransportProcessClientPayload: opID=c4c97158 seq=3942: Receiving GuestOps.ListProcesses request.
2024-06-04T21:58:01.013Z In(05) vcpu-1 - VigorTransport_ServerSendResponse opID=c4c97158 seq=3942: Completed GuestOps request with messages.
2024-06-04T21:58:06.038Z In(05) vmx - VigorTransportProcessClientPayload: opID=c4c9715a seq=3948: Receiving GuestOps.ListProcesses request.
2024-06-04T21:58:06.102Z In(05) vcpu-0 - VigorTransport_ServerSendResponse opID=c4c9715a seq=3948: Completed GuestOps request with messages.
2024-06-04T21:58:06.123Z In(05) vmx - VigorTransportProcessClientPayload: opID=c4c9715b seq=3954: Receiving GuestOps.InitiateFileTransferFromGuest request.
2024-06-04T21:58:06.169Z In(05) vcpu-0 - VigorTransport_ServerSendResponse opID=c4c9715b seq=3954: Completed GuestOps request with messages.
2024-06-04T21:58:06.319Z In(05) vmx - VigorTransportProcessClientPayload: opID=c4c9715e seq=3966: Receiving GuestOps.DeleteFile request.
2024-06-04T21:58:06.363Z In(05) vcpu-0 - VigorTransport_ServerSendResponse opID=c4c9715e seq=3966: Completed GuestOps request.
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
