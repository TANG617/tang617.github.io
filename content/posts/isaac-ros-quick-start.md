---
title: Isaac ROS Quick Start
description: A practical quick start for Isaac ROS on Jetson with Docker, RealSense, and Visual SLAM.
publishDate: 2024-07-27T16:43:20+08:00
updatedDate: "2026-07-19T00:00:00+08:00"
tags:
  - IntelRealSenseD455
  - SLAM
  - JetsonOrin
---

## About Isaac ROS

![Isaac ROS packages and supported hardware](https://cdn.jsdelivr.net/gh/TANG617/images/202407271647117.png)

Isaac ROS is a collection of NVIDIA-accelerated ROS 2 packages, development tools, and container-based workflows. It is not a standalone ROS 2 distribution.

Start with the current [Isaac ROS documentation](https://nvidia-isaac-ros.github.io/getting_started/index.html). Package compatibility changes between releases, so record the Isaac ROS, JetPack, ROS 2, sensor firmware, and driver versions used by a project.

## Quick setup

NVIDIA's documented development workflow uses an Isaac ROS development container.

### NVIDIA Container Toolkit

Install the runtime by following the [NVIDIA Container Toolkit installation guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html). This setup uses Docker.
Validate with a sample workload:

```bash
sudo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```

### Isaac ROS development container

- [Set up the Jetson hardware and software](https://nvidia-isaac-ros.github.io/getting_started/index.html#setup).
- [Create the development container with `isaac_ros_common`](https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_common/index.html).

### Isaac ROS Jetson

Use [`isaac_ros_jetson_stats`](https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_jetson/isaac_ros_jetson_stats/index.html#quickstart) to validate the environment and inspect Jetson resource usage.

![Isaac ROS Jetson validation output](https://cdn.jsdelivr.net/gh/TANG617/images/202407271723078.png)

### Isaac ROS Visual SLAM

- [Source repository](https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_visual_slam)
- [Visual SLAM quick start](https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_visual_slam/isaac_ros_visual_slam/index.html#quickstart)

Complete the release-specific [RealSense setup](https://nvidia-isaac-ros.github.io/getting_started/hardware_setup/sensors/realsense_setup.html) first. The original 2024 test used RealSense firmware 5.13.0.50, librealsense 2.55.1, and the RealSense ROS driver 4.51.1. Those versions are a historical tested combination, not a universal requirement for later Isaac ROS releases.

Check the camera with `realsense-viewer` before starting the ROS launch files.

Continue with NVIDIA's [cuVSLAM tutorial for RealSense](https://nvidia-isaac-ros.github.io/concepts/visual_slam/cuvslam/tutorial_realsense.html).
