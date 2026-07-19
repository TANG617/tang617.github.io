---
title: "Pancake: An Automated Guided Vehicle with ROS and Visual SLAM"
description: An open-source automated guided vehicle built with ROS, Jetson Nano, Kinect V2, and visual SLAM.
publishDate: 2024-03-17T20:00:36+08:00
updatedDate: "2026-07-19T00:00:00+08:00"
tags:
  - JetsonNano
  - ROS
  - Kinect2
---

# Pancake

Pancake is an open-source automated guided vehicle built as a practical robotics platform. The name comes from the goal of making **Pan** a piece of **cake**.

The project combines a Jetson Nano, a Kinect V2 RGB-D camera, ROS, and visual SLAM. Its software and hardware notes are available in the [Pancake GitHub repository](https://github.com/TANG617/Pancake).

## System overview

- **Compute:** NVIDIA Jetson Nano
- **Perception:** Kinect V2 RGB-D camera
- **Middleware:** ROS 1
- **Localization and mapping:** visual SLAM
- **Purpose:** provide a compact platform for experimenting with mapping, navigation, and robot control

The RGB-D camera supplies color and depth data to the perception pipeline. ROS nodes expose the sensor streams and connect them to the SLAM and vehicle-control components.

## Related setup notes

- [A Brief Introduction to ROS](/posts/brief-of-ros/)
- [Using Kinect V2 with ROS on Jetson Nano](/posts/setup-kinect-v2-ros-jetson-nano/)

## Project status

This page is an overview rather than a complete build guide. For the tested source code, hardware details, and current limitations, refer to the repository and its revision history.
