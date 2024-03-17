---
title: Brief of ROS
date: 2024-03-17 19:38:04
tags:
  - ROS
---
## Brief
ROS or Robotic Operating System is not an operating system like Linux or Windows, strictly speaking. It defines some protocols or  messages that united packages in C/CPP or Python code by different author. The ROS itself is more like a series of tools of manage code, compile code, install package, and visualization.
The version of ROS is pretty confusing. ROS 1 is no longer in developing, and only ROS Noetic is in LTS. ROS 2 has  ROS Humble in LTS and ROS Iron in developing. I'd like to complain about the naming of ROS can't differ the latest version and recurring spelling issues, and to the worse, different version of ROS package (like Noetic and Melodic) may have compatible issue.

This Brief of ROS is focused on ROS 1, mainly Melodic and Noetic. The hardware platform is mostly Jetson Nano. Some exploration is on Jetson Orin Nano.
ROS Melodic is the latest version that is supported on Ubuntu 18.04 of Jetson Nano.
ROS official recommends to use Docker instead of installing locally. And Nvidia has a [official container repo](https://github.com/dusty-nv/jetson-containers) for Jetson, which is pretty convenient.
Nvidia official [Issac-ROS](https://nvidia-isaac-ros.github.io) seems to be powerful, but I have never tried.
[Foxglove](https://foxglove.dev)is one of the best visualization platform, which is able to remotely monitor the robot and can even publish the message.