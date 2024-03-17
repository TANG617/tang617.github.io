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
The official documents is here: https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html
Here is some valuable discussion on the process: https://forums.developer.nvidia.com/t/new-qspi-bootloaders-for-orin-nano-and-jetpack-6-0-dp/276776/17
In my view, when it get boot signal, after initialization of SOC with internal code, the bootloader from QSPI flash is loaded. It initialize the other components and ask the UEFI firmware to take over. The logo in the boot process comes from UEFI, which is seen as a minimal OS. Then, the Linux kernel takes over the device.
It's said that without BIOS, it could be simplified and almost impossible to be bricked.
Each update between main version like from JP5.x to JP6.x is supposed to update QSPI at the same time. If QSPI is broken or flash process is terminated accidentally, you will probably get `ECID is  Error: probing the target board failed.` message while flashing rootfs , which requires another QSPI flash.
Another interesting thing is that Jetson Orin is basically a giant USB storage device. When QSPI fails, it's successful to flash SD/eMMC or NVME attached to the Jetson, while it's impossible to boot, and SDK Manager didn't give the right error message, which is pretty confusing.

Note that 6.0DP has some differences than any other version.

### Flash Method
