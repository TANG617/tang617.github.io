---
title: Start SLAM with RTAB-Map
description: Connecting a RealSense D455 to RTAB-Map for ROS 2 visual SLAM on Jetson Orin.
publishDate: 2024-07-21T20:29:36+08:00
updatedDate: "2026-07-19T00:00:00+08:00"
tags:
  - IntelRealSenseD455
  - SLAM
---

This article records a ROS 2 setup that connects an Intel RealSense D455 to [RTAB-Map](https://github.com/introlab/rtabmap_ros/tree/ros2) for RGB-D SLAM. RTAB-Map provides ROS 1 and ROS 2 branches; use the branch and documentation matching your ROS distribution. `rtabmap_ros` provides the ROS integration for the RTAB-Map core library.

The commands below were tested in July 2024. Package versions and launch arguments may change.

## Examples

Start with the [`rtabmap_examples` launch files](https://github.com/introlab/rtabmap_ros/tree/ros2/rtabmap_examples/launch). Before launching ROS, verify the camera with `realsense-viewer`. The generic RealSense RGB-D setup can be adapted to the D455 by selecting compatible stream profiles and topic names.

- Launch the RealSense node:

```bash
ros2 launch realsense2_camera rs_launch.py align_depth.enable:=true depth_module.depth_profile:=640x360x90 depth_module.infra_profile:=640x360x90 rgb_camera.color_profile:=480x270x90
```

- Launch RTAB-Map:

```bash
ros2 launch rtabmap_launch rtabmap.launch.py frame_id:=camera_link args:="-d" rgb_topic:=/camera/camera/color/image_raw depth_topic:=/camera/camera/aligned_depth_to_color/image_raw camera_info_topic:=/camera/camera/color/camera_info approx_sync:=false queue_size:=20
```

- Console:
  ![Screenshot 2024-07-21 at 20.44.56.png](https://cdn.jsdelivr.net/gh/TANG617/images/202407212045588.png)

- GUI:

![Screenshot from 2024-07-21 20-47-51.png](https://cdn.jsdelivr.net/gh/TANG617/images/202407212049932.png)

- Remote ROS console (Foxglove):
  ![Screenshot 2024-07-21 at 20.25.24.png](https://cdn.jsdelivr.net/gh/TANG617/images/202407212051621.png)

Notes:

1. In the tested configuration, the default RealSense topic namespace contains `camera/camera`.
2. RTAB-Map launches a GUI by default. Set `DISPLAY=:0` in the current shell when the process needs access to the local X display; avoid adding it globally unless that is appropriate for the system.
3. Increasing `queue_size` to 20 prevented synchronization stalls in this test. Treat it as a troubleshooting observation, not a universal requirement.

## SLAM with Jetson Orin

- Reconnect the USB-C cable with the opposite orientation if the camera is not detected. Some cables or adapters may not expose SuperSpeed reliably in both orientations.
- Lower the camera resolution or frame rate if USB bandwidth is insufficient.

```bash
ros2 launch realsense2_camera rs_launch.py align_depth.enable:=true depth_module.depth_profile:=640x360x30 depth_module.infra_profile:=640x360x30 rgb_camera.color_profile:=480x270x30
```
