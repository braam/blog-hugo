---
title: "Rpi Reset Root Password"
date: 2022-02-07T14:29:37+01:00
draft: false
tags:
- Raspberry-Pi
---

These steps describe the process on how to reset the root password of your raspberry pi.

1. Power down and pull the SD card out from your Pi and put it into your linux computer. Open the file 'cmdline.txt' and add 'init=/bin/sh' to the end. This will cause the machine to boot to single user mode.
2. Put the SD card back in the Pi and boot.
3. When the prompt comes up, type '**su**' to log in as root (no password needed). Type "**passwd pi**" and then follow the prompts to enter a new password.
4. Shut the machine down, then pull the card again and put the cmdline.txt file back the way it was by removing the 'init=/bin/sh' bit.

**The cmdline.txt should look something like this**:
```bash
dwc_otg.lpm_enable=0 console=ttyAMA0,115200 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline rootwait init=/bin/sh
```

&nbsp;
### Possible Issues
* Root account prompting for password:
If the root account is prompting for a password (not common) you can, back on your computer, open the /etc/shadow file and replace the root password in there with an asterisk.
This will change the password to be blank.
* Error when changing the password
Sometimes the password won't be able to be changed because the Pi will boot in a read-only mode. You'll get an error that you can't change the password. 
To fix this, remount the drive in read-write mode:
```bash
mount -o remount,rw /
```
