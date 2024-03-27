---
title: Jetson Orin Nano Setup
date: 2024-03-13 17:03:38
tags: 
- JetsonOrin
- Setup
---

During my setup of newly bought Jetson Orin Nano and some evaluation of the latest Jetpack 6.0 Developer Preview, I have to complain about Nvidia's confusing documents and low robust software. 
Here are my understanding of this special device after the painful experience of the basic setup process.

## Setup OS 
The software can  be roughly divided into 2 parts: L4T or Linux for Tegra and JetPack SDK. L4T is a release version of Ubuntu. JetPack SDK contains frameworks like CUDA.
The version of JetPack and L4T is tightly related, here is the latest JetPack SDK release note: https://developer.nvidia.com/embedded/jetpack
The documents is here: https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/index.html (Mind the version! It seems the only place reveals version is on the URL)

### Bootloader, UEFI and Q-SPI
[The official documents ](https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html)
> There is NO version switch on documents website! It seems the only way of switch version is to do some tricks on URL...
[Some valuable discussion on the process]( https://forums.developer.nvidia.com/t/new-qspi-bootloaders-for-orin-nano-and-jetpack-6-0-dp/276776/17)
> The community and an engineer from Nvidia mention Jetson as a big USB drive in the view of storage. The flash is basically copying files to the right partition.
In my view, when it get a boot signal, after initialization of SOC with internal code, the bootloader from QSPI flash is loaded. It initialize the other components and ask the UEFI firmware to take over the boot process. The logo in the boot process comes from UEFI, which is seen as a minimal OS. Then, the Linux kernel is loaded.
It's said that Jetson has no BIOS, cause it could be simplified and almost impossible to be bricked.

> To be clear, almost all the arm SBC, like RaspberryPi, boot similar to this process. However, the thing deterred me a lot it's that Jetson requires to update bootloader between EVERY major Jetpack updates, like from JP5.x to JP6.x!
> What's worse, to  my experiment, the bootloader is not even backward compatible! which means it's a trouble to run different L4T even on different media.
> And, what's worse, the bootloader contains some supreme information like the module's model or the size of RAM. Any interruption in flashing process could make `nv_sdk_manager` impossible to recognize(`ECID is  Error: probing the target board failed.`). Then, it requires a manual `mmcblk` flash.

### Flash Methods
[Official Documents on Flashing](https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/SD/FlashingSupport.html)
> The scripts provided by Nvidia also contains "build" the system image from various config files of its model.
> Both `flash.sh` and `l4t_initrd_flash.sh` take flash `rootfs` as an option. It's not by default!
> `--storage ` or  option is to choose the storage on the HOST MACHINE! It's awful that the script would not give any error if you specified a invalid storage!

Mainly, there are 2 ways of flashing: `flash.sh` and `l4t_initrd_flash.sh`, which are different in its principle but the same in results for developing. Production systems  **must** use `initrd` flashing.
Command like `sudo ./flash.sh jetson-orin-nano-devkit-nvme internal` only flashes bootloader rather than `kernel` or `rootfs`

#### Flash and boot from NVME drive
1. Make bootloader ready
Flash the Jetson without SD card or SSD installed to make sure Q-SPI flash on-board is successfully flashed. Boot without any valid media to check whether proper version of UEFI is loaded correctly.

2. Flash Linux partition outside Jetson
Unmount the NVME from Jetson and install it to the host computer, flash use the script below: 
`sudo <env-var> ./tools/kernel_flash/l4t_initrd_flash.sh [ -S <rootfssize> ] -c <config> --external-device nvme0n1p1 --direct <nvmeXn1> <board> external`
example:
```bash
1. sudo BOARDID=3767 BOARDSKU=0005 FAB=TS4 ./tools/kernel_flash/l4t_initrd_flash.sh -c tools/kernel_flash/flash_l4t_external.xml --external-device nvme0n1p1 --direct <nvmeXn1> jetson-orin-nano-devkit external
```
`<nvmeXn1>`should be the derive name on the host machine. The drive should be initialize with erasing any partition, or script gives an error.
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
[Source](https://forums.developer.nvidia.com/t/cant-access-wifi-on-jetson-orin-nano-running-jetpack-6-dev/278835/3)
The installed AC8260 card should work immediately and the driver should be preinstalled in the Ubuntu( Indeed )
The solution is try to install `backport-iwlwifi-dkms` and reboot.

## To Be Solved
- Deploy SDK from Nvidia SDK Manager.



## Reference:
### `flash.sh` Usage
|Command line option|Description|
|---|---|
|-c <cfgfile>|Pathname of a flash partition table configuration file.|
|-d <dtbfile>|Pathname of a device tree file.|
|-f <flashapp>|Name of the flash application to be used. Flash applications are stored in the bootloader directory. The default flash application is `bootloader/tegraflash.py`.|
|-h|Prints descriptions of the command line syntax and command line options.|
|-k <partition_id>|Partition name or number specified in `flash.xml` or `flash.cfg`.|
|-m <mts_preboot>|Name of the MTS preboot file to be used, such as `mts_preboot`.|
|-n <nfs_args>|Static NFS network assignments: `<Client_IP>:­<Server_IP>:­<Gateway_IP>:­<Netmask>`.|
|-o <odmdata>|ODM data.|
|-p <bp_size>|Total eMMC hardware boot partition size.|
|-r|Skips building `system.img`; reuse the existing one.|
|-t <tegraboot>|Pathname of a tegraboot binary, such as `nvtboot.bin`.|
|-u <PKC_key_file>|Pathname of a file containing the PKC key used for an ODM fused board.|
|-v <SBK_key_file>|Pathname of a file containing the Secure Boot Key (SBK) used for an ODM fused board.|
|-w <wb0boot>|Pathname of a warm boot binary, such as `nvtbootwb0.bin`.|
|-x <tegraid>|Processor chip ID. The default value is:<br><br>- NVIDIA® Jetson Orin series: 0x23|
|-z <sn>|Serial number of the target board.|
|-B <boardid>|Board ID.|
|-C <args>|Kernel command line arguments. If this option is specified, it overrides the default values defined for `flash.sh`. If two or more arguments are specified, they must be enclosed in quotation marks and separated by spaces.<br><br>Kernel command line arguments are documented in the file `kernel-4.9/Documentation/kernel-parameters.txt`.<br><br>In the case of NFS booting, use this option to set NFS boot-related arguments if the `-I` option is omitted.|
|-F <flasher>|Pathname of a flash server, such as `nvtboot_recovery_cpu.bin`.|
|-G <file_name>|Reads the boot partition and saves the image to the specified file.|
|-I <initrd>|Pathname of the initrd file. The default value is null.|
|-K <kernel>|Pathname of a kernel image file such as `zImage` or `Image`.|
|-M <mts boot>|Pathname of an MTS boot file, such as `mts_si`.|
|-N <nfsroot>|NFS root address, such as `<my_IP_address>:­/my/exported/nfs/rootfs`.|
|-R <rootfs_dir>|Pathname of the sample rootfs directory.|
|-S <size>|Size of the rootfs in bytes. Valid only for an internal root device.<br><br>KB/MB/GB suffixes represent units of 1000, 10002, and 10003. The suffixes`KiB`/`MiB`/`GiB` represent of multipliers of 1024, 10242, and 10243.<br><br>For example, `2GiB` represents 2×10243 bytes.|
|–bup|Generates Bootloader update payload (BUP).|
|–clean-up|Cleans up the BUP buffer when the script is flashing a multi-spec BUP.|
|–multi-spec|Enables support for building a multi-spec BUP.|
|–no-flash|Performs all steps except physically flashing the board. The script creates a `system.img` file.|
|–no-systemimg|Prevents creation or re-creation of `system.img`.|
|–sparseupdate|Only flash partitions that have changed. Currently supports only SPI flash memory.|
|–usb-instance <id>|USB instance to connect to; `<id>` a USB port path (for examples 3-14). To find the <id>, read section [How to find usb-instance](https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/SD/FlashingSupport.html#how-to-find-usb-instance)|
|–user_key <user_key_file>|Pathname of a file that contains a user key that can be used to encrypt and decrypt kernel, kernel-dtb, and initrd binary images. If `user_key` is specified, then the `-v` option must also be specified.|
|–sparseupdate|only flash partitions that have changed. Currently only SPI flash memory is supported.|