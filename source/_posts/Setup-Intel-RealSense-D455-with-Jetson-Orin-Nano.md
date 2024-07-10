---
title: Setup Intel RealSense D455 with Jetson Orin Nano
date: 2024-07-10 14:00:36
tags:
  - JetsonOrin
  - IntelRealSenseD455
---
# Step1: Setup Jetson Orin 
Reference: [Jetson-Orin-Nano-Setup](Jetson-Orin-Nano-Setup.md)
The latest release of Jetpack comes to JP6.0 rev3, some minimal fixes are patched.
1. Flash Q-SPI with scripts
2. Flash SSD with scripts
3. Install Jetpack with apt manually

# Step2: Setup Intel RealSense D455
Reference: [Official Guide](https://dev.intelrealsense.com/docs/nvidia-jetson-tx2-installation)
## LibRealSense SDK Backend
Comparing with the installation for general Ubuntu, L4T has 2 USB drivers as LibRealSense SDK Backend: 
1. Linux native kernel drivers for UVC, USB and HID (Video4Linux and IIO respectively)  
2. Using `RSUSB` - user-space implementation of the UVC and HID data protocols, encapsulated and activated by selecting the SDK's `-DFORCE_RSUSB_BACKEND` flag (a.k.a. `-DFORCE_LIBUVC` with SDK versions prior to v.2.30).
When the second method is selected Librealsense2 communicates with the devices using the standard USB driver, while the higher-level protocols (UVC/HID) stacks are compiled directly into the SDK.
For the best compatibility, **the second method is preferred**.
Installation with `apt` is not available

## Building from source rather than using binary 

Use `libuvc_installation.sh` in `scripts` to download compile and install.
The following scripts are modified for these purposes:
1. Use `development` branch instead of `master` branch to support IMU in Jetpack6.x
2. Enable parallel compiling with `make -j6`
4. Enable `CUDA` and assert `CUDA_COMPILER` with flag: `-DCMAKE_CUDA_COMPILER="/usr/local/cuda/bin/nvcc"  -DBUILD_WITH_CUDA=true`
5. Enable `RSUSB` backend with flag `-DFORCE_RSUSB_BACKEND=true`(`DFORCE_LIBUVC` is used for SDK versions prior to v.2.30)
Note:
1. The scripts would download source code in `~/librealsense` folder.
2. The another backend or the installation with apt cause conflict. Remove them before installation.
3. Compiling time may lasts over 10 minutes.
```bash
#!/bin/bash -xe

#Locally suppress stderr to avoid raising not relevant messages
exec 3>&2
exec 2> /dev/null
con_dev=$(ls /dev/video* | wc -l)
exec 2>&3

if [ $con_dev -ne 0 ];
then
        echo -e "\e[32m"
        read -p "Remove all RealSense cameras attached. Hit any key when ready"
        echo -e "\e[0m"
fi

lsb_release -a
echo "Kernel version $(uname -r)"
sudo apt-get update
cd ~/
sudo rm -rf ./librealsense_build
mkdir librealsense_build && cd librealsense_build

if [ $(sudo swapon --show | wc -l) -eq 0 ];
then
        echo "No swapon - setting up 1Gb swap file"
        sudo fallocate -l 2G /swapfile
        sudo chmod 600 /swapfile
        sudo mkswap /swapfile
        sudo swapon /swapfile
        sudo swapon --show
fi

echo Installing Librealsense-required dev packages
sudo apt-get install git cmake libssl-dev freeglut3-dev libusb-1.0-0-dev pkg-config libgtk-3-dev unzip -y
#rm -f ./master.zip

wget https://github.com/IntelRealSense/librealsense/archive/development.zip
unzip ./development.zip -d .
cd ./librealsense-development

echo Install udev-rules
sudo cp config/99-realsense-libusb.rules /etc/udev/rules.d/
sudo cp config/99-realsense-d4xx-mipi-dfu.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules && sudo udevadm trigger
mkdir build && cd build
cmake ../ -DCMAKE_CUDA_COMPILER="/usr/local/cuda/bin/nvcc" -DFORCE_LIBUVC=true -DFORCE_RSUSB_BACKEND=true -DBUILD_WITH_CUDA=true -DCMAKE_BUILD_TYPE=release
make -j6
sudo make install
echo -e "\e[92m\n\e[1mLibrealsense script completed.\n\e[0m"

```

# Step3: Verify the setup
Plugin the USB cable first.
Output of `lsusb`
```bash
Bus 002 Device 007: ID 8086:0b5c Intel Corp. Intel(R) RealSense(TM) Depth Camera 455
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 003: ID 8087:0a2b Intel Corp. Bluetooth wireless interface
Bus 001 Device 002: ID 1a40:0101 Terminus Technology Inc. Hub
Bus 001 Device 005: ID 2717:5010 Xiaomi Inc. Mi Wireless Combo
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```
Output of `lsusb -t`
```bash
lsusb -t
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=tegra-xusb/4p, 10000M
    |__ Port 2: Dev 7, If 0, Class=Video, Driver=uvcvideo, 5000M
    |__ Port 2: Dev 7, If 1, Class=Video, Driver=uvcvideo, 5000M
    |__ Port 2: Dev 7, If 2, Class=Video, Driver=uvcvideo, 5000M
    |__ Port 2: Dev 7, If 3, Class=Video, Driver=uvcvideo, 5000M
    |__ Port 2: Dev 7, If 4, Class=Video, Driver=uvcvideo, 5000M
    |__ Port 2: Dev 7, If 5, Class=Human Interface Device, Driver=usbhid, 5000M
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=tegra-xusb/4p, 480M
    |__ Port 2: Dev 5, If 0, Class=Human Interface Device, Driver=usbhid, 12M
    |__ Port 2: Dev 5, If 1, Class=Human Interface Device, Driver=usbhid, 12M
    |__ Port 3: Dev 2, If 0, Class=Hub, Driver=hub/4p, 480M
        |__ Port 4: Dev 3, If 0, Class=Wireless, Driver=btusb, 12M
        |__ Port 4: Dev 3, If 1, Class=Wireless, Driver=btusb, 12M
```
Graphic interface of `realsense-viewer`
The development branch has a different version number than master branch.
The GPU usage indicates the CUDA is enabled.
IMU and camera work fine.
![Screenshot from 2024-07-10 14-00-15.png](https://cdn.jsdelivr.net/gh/TANG617/images/202407101437026.png)
