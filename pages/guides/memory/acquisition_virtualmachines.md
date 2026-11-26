---
title: Memory acquisition - Virtual machines
summary: 'Memory acquisition of virtual machines from the hypervisor: VMware vSphere / ESXi memory snapshots, VMware memory and state files (vmsn, vmss, and vmem), conversion with vmss2core, and Microsoft Hyper-V (LiveKd).'
keywords: memory, RAM, memory dump, memory acquisition, virtual machine, hypervisor, VMware, ESXi, vSphere, vSAN, snapshot, suspend, vmsn, vmss, vmem, vim-cmd, objtool, vmss2core, memory.dmp, vmss.core, zlib1.dll, Hyper-V, vmrs, LiveKd, Proxmox, KVM
tags:
  - memory
  - guides
last_updated: 2026-11-25
sidebar: sidebar
permalink: memory_acquisition_virtualmachines.html
folder: memory
---

Detailed guides for `VMware ESXi`, `Microsoft Hyper-V`, `Proxmox VE`, and
`KVM` are available in the Kaspersky forum post
["How to get a memory dump of a virtual machine from its hypervisor"](https://forum.kaspersky.com/topic/how-to-get-a-memory-dump-of-a-virtual-machine-from-its-hypervisor-36407/).

## VMware vSphere / ESXi

### Memory snapshot

The memory of a virtual machine should be captured through a snapshot, with
the virtual machine's memory included, and not by suspending the virtual
machine. While suspending a virtual machine also saves its memory, the
execution of the virtual machine (and thus its services and network
connections) is interrupted until the virtual machine is resumed.

A snapshot of the virtual machine, with its memory included, can be taken:

  - In the `ESXi Host Client`, on the virtual machine page:
    `Actions -> Snapshots -> Take snapshot -> Check "Snapshot the virtual machine's memory" -> Take snapshot`.

  - In the `ESXi Shell`:
    1. The `VMID` of the virtual machine to snapshot must first be retrieved,
       for instance with `vim-cmd vmsvc/getallvms`.
    2. Then a snapshot of the virtual machine, with the virtual memory included,
       can be taken using:
       `vim-cmd vmsvc/snapshot.create <VMID> "<SNAPSHOT_NAME>" "<SNAPSHOT_DESCRIPTION>" 1 0`.
       The two last parameters, `includeMemory` and `quiesced`, respectively
       specify to include the virtual memory (`1`) and to not quiesce the
       guest file system (`0`), as quiescing and memory snapshots are mutually
       exclusive.

The snapshot memory (`vmem`) and state (`vmsn`) files, respectively named
`<VM_NAME>-Snapshot<N>.vmem` and `<VM_NAME>-Snapshot<N>.vmsn`, must then
both be retrieved from the virtual machine folder in the datastore:

  - From the `ESXi Host Client`:
    `Storage -> Datastores -> Datastore browser -> <DATASTORE> -> <VM_NAME> folder -> Download the vmem and vmsn files`.

  - Over `SCP` (if `SSH` is enabled on the `ESXi` host), from the
    `/vmfs/volumes/<DATASTORE>/<VM_NAME>/` folder.

On `vSAN` datastores, the `vmem` file only references a `vSAN` object, which
must be retrieved through the `ESXi Shell`:

```bash
# Retrieves the ID of the vSAN object referenced by the vmem file.
cat /vmfs/volumes/<VSAN_DATASTORE>/<VM_NAME>/<VM_NAME>-Snapshot<N>.vmem

# Opens the vSAN object, which can then be copied from /vmfs/devices/vsan/<OBJECT_ID> (and renamed to <VM_NAME>-Snapshot<N>.vmem).
/usr/lib/vmware/osfs/bin/objtool open -u <OBJECT_ID>
```

The acquisition of the virtual disks of the virtual machine is covered in the
[virtual machines image acquisition note](../image/acquisition_virtual_machines.md#vmware-vsphere).

### VMware memory and state files

The following state and memory files can be found in the folder of a `VMware`
virtual machine:

  - `vmsn` (snapshot state file): created for each snapshot, as
    `<VM_NAME>-Snapshot<N>.vmsn`. The `vmsn` file contains the configuration
    and, if the memory was included in the snapshot, the active state of the
    virtual machine at the time of the snapshot.

  - `vmss` (suspended state file): created upon the suspension of the virtual
    machine, and containing the state of the virtual machine at the time of
    the suspension.

  - `vmem` (volatile memory file): raw content of the memory of the virtual
    machine, for a snapshot that includes the memory
    (`<VM_NAME>-Snapshot<N>.vmem`) or for a suspended virtual machine (with
    the same base name as the `vmss` file).

The `vmsn` / `vmss` file notably describes the layout of the memory regions
in the `vmem` file, which is required to correctly rebuild the physical memory
of a virtual machine whose memory is split into multiple regions. The `vmsn` /
`vmss` file should thus always be retrieved along with the `vmem` file, and
both files placed in the same folder, with the same base name, for analysis
(with `Volatility 3` or `MemProcFS` for instance).

On older versions of `ESXi`, the volatile memory was directly included in the
`vmsn` / `vmss` file, without a `vmem` file being created. This is still the
case for virtual machines with a virtual hardware version 10 or lower (the
latest version supported by `ESXi 5.5`), while a separate `vmem` file is
created for virtual machines with a virtual hardware version 11 (introduced
with `ESXi 6.0`) or later, or stored on `vVols` datastores.

### Conversion using vmss2core

The `VMware`
[`vmss2core`](https://knowledge.broadcom.com/external/article/323788/converting-a-memory-checkpoint-into-a-gu.html)
utility can be used to convert a snapshot (`vmsn`) or suspended state
(`vmss`) file, along with its `vmem` file if any, to a memory dump in a
standard format: a Microsoft crash dump (`memory.dmp`) for Windows virtual
machines, supported by most memory analysis tools (`Volatility`, `MemProcFS`,
`WinDbg`, etc.), or a core dump (`vmss.core`) for Linux virtual machines,
supported by the Red Hat `crash` utility. `vmss2core` supports both the
"monolithic" (memory included in the `vmsn` / `vmss` file, a format not
supported by `Volatility 3`) and "non-monolithic" (separate `vmem` file)
formats.

`vmss2core` is included in `VMware Workstation Pro` (free for all uses since
November 2024) and `VMware Fusion Pro`, and is also available as a standalone
`Fling` (the version bundled with `VMware Workstation Pro` / `Fusion Pro` should
however be used for recent guest operating systems). For Windows, the default
directory of `vmss2core.exe` is
`C:\Program Files (x86)\VMware\VMware Workstation` (and `/usr/bin` for Linux).
To use `vmss2core.exe` on another system, the binary and the `zlib1.dll`
shared library, from the same directory, must be copied.

```bash
# The snapshot (vmsn) or suspended state (vmss) file must be specified first, followed by the vmem file (if any).
# <OS_OPTION>:
#   -W8: Windows 8 / Windows Server 2012 and later.
#   -W: older Windows versions.
#   -N: Linux.
vmss2core.exe <OS_OPTION> <VM_NAME>-Snapshot<N>.vmsn <VM_NAME>-Snapshot<N>.vmem
vmss2core.exe <OS_OPTION> <VM_NAME>.vmss <VM_NAME>.vmem

# Example for a snapshot of a Windows 11 virtual machine.
vmss2core.exe -W8 <VM_NAME>-Snapshot1.vmsn <VM_NAME>-Snapshot1.vmem
```

Once complete, the memory dump file (`memory.dmp` for Windows virtual
machines, `vmss.core` for Linux virtual machines) is created in the current
directory.

For Windows virtual machines with `Virtualization-Based Security (VBS)` /
`Hyper-V` enabled (which requires the nested hardware virtualization, i.e.,
`vhv.enable = "TRUE"` in the virtual machine `vmx` configuration file), the
conversion may fail (with a `Cannot translate linear address fffff78000000000`
error) or produce a `vmss.core` file instead of a `memory.dmp` file. For
encrypted virtual machines (such as virtual machines with a `vTPM`), the files
must first be decrypted
([Broadcom KB 414452](https://knowledge.broadcom.com/external/article/414452/unable-to-convert-a-windows-11-vm-snapsh.html)).

Alternatively, the physical memory can be exported in the raw format, from the
`vmem` and `vmsn` / `vmss` files, using the
[`Volatility 3`](./analysis_volatility.md) `layerwriter.LayerWriter` plugin or
the `memory.pmem` file exposed by [`MemProcFS`](./analysis_memprocfs.md) (for
Windows virtual machines).

```bash
# Writes the physical memory layer (rebuilt from the vmem and vmsn / vmss files placed in the same folder) to <OUTPUT_DIR>/<LAYER_NAME>.raw.
vol -o <OUTPUT_DIR> -f <VM_NAME>-Snapshot<N>.vmem layerwriter.LayerWriter

# Lists the available layers, a specific layer being written with --layers <LAYER_NAME>.
vol -f <VM_NAME>-Snapshot<N>.vmem layerwriter.LayerWriter --list
```

## Microsoft Hyper-V

The
[Sysinternals `LiveKd`](https://learn.microsoft.com/en-us/sysinternals/downloads/livekd)
utility can be used, from the `Hyper-V` host, to dump the memory of a
`Hyper-V` virtual machine, as the `Hyper-V` saved state (`vmrs`) files are not
supported by most memory analysis tools (but are by `MemProcFS` on Windows,
using the `vmsavedstatedumpprovider.dll` library of the `Windows SDK`).

`LiveKd` requires the
[`Debugging Tools for Windows`](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools)
to be installed on the host, for instance with the `Windows SDK` installer
(`winsdksetup.exe`):

```bash
winsdksetup.exe /features OptionId.WindowsDesktopDebuggers /q /norestart
```

The Microsoft symbol server should also be configured on the host:

```bash
# Requires opening a new session to be effective.
setx _NT_SYMBOL_PATH "srv*C:\symbols*https://msdl.microsoft.com/download/symbols"
```

Before execution, `livekd.exe` should be copied to the `Debugging Tools`
folder (by default `C:\Program Files (x86)\Windows Kits\10\Debuggers\x64`).

```bash
# Lists the virtual machines running on the host.
livekd.exe -hvl

# Dumps the memory of the specified virtual machine to file.
# -p: pauses the virtual machine while LiveKd is active (recommended with -o).
livekd.exe -hv "<VM_NAME>" -p -o "<DUMP_FILE_PATH>"
```

--------------------------------------------------------------------------------

## References

  - [Broadcom - Converting a memory checkpoint into a guest memory dump using the vmss2core tool (KB 323788)](https://knowledge.broadcom.com/external/article/323788/converting-a-memory-checkpoint-into-a-gu.html)

  - [Broadcom - Converting virtual machine snapshots to memory dumps in VMware vSphere (KB 181598)](https://knowledge.broadcom.com/external/article/181598/how-to-convert-a-vmware-virtual-machine.html)

  - [Broadcom - Overview of virtual machine snapshots in vSphere (KB 1015180)](https://knowledge.broadcom.com/external/article?legacyId=1015180)

  - [Broadcom - Suspending a virtual machine on ESX/ESXi to collect diagnostic information (KB 326327)](https://knowledge.broadcom.com/external/article?legacyId=2005831)

  - [Kaspersky - How to get a memory dump of a virtual machine from its hypervisor](https://forum.kaspersky.com/topic/how-to-get-a-memory-dump-of-a-virtual-machine-from-its-hypervisor-36407/)

  - [Ulf Frisk - LeechCore wiki - Memory Acquisition Method: File](https://github.com/ufrisk/LeechCore/wiki/Device_File)

  - [Ulf Frisk - MemProcFS wiki](https://github.com/ufrisk/MemProcFS/wiki)

  - [vDan - Simplified Guide: How to Convert VM Snapshots into Memory Dumps Using vmss2core](https://vdan.cz/vmware/esxi/homelab/simplified-guide-how-to-convert-vm-snapshots-into-memory-dumps-using-vmss2core/)
