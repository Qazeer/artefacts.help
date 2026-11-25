---
title: Memory acquisition - Linux
summary: 'Memory acquisition of Linux systems with AVML (Acquire Volatile Memory for Linux) and LiME (Linux Memory Extractor).'
keywords: memory, RAM, volatile memory, memory dump, memory image, memory acquisition, Linux, AVML, LiME, Snappy, /proc/kcore, insmod, kernel module, Azure Blob Storage, symbol table
tags:
  - memory
  - procedures
last_updated: 2026-11-25
sidebar: sidebar
permalink: memory_acquisition_linux.html
folder: memory
---

The analysis of Linux memory images with `Volatility` requires a symbol table
matching exactly the kernel of the system (refer to the
[Volatility note](./analysis_volatility.md#symbol-tables-and-profiles)).
Recording the kernel version and the distribution of the system (with
`uname -a` and `cat /etc/os-release` for instance) upon acquisition can thus
ease the analysis.

## Acquire Volatile Memory for Linux (AVML)

[`AVML`](https://github.com/microsoft/avml) is a memory acquisition utility,
written in `Rust` and open-sourced by Microsoft, distributed as a static
binary. `AVML` does not require any prior knowledge of the target distribution
or kernel, nor any compilation on the target system. `AVML` reads the memory
from `/proc/kcore`, `/dev/crash`, or `/dev/mem`, and cannot acquire memory if
the kernel `lockdown` feature is enabled.

The memory images are produced in the `LiME` format or in a compressed
(`Snappy`) format, which significantly reduces the size of the image. The
compressed format is directly supported by `Volatility 3` and `MemNixFS`, and
can otherwise be converted back to the `LiME` format using `AVML`.

`AVML` also supports the upload of the memory image to `Azure Blob Storage`
or through `HTTP PUT` requests (for instance to a pre-signed `AWS S3` or
`Google Cloud Storage` URL).

```bash
# --- AVML 0.19.0 and later (subcommands)
# Generates a memory image in the LiME format.
avml acquire <OUTPUT_DUMP_LIME>

# Generates a compressed memory image, that can then be converted to the LiME format.
avml acquire --compress <OUTPUT_DUMP_COMPRESSED>
avml convert <OUTPUT_DUMP_COMPRESSED> <OUTPUT_DUMP_LIME>

# Uploads the memory image to the specified URL using an HTTP PUT request (or to Azure Blob Storage with --sas-url) and deletes the local file upon successful upload.
avml acquire --url <URL> --delete <OUTPUT_DUMP_LIME>
avml acquire --sas-url <AZURE_SAS_URL> --delete <OUTPUT_DUMP_LIME>

# --- Older versions
avml [--compress] <OUTPUT_DUMP>
avml-convert <OUTPUT_DUMP_COMPRESSED> <OUTPUT_DUMP_LIME>
avml --put <URL> --delete <OUTPUT_DUMP_LIME>
```

## Linux Memory Extractor (LiME)

[`LiME`](https://github.com/504ensicsLabs/LiME) is another memory acquisition
utility that can be used to capture the memory of Linux systems.

`LiME` is implemented as a `Loadable Kernel Module (LKM)`, loaded and executed
using the `insmod` command. The module must be compiled for the exact kernel
of the target system, either on the system itself or on a system with the
same distribution and kernel version.

```bash
# Compiles the LiME module (requires the headers of the targeted kernel, such as the linux-headers-$(uname -r) package on Debian-based distributions).
git clone https://github.com/504ensicsLabs/LiME.git
cd LiME/src && make

# format=lime: memory ranges prepended with a header containing their address space information.
# format=raw: concatenation of all the System RAM ranges.
# format=padded: all the non-System RAM ranges padded with zeros.
sudo insmod lime-$(uname -r).ko "path=<OUTPUT_DUMP> format=<lime | raw | padded>"

# Sends the memory image over the network, the image being retrieved by connecting to the specified port.
sudo insmod lime-$(uname -r).ko "path=tcp:<PORT> format=lime"
nc <TARGET_HOST> <PORT> > <OUTPUT_DUMP_LIME>

# Unloads the module.
sudo rmmod lime
```

--------------------------------------------------------------------------------

## References

  - [504ensicsLabs - LiME](https://github.com/504ensicsLabs/LiME)

  - [Microsoft - AVML](https://github.com/microsoft/avml)
