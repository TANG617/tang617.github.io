---
title: A Brief Introduction to ROS
description: A concise introduction to ROS, its supported distributions, and deployment options on NVIDIA Jetson.
publishDate: 2024-03-17T19:38:04+08:00
updatedDate: "2026-07-19T00:00:00+08:00"
tags:
  - ROS
---

ROS, the Robot Operating System, is not an operating system like Linux or Windows. It is a collection of middleware, libraries, command-line tools, build systems, conventions, and message definitions for building modular robotic systems. Nodes written in C++, Python, and other languages can communicate through topics, services, and actions.

## ROS 1 and ROS 2

ROS 1 Noetic reached end of life in May 2025. Existing ROS 1 systems can still be maintained, but new projects should normally use ROS 2 unless they depend on legacy hardware or packages.

ROS distributions are tied to particular operating-system versions. As of this update, Humble and Jazzy are supported ROS 2 LTS releases; Kilted is a supported non-LTS release. Always check the official [ROS 2 distributions](https://docs.ros.org/en/rolling/Releases.html) page before choosing a version.

This blog's older Jetson Nano projects use ROS 1 Melodic because JetPack 4 is based on Ubuntu 18.04. The newer Jetson Orin experiments use ROS 2. That historical constraint should not be read as a recommendation for a new robot.

## ROS on NVIDIA Jetson

Install ROS natively when the Jetson Linux/Ubuntu release matches a supported ROS platform. Containers are useful when dependencies conflict or when a reproducible development environment is more important than direct host integration. The community-maintained [jetson-containers](https://github.com/dusty-nv/jetson-containers) project provides useful Jetson container recipes.

[Isaac ROS](https://nvidia-isaac-ros.github.io) is a collection of NVIDIA-accelerated ROS 2 packages and container-based development workflows. See the [Isaac ROS quick start](/posts/isaac-ros-quick-start/) for a tested example.

[Foxglove](https://foxglove.dev) can visualize live or recorded ROS data, connect to a robot remotely, and publish messages when the connection supports it.
