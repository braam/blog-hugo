---
title: "Linux Intel Graphic Drivers"
date: 2022-04-19T12:18:13+02:00
draft: false
tags:
- Linux
---

When playing video files I did notice that my monitor sometimes was showning lines across the monitor it was like there is some lag when action scenes where playing inside the movie.
Also scrolling in firefox was a bit laggy. 

Did is what I did to fix it, make sure you are running the intel driver:
```
sudo inxi -Gx
```

If you are not running the intel driver but probably the vesa driver, make sure that Xorg is running the intel driver. Use the following trick:
Create the file intel.conf inside the directory /etc/X11/xorg.conf.d with following content:
```
Section "Device"
  Identifier  "DEvice0"
  Driver      "intel"
  Option      "TearFree" "True"
```

After the changes, please reboot your linux machine.

![Linux intel graphic driver](/posts_images/linux_intel_driver_graphics_driver.png)
