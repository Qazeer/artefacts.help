---
title: Memory analysis - Overview
summary: 'Overview of the memory analysis tools (MemProcFS, MemNixFS, and Volatility) and general memory analysis steps.'
keywords: memory, memory analysis, memory forensics, MemProcFS, MemNixFS, Volatility, FindEvil, process tree, process injection, network connections, YARA
tags:
  - memory
  - procedures
last_updated: 2026-11-25
sidebar: sidebar
permalink: memory_analysis_overview.html
folder: memory
---

Memory analysis can be conducted with different tools, using distinct
approaches:

| Tool | Analyzed systems | Approach |
|------|------------------|----------|
| [`MemProcFS`](./analysis_memprocfs.md#memprocfs) | Windows | Presents the memory as files in a virtual file system (processes, network connections, registry, etc.), with a forensic mode producing timelines and `FindEvil` detections. |
| [`MemNixFS`](./analysis_memprocfs.md#memnixfs) | Linux (`x86-64`) | Presents the memory as files in a virtual file system, similarly to `MemProcFS`. |
| [`Volatility 3`](./analysis_volatility.md) (and `Volatility 2`) | Windows, Linux, and macOS | Command line framework, composed of numerous plugins executed individually. |

`MemProcFS` and `MemNixFS` allow browsing and searching the memory artefacts
with any tool, and provide a quick triage of the memory image (with their
`FindEvil` detections notably), while `Volatility` provides more granular
plugins, for instance to detect specific malware or rootkit techniques. The
tools are complementary and can be used together.

The acquisition of memory images is detailed in the
[memory acquisition overview](./acquisition_overview.md) and in the
[Windows](./acquisition_windows.md), [Linux](./acquisition_linux.md), and
[virtual machines](./acquisition_virtualmachines.md) memory acquisition notes.

## General analysis steps

The memory analysis of a compromised system depends on the context of the
investigation. For example, if a workstation is suspected to have been
compromised through a phishing attack, extracting the `.pst` / `.ost` files
associated with `Outlook` (using the `filescan` and `dumpfiles` plugins) for
analysis may be a good first step.

The following general, context-independent, steps can be followed to
investigate the memory of a system:

  - Review of the processes hierarchy, looking for suspicious parent / child
    relationships, such as `outlook.exe` or `iexplore.exe` spawning
    `cmd.exe` or `powershell.exe` processes.

  - Identification of rogue / unlinked processes and of process injection
    (using the `psxview` and `malfind` plugins for instance).

  - Review of the network connections and artefacts, looking notably for
    suspicious patterns, such as:
    - network activity of processes that do not normally interact over the
      network.
    - connections on non-web ports established by web browsers.
    - connections to known malicious IP addresses.

  - Scan of the memory for known patterns / strings, using `YARA` rules.

--------------------------------------------------------------------------------

## References

  - [Black Hat - Investigating Malware Using Memory Forensics - A Practical Approach](https://www.youtube.com/watch?v=BMFCdAGxVN4)

  - [K A, Monnappa - Learning Malware Analysis: Explore the concepts, tools, and techniques to analyze and investigate Windows malware](https://www.packtpub.com/en-us/product/learning-malware-analysis-9781788392501)

  - [OpenClassrooms - Récupérez les informations importantes de la mémoire Windows pour l'analyse](https://openclassrooms.com/fr/courses/1750151-menez-une-investigation-d-incident-numerique-forensic/6473549-recuperez-les-informations-importantes-de-la-memoire-windows-pour-lanalyse)
