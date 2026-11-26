---
title: Image acquisition - Unix / Linux appliances
summary: 'Image acquisition of Unix / Linux appliances, such as physical network appliances, over SSH with dd, and remote filesystem mounting over SSH with SSHFS.'
keywords: image, disk, image acquisition, appliance, network appliance, Unix, Linux, live system, SSH, dd, od, tail, SSHFS, remote filesystem, mount
tags:
  - guides
last_updated: 2025-09-01
sidebar: sidebar
permalink: image_acquisition_appliances.html
folder: image
---

For some appliances, such as physical network appliances, taking an image of
the filesystem can be more easily achieved over `SSH`, from a mounted
partition / device of the live system. If taking an image is not practical, the
filesystem of the remote appliance can also be mounted locally over `SSH`
using `SSHFS`.

## Disk imaging over SSH

The mounted device(s) / partition(s) can be copied over `SSH` by
using `dd` on both the server and client sides.

```bash
# Running the ssh client from the system that will receive the image:
ssh <REMOTE_USER>@<REMOTE_HOST> "dd if=<DEVICE | PARTITION>" | dd of=<IMAGE_FILE>

# Depending on the targeted appliance, it might be required to "drop" into a shell using a specific appliance command (such as "shell" in the example below):
ssh <REMOTE_USER>@<REMOTE_HOST> 'shell "dd if=<DEVICE | PARTITION>"' | dd of=<IMAGE_FILE>

# If running the ssh client directly on the remote appliance:
dd if=<DEVICE | PARTITION> | ssh <LOCAL_USER>@<LOCAL_SYSTEM> "dd of=<OUTPUT_FILE>"
```

Following the copy, any message printed by the remote host to `stdout`
upon an `SSH` connection will have to be removed to get the image in a clean
state:

```bash
# Checks the first NUMBER_BYTES bytes of the image for any printed string.
od -A d -N <NUMBER_BYTES> <IMAGE_FILE>

# Trims the first NUMBER_BYTES bytes of the image to remove any string printed to stdout.
# Note: tail is much faster than dd.
tail -c+<NUMBER_BYTES+1> <IMAGE_FILE> > <TRIMMED_IMAGE_FILE>
dd iflag=skip_bytes skip=<NUMBER_BYTES> if=<IMAGE_FILE> of=<TRIMMED_IMAGE_FILE>
```

## Remote filesystem mounting using SSHFS

```bash
sudo apt install sshfs

sudo mkdir -p /mnt/<SSH_MOUNT_POINT>

# -o IdentityFile: if needed, private key to use instead of the user's password.
sudo sshfs [-odebug,sshfs_debug,loglevel=debug] -o reconnect [-o IdentityFile=<KEY_FILE>] <REMOTE_USER>@<REMOTE_HOST>:/ /mnt/<SSH_MOUNT_POINT>/
```
