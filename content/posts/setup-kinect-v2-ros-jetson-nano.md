---
title: Setup Kinect V2 in ROS for Jetson Nano
description: Hardware and software setup for using a Kinect V2 with ROS on an NVIDIA Jetson Nano.
publishDate: 2024-03-17T20:00:36+08:00
updatedDate: "2026-07-19T00:00:00+08:00"
tags:
  - JetsonNano
  - ROS
  - Kinect2
---

## Hardware modification

> **Safety warning:** This modification involves a mains-powered adapter and a 12 V supply. Verify the connector pinout, voltage, polarity, grounding, and current capacity before connecting the camera. A wiring error can damage the Kinect, Jetson, or power supply.

Kinect V2 was designed for the Xbox One and uses a proprietary connector that carries USB 3.0 and external power. The developer adapter breaks this out to a standard USB connection and supplies power. For this robot, I added a 12 V power lead with an XT30 connector and used a USB 3.0 Type-B to Type-C cable. This paragraph describes one tested unit; confirm the pinout independently before reproducing the modification.

## [libfreenect2](https://github.com/OpenKinect/libfreenect2)

Libfreenect2 is an open-source, cross-platform Kinect V2 driver. This setup uses legacy Jetson Nano software, so pin the tested source revision and dependencies rather than assuming the current default branch behaves identically.

- Make sure that `cmake ..` can find the CUDA dependency.
- Disable the OpenCL backend if OpenCL is unavailable. Test the CUDA backend with `Protonect` before adding ROS.
- Build with `cmake .. -DENABLE_CXX11=ON -DCUDA_PROPAGATE_HOST_FLAGS=off -DCMAKE_INSTALL_PREFIX=$HOME/freenect2`
- Run `make install`; the command above installs the library in `$HOME/freenect2`.
- Install USB driver for Kinect V2 `sudo cp ../platform/linux/udev/90-kinect2.rules /etc/udev/rules.d/`
- Note that Kinect V2 only supports USB 3, and some USB-C to USB-A is sensitive for flipping.
- If `tegraxusb` reports an error when launching `Protonect`, record the exact error and test another SuperSpeed cable or port before changing the build configuration.

## [iai_Kinect2](https://github.com/paul-shuvo/iai_kinect2_opencv4)

This bridge connects libfreenect2 to ROS. The linked fork contains OpenCV 4 compatibility changes for the legacy ROS 1 environment. Test libfreenect2 independently before debugging `kinect2_bridge`, and record the exact OpenCV, CUDA, and source revisions used by a reproducible build.

## Others

- In this setup, depth processing used CUDA while other image processing still consumed substantial CPU. Streaming multiple image topics also caused packet loss.
- Compressed image transport reduced network bandwidth and end-to-end latency in the remote-viewing setup.
