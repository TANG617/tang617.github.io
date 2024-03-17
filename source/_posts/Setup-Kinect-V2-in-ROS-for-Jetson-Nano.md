---
title: Setup Kinect V2 in ROS for Jetson Nano
date: 2024-03-17 20:05:42
tags:
  - JetsonNano
  - ROS
  - Kinect2
---
## Hardware Hacking
Kinect V2 is originally designed for Microsoft XBOX One, and it's capable to connect to console in a single wire with a special port (which is similar to USB 3.0 type B, but with additional pin ). However, KinectV2 developer kit add an additional power adapter accepting 110V-220V AC and a HUB to transfer the special port to generic USB 3.0 type B, which makes it impossible to be mounted on a tiny robotic vehicle. I have done a fews tests to prove the special is just an USB 3.0 type B with additional 12V DC power in, and the HUB is just a USB 3.0 Hub, of which the only function is to trigger a LED to indicate whether USB is plugged in. (Microsoft, great works) So, I tear down the shield of KinectV2 and weld a power cable with XT-30 connector and buy a USB 3.0 type B to USB 3.0 type C cable. The total costs comes to $10, saving official adapter and HUB. It works perfectly!
## [libfreenect2](https://github.com/OpenKinect/libfreenect2)
Libfreenect2 is an open source Kinect V2 driver that supports multi-platforms.
The most recent update driver to 5 years ago, the dependence is quite outdated.
- Make sure that `cmake ..` can find CUDA dependence. 
- Disable OpenCL in `CMakeLists.txt`, which because JetPack doesn't have OpenCL dependence, and `./Protonect cl` lead errors. It's fine to clone the source code of OpenCL and compile locally, but it's not necessary.
- Build with `cmake .. -DENABLE_CXX11=ON -DCUDA_PROPAGATE_HOST_FLAGS=off -DCMAKE_INSTALL_PREFIX=$HOME/freenect2` 
- Don't forget `make install`, and library should goes to `$HOME/freenect2` 
- Install USB driver for Kinect V2 `sudo cp ../platform/linux/udev/90-kinect2.rules /etc/udev/rules.d/`
- Note that Kinect V2 only supports USB 3, and some USB-C to USB-A is sensitive for flipping.
- If `tegraxusb` reports error when launching `Pronect`, try to disable it in `CMakeLists.txt`

## [iai_Kinect2](https://github.com/paul-shuvo/iai_kinect2_opencv4)
This is a bridge connecting libfreenect2 driver to ROS. The original iai_kinect2 is not compatible with OpenCV 4, so the modified fork is a better choice.
When compiling, OpenCV is still reporting warning on not finding OpenCV 3.2 but OpenCV 4, it's just fine.
Make sure  libfreenect2 is tested before.
My current environment contains a self compiled OpenCL with [this guide](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwis3OKhp_uEAxXvb_UHHQNzCJUQFnoECBQQAQ&url=https%3A%2F%2Fyunusmuhammad007.medium.com%2Fbuild-and-install-opencl-on-jetson-nano-10bf4a7f0e65&usg=AOvVaw22xQefH7une_caKQbyzb30&opi=89978449). If errors occurs when launching `kinect2_bridge`, try to compile and install OpenCL.
## Others
- When streaming Kinect2 images, only depth images's process is on CUDA, which makes it a very high CPU usage, and when streaming 2 videos in parallel, serval packet loss occurs.
- Compressed videos comes with significant lower latency.
