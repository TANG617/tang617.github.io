---
title: Jetson Orin Nano Setup
description: Setup notes for Jetson Orin Nano, JetPack, bootloaders, and flashing Linux to NVMe.
publishDate: 2024-03-13T17:03:38+08:00
updatedDate: "2026-07-19T00:00:00+08:00"
tags:
  - JetsonOrin
  - Setup
---

This article records a Jetson Orin Nano installation performed with JetPack 6.0 Developer Preview and Jetson Linux 36.2 in March 2024. It is a historical troubleshooting record, not a current-version installation guide. Check the [JetPack release archive](https://developer.nvidia.com/embedded/jetpack-archive) and use the documentation matching the release you install.

## Setup OS

Jetson Linux, formerly Linux for Tegra or L4T, combines the Linux kernel, Ubuntu-based root filesystem, NVIDIA drivers, firmware, and board-support components. JetPack SDK adds components such as CUDA and other accelerated libraries. JetPack and Jetson Linux versions are coupled; this article links to the archived [Jetson Linux 36.2 Developer Guide](https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/index.html).

### Bootloader, UEFI and Q-SPI

[Jetson Orin boot-flow documentation](https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html)
[Some valuable discussion on the process](https://forums.developer.nvidia.com/t/new-qspi-bootloaders-for-orin-nano-and-jetpack-6-0-dp/276776/17)

> A useful mental model from the forum discussion is that flashing writes release artifacts to a defined partition layout; the real boot chain is more complex than treating the Jetson as a USB drive.
> At a high level, on-chip boot code initializes the SoC and loads boot firmware from QSPI. UEFI then initializes the platform and loads the operating-system bootloader or Linux kernel. UEFI is firmware, not a minimal operating system.

A JetPack major-version transition may require a QSPI boot-firmware update. Boot firmware and root-filesystem compatibility depends on the Jetson Linux release, so do not mix releases unless NVIDIA documents that combination as supported. An interrupted flash may leave the board unable to boot normally, but recovery mode and release-specific flashing tools can often restore it.

### Flash Methods

[Official Documents on Flashing](https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/SD/FlashingSupport.html)

> NVIDIA's scripts also build system images from board-specific configuration files.
> Both `flash.sh` and `l4t_initrd_flash.sh` take flash `rootfs` as an option. It's not by default!
> Some options identify storage attached to the host. Verify every block-device path before flashing; selecting the wrong device can destroy data.

The two main script-based workflows are `flash.sh` and `l4t_initrd_flash.sh`. The initrd workflow is recommended for many external-storage and production-provisioning scenarios, but the correct method depends on the target media and release documentation.

#### Flash and boot from NVME drive

1. Make bootloader ready
   Flash the Jetson without SD card or SSD installed to make sure Q-SPI flash on-board is successfully flashed. Boot without any valid media to check whether proper version of UEFI is loaded correctly.

2. Flash Linux partition outside Jetson
   Unmount the NVME from Jetson and install it to the host computer, flash use the script below:
   `sudo <env-var> ./tools/kernel_flash/l4t_initrd_flash.sh [ -S <rootfssize> ] -c <config> --external-device nvme0n1p1 --direct <nvmeXn1> <board> external`
   example:

```bash
sudo BOARDID=3767 BOARDSKU=0005 FAB=TS4 ./tools/kernel_flash/l4t_initrd_flash.sh -c tools/kernel_flash/flash_l4t_external.xml --external-device nvme0n1p1 --direct <nvmeXn1> jetson-orin-nano-devkit external
```

`<nvmeXn1>` is the NVMe device name on the host machine. Confirm it with a read-only command such as `lsblk` before running the flash command.
Note that `BOARDID=3767 BOARDSKU=0005` is of great importance, or the model recognized by L4T could be wrong.

3. Boot!
   After the flashing, mount the NVME back to Jetson and it would boot in theory.
   Note that the flashed system is L4T with basic BSP, no such library like CUDA is contained. Just use `apt` to install them.

## Some Other Problems

### Docker gives error after upgrading ubuntu

[Source](https://forums.developer.nvidia.com/t/docker-gives-error-after-upgrading-ubuntu/283563)
The problems comes from docker fails to create `docker0` lan bridge.
The following scripts should solve it.

```bash
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo apt reinstall docker-ce
```

### Can't access Wi-Fi

Reference: https://forums.developer.nvidia.com/t/jetpack-6-wifi-slow-startup-with-backport-iwlwifi-dkms/297967/7
tldr: Use `backports-5.15-*` instead of the latest `backports-6.*`

**Below methods are deprecated**
[Source](https://forums.developer.nvidia.com/t/cant-access-wifi-on-jetson-orin-nano-running-jetpack-6-dev/278835/3)
The installed AC8260 card should work immediately and the driver should be preinstalled in the Ubuntu( Indeed )
The solution is try to install `backport-iwlwifi-dkms` and reboot.

## Reference:

### `flash.sh` Usage

| Command line option       | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| -c <cfgfile>              | Pathname of a flash partition table configuration file.                                                                                                                                                                                                                                                                                                                                                                                                             |
| -d <dtbfile>              | Pathname of a device tree file.                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| -f <flashapp>             | Name of the flash application to be used. Flash applications are stored in the bootloader directory. The default flash application is `bootloader/tegraflash.py`.                                                                                                                                                                                                                                                                                                   |
| -h                        | Prints descriptions of the command line syntax and command line options.                                                                                                                                                                                                                                                                                                                                                                                            |
| -k <partition_id>         | Partition name or number specified in `flash.xml` or `flash.cfg`.                                                                                                                                                                                                                                                                                                                                                                                                   |
| -m <mts_preboot>          | Name of the MTS preboot file to be used, such as `mts_preboot`.                                                                                                                                                                                                                                                                                                                                                                                                     |
| -n <nfs_args>             | Static NFS network assignments: `<Client_IP>:­<Server_IP>:­<Gateway_IP>:­<Netmask>`.                                                                                                                                                                                                                                                                                                                                                                                |
| -o <odmdata>              | ODM data.                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| -p <bp_size>              | Total eMMC hardware boot partition size.                                                                                                                                                                                                                                                                                                                                                                                                                            |
| -r                        | Skips building `system.img`; reuse the existing one.                                                                                                                                                                                                                                                                                                                                                                                                                |
| -t <tegraboot>            | Pathname of a tegraboot binary, such as `nvtboot.bin`.                                                                                                                                                                                                                                                                                                                                                                                                              |
| -u <PKC_key_file>         | Pathname of a file containing the PKC key used for an ODM fused board.                                                                                                                                                                                                                                                                                                                                                                                              |
| -v <SBK_key_file>         | Pathname of a file containing the Secure Boot Key (SBK) used for an ODM fused board.                                                                                                                                                                                                                                                                                                                                                                                |
| -w <wb0boot>              | Pathname of a warm boot binary, such as `nvtbootwb0.bin`.                                                                                                                                                                                                                                                                                                                                                                                                           |
| -x <tegraid>              | Processor chip ID. The default value is:<br><br>- NVIDIA® Jetson Orin series: 0x23                                                                                                                                                                                                                                                                                                                                                                                  |
| -z <sn>                   | Serial number of the target board.                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| -B <boardid>              | Board ID.                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| -C <args>                 | Kernel command line arguments. If this option is specified, it overrides the default values defined for `flash.sh`. If two or more arguments are specified, they must be enclosed in quotation marks and separated by spaces.<br><br>Kernel command line arguments are documented in the file `kernel-4.9/Documentation/kernel-parameters.txt`.<br><br>In the case of NFS booting, use this option to set NFS boot-related arguments if the `-I` option is omitted. |
| -F <flasher>              | Pathname of a flash server, such as `nvtboot_recovery_cpu.bin`.                                                                                                                                                                                                                                                                                                                                                                                                     |
| -G <file_name>            | Reads the boot partition and saves the image to the specified file.                                                                                                                                                                                                                                                                                                                                                                                                 |
| -I <initrd>               | Pathname of the initrd file. The default value is null.                                                                                                                                                                                                                                                                                                                                                                                                             |
| -K <kernel>               | Pathname of a kernel image file such as `zImage` or `Image`.                                                                                                                                                                                                                                                                                                                                                                                                        |
| -M <mts boot>             | Pathname of an MTS boot file, such as `mts_si`.                                                                                                                                                                                                                                                                                                                                                                                                                     |
| -N <nfsroot>              | NFS root address, such as `<my_IP_address>:­/my/exported/nfs/rootfs`.                                                                                                                                                                                                                                                                                                                                                                                               |
| -R <rootfs_dir>           | Pathname of the sample rootfs directory.                                                                                                                                                                                                                                                                                                                                                                                                                            |
| -S <size>                 | Size of the rootfs in bytes. Valid only for an internal root device.<br><br>KB/MB/GB suffixes represent units of 1000, 10002, and 10003. The suffixes`KiB`/`MiB`/`GiB` represent of multipliers of 1024, 10242, and 10243.<br><br>For example, `2GiB` represents 2×10243 bytes.                                                                                                                                                                                     |
| –bup                      | Generates Bootloader update payload (BUP).                                                                                                                                                                                                                                                                                                                                                                                                                          |
| –clean-up                 | Cleans up the BUP buffer when the script is flashing a multi-spec BUP.                                                                                                                                                                                                                                                                                                                                                                                              |
| –multi-spec               | Enables support for building a multi-spec BUP.                                                                                                                                                                                                                                                                                                                                                                                                                      |
| –no-flash                 | Performs all steps except physically flashing the board. The script creates a `system.img` file.                                                                                                                                                                                                                                                                                                                                                                    |
| –no-systemimg             | Prevents creation or re-creation of `system.img`.                                                                                                                                                                                                                                                                                                                                                                                                                   |
| –sparseupdate             | Only flash partitions that have changed. Currently supports only SPI flash memory.                                                                                                                                                                                                                                                                                                                                                                                  |
| –usb-instance <id>        | USB instance to connect to; `<id>` a USB port path (for examples 3-14). To find the <id>, read section [How to find usb-instance](https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/SD/FlashingSupport.html#how-to-find-usb-instance)                                                                                                                                                                                                                    |
| –user_key <user_key_file> | Pathname of a file that contains a user key that can be used to encrypt and decrypt kernel, kernel-dtb, and initrd binary images. If `user_key` is specified, then the `-v` option must also be specified.                                                                                                                                                                                                                                                          |
| –sparseupdate             | only flash partitions that have changed. Currently only SPI flash memory is supported.                                                                                                                                                                                                                                                                                                                                                                              |
