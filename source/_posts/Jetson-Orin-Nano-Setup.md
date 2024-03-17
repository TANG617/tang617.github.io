---
title: Jetson Orin Nano Setup
date: 2024-03-13 17:03:38
tags: 
- JetsonOrin
- Setup
---

Recently, Nvidia has released the JetPack_6.0_DP and I've got a new Jetson Orin Nano board. Though I have a little experience in Jetson Nano with outdated JetPack, Orin with latest Jetpack has confused me a lot.

There are a series of blog to log the process of setup from scratch.

## Setup OS 
The software can  be roughly divided into 2 parts: L4T or Linux for Tegra and JetPack SDK. L4T is a release version of Ubuntu. JetPack SDK contains frameworks like CUDA.
The version of JetPack and L4T is tightly related, here is the latest JetPack SDK release note: https://developer.nvidia.com/embedded/jetpack
The documents is here: https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/index.html (Mind the version! It seems the only place reveals version is on the URL)
### Boot Flow, Specially QSPI
Before the OS installation, It's necessary to aware that Orin has various storage and a complicated process of boot flow.   
[The official documents ](https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html)
[Some valuable discussion on the process]( https://forums.developer.nvidia.com/t/new-qspi-bootloaders-for-orin-nano-and-jetpack-6-0-dp/276776/17)
In my view, when it get boot signal, after initialization of SOC with internal code, the bootloader from QSPI flash is loaded. It initialize the other components and ask the UEFI firmware to take over. The logo in the boot process comes from UEFI, which is seen as a minimal OS. Then, the Linux kernel takes over the device.
It's said that without BIOS, it could be simplified and almost impossible to be bricked.
Each update between main version like from JP5.x to JP6.x is supposed to update QSPI at the same time. If QSPI is broken or flash process is terminated accidentally, you will probably get `ECID is  Error: probing the target board failed.` message while flashing rootfs , which requires another QSPI flash.
Another interesting thing is that Jetson Orin is basically a giant USB storage device. When QSPI with bootloader fails to run, it could be successful to flash SD/eMMC or NVME attached to the Jetson with `l4t_initrd_flash.sh` command, while it's impossible to boot, and SDK Manager didn't give the right error message, which is pretty confusing.

Note that 6.0DP has some differences than any other version.

### Flash Method
[Official Documents on Flashing](https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/SD/FlashingSupport.html)

Mainly, there are 2 ways of flashing: `flash.sh` and `l4t_initrd_flash.sh`. The official method of flashing Orin NX and Nano is to use `initrd` flashing.
Use `flash.sh` to flash a Jetson device with Bootloader and the kernel.
Use `l4t_initrd_flash.sh` to flash internal or external media connected to a Jetson device. This script uses the recovery initial ramdisk to do the flashing, and can flash internal and external media using the same procedure. 
Personally, `nvsdkmanager_flash.sh` is not recommended.
During the process, `flash.sh` script should read the EEPROM containing hardware information, use `SKIP_EEPROM_CHECK=1 BOARDID=<BOARDID> BOARDSKU=<BOARDSKU>`to skip.
There are multiple examples on official documents. The only thing you should take care is to flash bootloader before rootfs.

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
- Install and boot OS from NVME drive.
- Deploy SDK from Nvidia SDK Manager.