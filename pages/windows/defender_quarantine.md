---
title: Windows Defender - Quarantine
summary: 'Windows Defender quarantines files that were detected as malicious, storing the full content of the files. It is thus possible to recover the quarantined files for further investigation.\n\nAdditionally, Windows Defender stores some metadata on each detection under the "Windows Defender\Quarantine" folder, including the original file path of the file, the timestamp of quarantine, and the associated threat name.'
keywords: Windows Defender, Defender, Quarantine, dissect, defender-dump.py
tags:
  - windows_program_execution
  - windows_file_knowledge
  - windows_defense_evasion
location: 'Quarantined files:\n<SYSTEM_DRIVE>\ProgramData\Microsoft\Windows Defender\Quarantine\ResourceData\n\nMetadata on the detections associated with quarantined files:\n<SYSTEM_DRIVE>\ProgramData\Microsoft\Windows Defender\Quarantine\Entries'
last_updated: 2024-02-18
sidebar: sidebar
permalink: windows_defender_quarantine.html
folder: windows
---

### Overview

`Windows Defender` quarantines files that were detected as malicious, storing
and encrypting (in `RC4`) the full content of the files. The quarantined files
can be restored, or deleted permanently, by the end user. It is thus possible
to recover the files put in quarantine by `Windows Defender` for further
investigations.

Additionally, `Windows Defender` stores some metadata on each detection under
the `Windows Defender\Quarantine` folder. The information is similarly stored
in RC4-encrypted files.

### Information of interest

The quarantined files are stored under:
`%SystemDrive%\ProgramData\Microsoft\Windows Defender\Quarantine\ResourceData`.
The `RC4` key used to encrypt the file is hardcoded (in the `mpengine.dll`
`DLL`).

The metadata on each detection are stored under:
`%SystemDrive%\ProgramData\Microsoft\Windows Defender\Quarantine\Entries`.
For each quarantined file, its associated metadata notably includes:

  - The original file path of the file.

  - The identified threat name (such as `Backdoor:JS/Chopper.VH!MSR` or
    `Trojan:PowerShell/ReverseShell.SA`).

  - The timestamp of when the file was placed in quarantine (in `UTC`).

  - The detection `ThreatID`, that can be used to correlate the quarantined
    file with other `Windows Defender` artifacts, such as the
    `Windows Defender` [ETW events](./etw_windows_defender.md) and
    [Support log files](./defender_support_logs.md).

More information on the data structures holding the metadata and how the
quarantined files are stored can be found in
[FOX IT's "Reverse, Reveal, Recover: Windows Defender Quarantine Forensics" blog post](https://blog.fox-it.com/2023/12/14/reverse-reveal-recover-windows-defender-quarantine-forensics/).

### Tool(s)

The `dissect` functions `defender.quarantine` and `defender.recover` and the
[defender-dump.py](https://github.com/knez/defender-dump/blob/master/defender-dump.py)
Python script can be used to extract the metadata on the detections and the
quarantined files.

```bash
# Retrieves the metadata on the detections from the "Quarantine\Entries" folder.
python3 defender-dump.py <SYSTEM_ROOT_DIR>
target-query <TARGET> -v -f defender.quarantine

# Extracts and decrypt the quarantined files from the "Quarantine\ResourceData" folder.
python3 defender-dump.py --dump <SYSTEM_ROOT_DIR>
target-query <TARGET> -v -f defender.recover -o <OUTPUT_FOLDER>
```

### References

  - [FOX IT - Max Groot & Erik Schamper - Reverse, Reveal, Recover: Windows Defender Quarantine Forensics](https://blog.fox-it.com/2023/12/14/reverse-reveal-recover-windows-defender-quarantine-forensics/)
