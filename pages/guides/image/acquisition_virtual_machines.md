---
title: Image acquisition - Virtual machines
summary: 'Image acquisition of VMware vSphere virtual machines: procedure, thick and thin provisioning, virtual disks and disk descriptor files created by snapshots (SEsparse and VMFSsparse formats), reassembly of the snapshot virtual disks (vmdksync and vmware-vdiskmanager), and known limitations.'
keywords: image, disk, image acquisition, virtual machine, VMware, vSphere, ESXi, ESXi Host Client, datastore, VMFS, vmdk, flat, SEsparse, VMFSsparse, sesparse, delta, disk descriptor file, parentFileNameHint, snapshot, thin provisioning, thick provisioning, lazily zeroed, eagerly zeroed, vmsn, vmem, vmware.log, vmdksync, vmware-vdiskmanager, X-Ways, OVF
tags:
  - guides
last_updated: 2025-09-01
sidebar: sidebar
permalink: image_acquisition_virtual_machines.html
folder: image
---

## VMware vSphere

### Procedure in brief

  1. Take a snapshot of the virtual machine to save the VM virtual memory (the
     corresponding option must be enabled when taking the snapshot). This can
     be done through the `ESXi Host Client` or over `SSH`.

  2. Shut down the virtual machine and copy all the files in the VM folder from
     the datastore. Depending on the number of past snapshot(s) taken, relevant
     data may be present in different virtual drives. Additionally, metadata
     files (`vmsn` for the virtual memory or disk descriptor files) can be
     required / relevant for analysis.

### Thick versus thin provisioning

For a thick-provisioned virtual disk, the specified space is reserved directly
upon the creation of the disk, whereas, for a thin-provisioned virtual disk,
the allocated space only grows over time, as required by the data used by the
virtual machine. However, as thin provisioning is a function of the `VMFS`
filesystem, downloading a thin-provisioned virtual disk (to an `NTFS`,
`ext3/4`, etc. filesystem) will result in a full-sized file.

Thick-provisioned drives can be either "lazily" or "eagerly" zeroed. While in
both configurations the desired size of the virtual disk is reserved directly
upon creation, the way empty disk space is processed differs. In an "eagerly"
zeroed virtual disk, the unused space is zeroed directly at the time of
creation, whereas in a "lazily" zeroed virtual disk, the unallocated space
remains untouched on the datastore until an I/O operation occurs for the
virtual machine.

### Impacts of snapshots on virtual drives

When taking a snapshot of a virtual machine, a new virtual drive will be
created. Modifications to the filesystem of the virtual machine after that
snapshot will be applied to this newly created virtual drive. The
drive is associated with its own disk descriptor file, which references the
parent virtual disk (including in a human-readable format in the
`parentFileNameHint` field).

The following files are thus created after a snapshot is taken:

  - The virtual drive `vmdk`: `disk-X-sesparse.vmdk` (`SEsparse` format, used
    on `VMFS6` datastores and for virtual disks larger than 2 TB on `VMFS5`
    datastores) or `disk-X-delta.vmdk` (`VMFSsparse` format, used for virtual
    disks smaller than 2 TB on `VMFS5` datastores), such as
    `disk-000001-sesparse.vmdk`, with `X` being an incremented number (that
    usually matches the snapshot number, but may not, as the file names are
    chosen based on availability).

  - The disk descriptor file: `disk-X.vmdk`, such as `disk-000001.vmdk`
    (with `X` being the same number as for the virtual drive).

The virtual machine logs, `vmware.log` and `vmware-XX.log` files, in the
virtual machine folder in the datastore, can be used to determine which
snapshot / virtual disk was used at a given time. The `scsi0:0.filename` log
entry specifies the virtual disk associated with the virtual machine at a boot
event.
Example: `2025-09-11T21:51:47.571Z In(05) vmx - DICT          scsi0:0.filename = "disk-000003.vmdk"`.

The virtual disks of snapshots can be reassembled / reconsolidated using
the third-party open-source script
[`vmdksync`](https://github.com/mpalmer/vmdksync/blob/master/vmdksync)
or the VMware `vmware-vdiskmanager.exe` utility. Some forensic tools, such as
`X-Ways`, can also be used to directly reassemble and mount snapshot virtual
disks (if the parent virtual drive(s) are also available and, for some tools,
already loaded).

For instance, the `vmware-vdiskmanager.exe` utility can be used to merge a
snapshot virtual disk and its parent virtual disk(s), using the disk
descriptor file of the latest snapshot as input:

```
vmware-vdiskmanager.exe -r <LATEST_SNAPSHOT_DISK_DESCRIPTOR_FILE> -t 0 <OUTPUT_VMDK>
```

The virtual disks can then be mounted as detailed in the
[Windows](./mounting_windows.md) and
[Linux](./mounting_linux.md#virtual-machine-disks) image mounting notes.

### Known limitations

If downloading the virtual disk(s) from the `ESXi Host Client` `Datastore`
browser web interface, a virtual drive and its associated
disk descriptor file are listed as a single file. For instance, as `disk.vmdk`
for `disk.vmdk` (disk descriptor file) and `disk-flat.vmdk` (virtual disk), or
as `disk-000001.vmdk` for `disk-000001.vmdk` and
`disk-000001-sesparse.vmdk`. While the associated virtual machine is running,
this file will be mapped to the disk descriptor file of the VM, and when the
VM is powered off, the file will correspond to the actual virtual disk.

It is not possible to download any of the virtual drive(s), either using the
`ESXi Host Client` web interface or through `SCP`, if the virtual machine is
running. The downloaded `vmdk` will indeed appear empty.

Before shutting down the virtual machine, it is recommended to take a snapshot
of the virtual memory, as detailed in the
[VMware vSphere / ESXi section of the virtual machines memory acquisition note](../memory/acquisition_virtualmachines.md#vmware-vsphere--esxi).

--------------------------------------------------------------------------------

## References

  - [4n6k - Forensics Quickie: Merging VMDKs & Delta/Snapshot Files (2 Solutions)](https://www.4n6k.com/2014/04/forensics-quickie-merging-vmdks.html)

  - [Broadcom - Overview of virtual machine snapshots in vSphere (KB 1015180)](https://knowledge.broadcom.com/external/article?legacyId=1015180)

  - [Brendan Kinchla - Forensic Recovery of Evidence From Deleted VMware VSphere Hypervisor Virtual Machines](https://www.scribd.com/document/911111385/Forensic-Recovery-of-Evidence-From-Deleted-VMware-VSphere-Hypervisor-Virtual-Machines#content=query:eage,pageNum:16,indexOnPage:1,bestMatch:false)

  - [Mickaël Walter - A quick walkthrough in the VMDK format](https://web.archive.org/web/20250802235346/https://www.mickaelwalter.fr/a-quick-walkthrough-in-the-vmdk-format/)
