---
title: Windows Defender - Detection History files
summary: 'Windows Defender stores information on past detections, from its real-time and cloud-delivered protection components, in DetectionHistory file(s).\n\nInformation of interest, per detection:\n - file path, size, md5, sha1, and sha256 hashes of the file that triggered the detection.\n - The threat name.\n - The process and domain and username of the user associated with the detection.\n - For Potentially Unwanted Applications (PUA) detections, the associated Uninstall registry key.'
keywords:
tags:
  - windows_file_knowledge
location: 'Files, with GUID filenames, under:\n<SYSTEMDRIVE>:\ProgramData\Microsoft\Windows Defender\Scans\History\Service\DetectionHistory\'
last_updated: 2024-02-06
sidebar: sidebar
permalink: windows_defender_detectionhistory.html
folder: windows
---

### Overview

`Windows Defender` stores information on past detections, from its real-time
and cloud-delivered protection components, in `DetectionHistory` file(s). Part
of the information contained in the `DetectionHistory` files is displayed to
end-users in the `Windows Defender` "Current threats" and "Protection history"
interfaces (Windows Security -> Virus & threat protection ->
Current threats -> Protection history).

### Information of interest

The `DetectionHistory` file(s) are located in subdirectories under
`<SYSTEMDRIVE>:\ProgramData\Microsoft\Windows Defender\Scans\History\Service\DetectionHistory\`,
with a `GUID` filename. The files follow a specific, partially non-human
readable format, detailed in the
[DetectionHistory Parser README](https://github.com/jklepsercyber/defender-detectionhistory-parser).

For each threat detected, the following notable information is available:

  - The file path of the file that triggered the detection.

  - The threat name (such as `Backdoor:JS/Chopper.VH!MSR` or
    `Trojan:PowerShell/ReverseShell.SA`).

  - The process and domain and username of the user associated with
    the detection.

  - The size, `md5`, `sha1`, and `sha256` hashes of the file that triggered the
    detection.

  - For `Potentially Unwanted Applications (PUA)` detections, the associated
    [`Uninstall registry key`](./registry_systeminfo.md#installed-applications-uninstall)
    associated with the application.

  - Various metadata on the detection:

    - The detection `ThreatID`, that can be used to correlate the
      `DetectionHistory` entry with `Windows Defender`
      [ETW events](./etw_windows_defender.md) and
      [Support log files](./defender_support_logs.md).

    - Internal `Windows Defender` data about the signature that triggered the
      detection.

### Tool(s)

The [`DetectionHistory Parser`](https://github.com/jklepsercyber/defender-detectionhistory-parser)
tool (`KAPE` `DHParser` module) can be used to parse `DetectionHistory`
file(s) into JSON.

```bash
# Parses the specified file (and outputs the JSON result in the output directory).
dhparser.exe -f <DH_FILE> -o <OUTPUT_FOLDER>

# Recursively process the specified folder and parses the DH file(s) found.
dhparser.exe -rgf <DH_FOLDER> -o <OUTPUT_FOLDER>
```

### References

  - [jklepsercyber Jordan Klepser - DetectionHistory Parser README](https://github.com/jklepsercyber/defender-detectionhistory-parser)
